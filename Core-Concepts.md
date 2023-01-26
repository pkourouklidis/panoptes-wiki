This page provides an overview of the core concepts that Panoptes uses and the PDL syntax used to express them. The [web editor](http://editor.panoptes.uk/) provides syntax highlighting and error checking that can help you get familiar with PDL.

## Platform
A _Platform_ describes the underlying infrastructure/environment used for the training, deployment, and monitoring of ML models. Every PDL script maps to exactly one _Platform_ instance so there's no need to explicitly define it.

## Feature Store
Every _Platform_ must contain a _Feature Store_ which stores information about all available _Features_ and _Labels_. 

```
FeatureStore{
    features
        wait_duration,
        service_duration,
        is_solved
    labels 
        is_happy
}
```

## Model
To describe trained ML models that are ready for deployment, users can create _Model_ instances.

```
Model callcenter-tree{
    uses wait_duration, service_duration, is_solved
    outputs hapiness_prediction
    predicts is_happy
}
```


## Algorithm
An _Algorithm's_ job is to decide on the presence of dataset shift based on the data it receives as input. The input differs for the two types of _Algorithms_ that a user can define. The first type is the _Base Algorithm_. In the general case, _Base Algorithms_ receives as input _historical data_ (ie. data that was used to train the ML model) and _live data_ (ie. recent data seen in prediction requests). Here is an example of a _Base Algorithm_ in PDL.

```
BaseAlgorithm kolmogorovsmirnov{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    parameters pValue
}
```
A couple of clarifications:
- As PDL is used for specifying the monitoring process at a high level, the algorithm itself is implemented in a general-purpose programming language and stored in the git repository specified with the _codebase_ keyword. If you visit the [git repository](https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo) of the above _Base Algorithm_, you will observe that it is very sparse. It only contains a single python file with a single function. It contains no logic regarding fetching _historical data_ and _live data_. This is because the data fetching is handled by the _Algorithm Runtime_ that each _Algorithm references. We will explain more about _Algorithm Runtimes_ in the next section. 

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
The job of an _Algorithm Runtime_ is to fetch all the data that an _Algorithm_ needs for its execution, call the _Algorithm_, receive the result of the execution and forward it to the component that requested the execution of the _Algorithm_. All of this functionality is implemented in a general-purpose language (e.g Python). The only thing necessary is a simple "placeholder" that we can reference when we add _Algorithms_ in our PDL scripts. Here are two examples of the two types of _Algorithm Runtimes_ that we can define (corresponding to the two types of _Algorithms_).

```
BaseAlgorithmRuntime pythonFunction
```

```
HigherOrderAlgorithmRuntime higherOrderPythonFunction
```

## Algorithm Execution
So far we have seen that we can create _Algorithms_ to detect dataset shift based on their input. In addition to that, we also need a way to specify which data the _Algorithms_ should operate on and what should happen if dataset shift is detected. In this way, we can reuse an _Algorithm_ to detect dataset shift for multiple deployed models.

For this purpose, we can create _Base Algorithm Executions_ and _Higher Order Algorithm Executions_. Here is an example of a _Base Algorithm Execution_.

```
BaseAlgorithmExecution wait_duration_shift{
    algorithm kolmogorovsmirnov
    live data wait_duration
    historical data wait_duration
    actions 1->retrainCallcenterLinear
    parameter values pValue = 0.05
}
```

As we can see for _Base Algorithm Executions_, we can specify the following:
- The _Algorithm_ to be executed.
- The features/predictions/labels that will be used as input from the _historical_ and the _live data_. **Beware**, it is not necessary to include data from both sets. Some _Algorithms_ might be fine to operate only on _live data_. We can for example calculate the accuracy/recall/f1 score/etc. on recent data of a deployed model by using as input the prediction and the labels in the _live data_.    
- A mapping from the potential results of the _Algorithm Execution_ to the _Action_ that should be triggered.
- Values for the parameters of the _Algorithm_ if there are any.

## Action
An _Action_ is a functionality of the underlying platform that can be triggered in response to dataset shift.

```
Action retrain{
    parameters
        ioNames,
        containerImage
}
```

## Action Execution
In line with _Algorithm Executions_, _Action Executions_ are the application of a generic _Action_ in a specific scenario.

```
ActionExecution retrainCallcenterLinear{
    action retrainAction
    parameter values
        ioNames="wait_duration,service_duration,is_happy",  
        containerImage="registry.docker.nat.bt.com/panoptes/callcenter-model-training:latest"
}
```
## Trigger
A _Trigger_ specifies how often an Algorithm execution should be triggered.

```
Trigger t1{
    every
    100 samples
    100 predictions
    100 labels
    or
    every
    one day
    execute wait_duration_shift
}
```
The above trigger, for example, specifies that the _Algorithm Execution_ *exec1* will be triggered if either of the following two alternatives is true:
- There have been at least 100 inference requests, 100 inference responses and 100 ground truth labels since the last time this trigger has gone off.
- There has been at least one day since the last time this trigger has gone off.

## Deployment
A _Deployment_ groups together a deployed ML model and all of the information needed to monitor its performance. _Algorithm Executions_, _Action Executions_, and _triggers_ can only be defined within a _Deployment_.

```
Deployment callcenter{
    model callcenter-linear
    
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
    
    ActionExecution retrainCallcenterLinear{
        action retrainAction
        parameter values ioNames="wait_duration,service_duration,is_happy",  
            containerImage="registry.docker.nat.bt.com/panoptes/callcenter-model-training:latest"
    }
    
    Trigger t1{
        every
        100 samples
        or
        every
        one day
        execute wait_duration_shift
    }
}
```