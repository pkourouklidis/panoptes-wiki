This page provides an overview of the core concepts that Panoptes uses and the PDL syntax used to express them.

## Platform
A _Platform_ describes the underlying infrastructure/environment used for the training, deployment, and monitoring of ML models. Every PDL script maps to exactly one _Platform_ instance so there's no need to explicitly define it.

## Feature Store
Every _Platform_ must contain a _Feature Store_ which stores information about all available _Features_ and _Labels_. 

```
FeatureStore{
    features
        wait_duration
        service_duration
    labels 
        is_happy
}
```

## Model
To describe trained ML models that are ready for deployment, users can create _Model_ instances.

```
Model callcenter-linear{
    uses wait_duration, service_duration
    outputs hapiness_prediction
    predicts is_happy
}
```


## Algorithm
An _Algorithm_ can be used to detect dataset shift. There are two kinds of _Algorithms_ that a user can define. _Base Algorithms_ and _Higher Order Algorithms_. A _Base Algorithm_ receives as input historical data (ie. data that was used to train the ML model) and live data (ie. recent data seen in inference requests). A _Higher Order Algorithm_ receives as input, a set of outputs from previous executions of other _Algorithms_.

```
BaseAlgorithm kolmogorovSmirnov{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    parameters pValue
}
```

- As PDL is used for specifying the monitoring process at a high level, the algorithm itself is implemented in a general-purpose programming language and stored in the git repository specified with the _codebase_ keyword. How the algorithm is implemented depends on the _Algorithm Runtime_ that is used to execute it.

- When an _Algorithm_ is executed it must return a number in [0,n), where n is the number specified with the _severity levels_ keyword. This can express different severity levels of dataset shift if the _Algorithm_ supports it. Users can specify different _Actions_ taken in response to different _severity levels_.

- An _Algorithm_ can be parameterized. The names of the _Algorithm's_ parameters are specified with the _parameters_ keyword.

## Algorithm Runtime
An _Algorithm Runtime_ represents the capability of the underlying platform to execute _Algorithms_ that are implemented using a specific technology (eg. An _Algorithm_ implemented as a Python function). As with _Algorithms_, there are two kinds of _Algorithm Runtimes_, _Base Algorithm Runtimes_ and _Higher Order Algorithm Runtimes_.

```
BaseAlgorithmRuntime pythonFunction
```

```
HigherOrderAlgorithmRuntime HoPythonFunction
```

## Algorithm Execution
An _Algorithm_ can detect dataset shift in a variety of diffent scenarios. On the other hand, an _Algorithm Execution_ is the application of an _Algorithm_ in a specific scenario. 

```
BaseAlgorithmExecution wait_duration_shift{
    algorithm kolmogorovSmirnov
    live data wait_duration
    historical data wait_duration
    actions 1->retrainCallcenterLinear
    parameter values pValue = 0.05
}
```

For _Base Algorithm Executions_, the user has to specify:
- The _Algorithm_ to be executed.
- Which features will to be used as input from the historic and the live dataset.   
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