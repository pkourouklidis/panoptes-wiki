To illustrate the core concepts of Panoptes uses we will follow the scenario of the [demo system](Demo-System.md) and build a PDL script step by step. The [web editor](http://editor.panoptes.uk/) provides syntax highlighting and error checking that can help you along the way.

## Preliminary concepts
We will start by defining a _Deployment_ which groups together all of the information needed to monitor the performance of a deployed ML model.
```
Deployment callcenter{
    model callcenter-tree
}
```
The above PDL snippet defines a _Deployment_ named _callcenter_ and specifies that this _Deployment_ is using the _callcenter-tree_ model to produce predictions about customers' happiness.

Since we reference an ML model called _callcenter-tree_ we also need to define it.
```
Model callcenter-tree{
    uses wait_duration, service_duration, is_solved
    outputs hapiness_prediction
    predicts is_happy
}

FeatureStore{
    features
        wait_duration,
        service_duration,
        is_solved
    labels 
        is_happy
}
```
In the PDL snippet above, we have defined a _Model_ as well as a _FeatureStore_ that holds the _Features_ and _Labels_ that the _Model_ references. As we can see the _callcenter-tree_ _Model_ uses three _Features_ as inputs, produces a _Prediction_ named _happiness_prediction_ that predicts the _Label_ _is_happy_.

## Defining Dataset-Shift-Detection Algorithms
We will now start defining our dataset-shift-detecting process. We will start by defining an _Algorithm_.
```
BaseAlgorithm kolmogorov-smirnov{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    parameters pValue
}
```
An _Algorithm's_ job is to decide on the presence of dataset shift based on the data it receives as input. The input differs for the two types of _Algorithms_ that a user can define. The first type is the _Base Algorithm_. In the general case, _Base Algorithms_ receives as input _historical data_ (ie. data that was used to train the ML model) and _live data_ (ie. recent data seen in prediction requests). The above is an example of a _Base Algorithm_ in PDL.

The referenced [git repository](https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo) includes the following files:

detector.py
```python
from scipy import stats

#two-sample Kolmogorov-Smirnov test
def detector(trainSet, liveSet, parameters):
    firstFeature = trainSet.axes[1][0]
    pValue = stats.ks_2samp(trainSet[firstFeature].to_list(), liveSet[firstFeature].to_list())[1]
    threshold = float (parameters.get("pValue", 0.05))
    return int(pValue < threshold), pValue
```

requirements.txt
```
scipy
```
A couple of clarifications:
- As PDL is used for specifying the monitoring process at a high level, the algorithm itself is implemented in a general-purpose programming language and stored in the git repository specified with the _codebase_ keyword. As you can see, the git repo of the above _Base Algorithm_ is very sparse. It only contains a single python file with a single function. It contains no logic regarding fetching _historical data_ and _live data_. This is because the data fetching is handled by the _Algorithm Runtime_ that each _Algorithm_ references. We will touch upon _Algorithm Runtimes_ in the next section. 

- When an _Algorithm_ is executed it must return a number in [0,n), where n is the number specified with the _severity levels_ keyword. The value 0 should be used to signify the absence of dataset shift and subsequent numbers to signify the presence of dataset shift of increasing severity.

- An _Algorithm_ can be parameterized. The names of the _Algorithm's_ parameters are specified with the _parameters_ keyword.

The second type of _Algorithm_ is the _HigherOrderAlgorithm_. This type of _Algorithm_ always receives as input the results from a series of executions of another _Algorithm_. Here is an example of a _Higher Order Algorithm_ in PDL.

```
HigherOrderAlgorithm exponential-moving-average{
    codebase "blah"
    runtime higherOrderPythonFunction
    parameters 
        alpha,
        threshold
    severity levels 2
}
```

As we can see, it is very similar to the _Base Algorithm_, it just references a different _Algorithm Runtime_.

## Algorithm Runtime
As seen in the previous PDL snippets, every _Algorithm_ definition needs to reference an _Algorithm Runtime_. The job of an _Algorithm Runtime_ is to fetch all the data that an _Algorithm_ needs for its execution, call the _Algorithm_, receive the result of the execution and forward it to the component that requested the execution of the _Algorithm_.

That being said, data scientists **do not** need to implement _Algorithm Runtimes_ themselves. They can choose one of the [available Algorithm Runtimes](Algorithm-Runtimes.md) that fits their needs. The only thing necessary is a simple "placeholder" that we can reference when we add _Algorithms_ in our PDL scripts. Here are two examples of the two types of _Algorithm Runtimes_ (corresponding to the two types of _Algorithms_).

```
BaseAlgorithmRuntime pythonFunction
```

```
HigherOrderAlgorithmRuntime higherOrderPythonFunction
```

## Using an Algorithm for Dataset Shift Detection
So far we have seen that we can create _Algorithms_ that detect dataset shift based on the received input. In addition to that, we also need a way to specify which data the _Algorithms_ should receive as input and what should happen if dataset shift is detected. This way, we can reuse an _Algorithm_ to detect dataset shift for multiple deployed models.

For this purpose, we can add to our _Deployment_, _Base Algorithm Executions_ and _Higher Order Algorithm Executions_. Here is an example of a _Base Algorithm Execution_.

```
Deployment callcenter{
    model callcenter-tree

    BaseAlgorithmExecution wait_duration_shift{
        algorithm kolmogorov-smirnov
        live data service_duration
        historical data service_duration
        actions 1->emailMe
        parameter values pValue = 0.05
    }
}
```

As we can see for _Base Algorithm Executions_, we can specify the following:
- The _Algorithm_ to be executed.
- The _Features_/_Predictions_/_Labels_ that will be used as input from the _historical data_ and the _live data_. **Beware**, it is not necessary to include data from both sets. Some _Algorithms_ might be fine to operate only on _live data_. We can for example calculate the accuracy/recall/f1 score that a deployed model has achieved on recent data by using as input the _Prediction_ that the _Model_ outputs and the _Label_ it is trying to predict in the _live data_.    
- A mapping from the potential results of the _Algorithm Execution_ to the _Action Execution_ that should be triggered. **Remember**: The result of an _Algorithm Execution_ is a number in [0,n) where n is the _severity levels_ of the _Algorithm_ used.
- Values for the parameters of the _Algorithm_ if there are any.

Let's now also add a _HigherOrderAlgorithmExecution_ in our _Deployment_.
```
Deployment callcenter{
    model callcenter-tree

    BaseAlgorithmExecution wait_duration_shift{
        algorithm kolmogorov-smirnov
        live data service_duration
        historical data service_duration
        actions 1->emailMe
        parameter values pValue = 0.05
    }

    HigherOrderAlgorithmExecution ema-wait_duration_shift{
	algorithm exponential-moving-average
	observed execution wait_duration_shift
	min observations 3
	max  observations 5
	parameter values alpha = 0.5, threshold = 0.8
	actions 1->emailMe
	}
}
```
As we can see it is quite similar to a _BaseAlgorithmExecution_ with a couple of exceptions:
- Instead of defining which _Features_/_Predictions_/_Labels_ should be used as input, we specify an _AlgorithmExecution_ whose execution results will be used as inputs.
- We specify a minimum number of execution results of the observed _AlgorihtmExecution_ that need to be available for the _HigherOrderAlgorithmExecution_ to start executing.
- We specify a maximum number of results of the observed_AlgorithmExecution_ that will be used as input for the _HigherOrderAlgorithmExecution_. In the example above, the latest 5 execution results of the wait_duration_shift _BaseAlgorithmExecution_ will be used as input for the ema-wait_duration_shift _HigherOrderAlgorithmExecution_.

## Action
An _Action_ is a functionality of the underlying platform that can be triggered in response to dataset shift. All of the currently available _Actions_ can be seen [here](Actions.md).

As an example, when an _Algorithm Execution_ indicates the presence of dataset shift, we could send an email notification.

```
Action email{
    parameters email
}
```

Similarly to _Algorithm Runtimes_, _Action_ implementations are not meant to be developed by data scientists. They exist in our PDL scripts just to be referenced by _Action Executions_ that can be triggered in response to dataset shift.

## Action Execution
_Action Executions_ parameterize a generic _Action_ so it can be used in a specific scenario. For example, the email _Action_ needs to be parametrized with the email address of the intended email notification recipient.

```
ActionExecution emailMe{
    action email
    parameter values email=panagiotis.kourouklidis@bt.com
}
```
## Trigger
A _Trigger_ specifies how often a _Base Algorithm Execution_ should be triggered. **Beware**, _Triggers_ only specify the execution schedule of _Base Algorithm Executions_ because the execution schedule of _Higher Order Algorithm Executions_ is implicitly defined by the schedule of the _Algorithm Execution_ they observe. In essence, a _Higher Order Algorithm Execution_ will run after _Algorithm Execution_ it observes has run and thus follows the same schedule. The only exception is when the observed _Algorithm Execution_ is new and hasn't run enough times to satisfy the _Higher Order Algorithm's_ _min observations_ threshold. In that case, the observed _Algorithm Execution_ will run a few times without triggering the _Higher Order Algorithm Execution_ but after being executed _min observation_ times, each new run **will** trigger the _Higer Order Algorithm Execution_.

Let's add a _Trigger_ to our _Deployment_.
```
Deployment callcenter{
    model callcenter-tree
    
    BaseAlgorithmExecution wait_duration_shift{
        algorithm kolmogorovSmirnov
        live data wait_duration
        historical data wait_duration
        actions 1->emailMe
        parameter values pValue = 0.05
    }
    
    ActionExecution emailMe{
        action emailAction
        parameter values email=panagiotis.kourouklidis@bt.com
    }
    
    Trigger t1{
        every
        100 samples
        100 prediction
        100 labels
        or
        every
        one day
        execute wait_duration_shift
    }
}
```

The above trigger, for example, specifies that the _Algorithm Execution_ *exec1* will be triggered if either of the following two alternatives is true:
- There have been at least 100 prediction requests, 100 prediction responses, and 100 ground truth labels since the last time this trigger has gone off (all three conditions must be true).
- There has been at least one day since the last time this trigger has gone off.

In summary, there are four types of events that can make a trigger go off:
- A new data point has been received that can be used as an ML model input (_samples_ keyword).
- A new prediction has been produced based on the above sample (_predictions_ keyword).
- The ground truth label for the above prediction has become available (_labels_ keyword).
- A certain amount of time has passed since the last time the _Trigger_ has gone off (one day/one week/one month/one year keywords).

A couple of clarifications:
- In an online serving setting, _Samples_ and _Predictions_ will come in pairs. But because of the possibility of batch serving, where we collect _samples_ over a period of time and make _predictions_ on all of them at once, PDL users can define _Triggers_ based on either event.
- When a _Base Algorithm Execution_ defines two or more related inputs (eg a _Prediction_ and its related _Label_) the data will be made available in a tabular format (ie a Pandas dataframe) where their relationship will be preserved. Given that a _Label_ always comes after a _Prediction_ is produced and a _Prediction_ is produced after a _Sample_ is received, the input will have some missing values if, for example, there is a _Prediction_ for which the corresponding _Label_ is not yet available. With that in mind, to guarantee that the input will have a certain number of complete rows, it should be based on the input that comes last. We should for example define a _Trigger_ every 100 _Labels_ to ensure that the input will have at least 100 rows with _Prediction_ as well as _Label_ values.
