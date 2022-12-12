Out of the box, the [Panoptes web editor](http://editor.panoptes.uk) can help users catch a lot of errors but these are mainly related to syntax. To help ensure that the semantics of users PDL scripts are also correct, the language provides a number of optional features.

## Parameter Type Correctness
_Algorithms_  and _Actions_ might be developed by one person and used in _Algorithm/Action Executions_ created by other members of a team. In that case the creator of the  _Algorithm_/_Action_ add type annotations to any defined _parameters_ to help guide others.
```
BaseAlgorithm kstest{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    accepts only continuous
parameters pValue:Real
}

BaseAlgorithmExecution exec1{
    algorithm kstest
    live data wait_duration
    historic data wait_duration
    actions 1->retrainCallcenterLinear
    parameter values pValue = true
}
```
The above PDL script will cause the web editor to output an error because the parameter _pValue_ of the _kolmogorovSmirnov Base Algorithm_ was expecting a real number value but received a _boolean_.

## Feature Type Correctness
Similarly to _parameters_, _features_ can also be annotated with a type. From a statistical point of view, a random variable can be _continuous_, _categorical_ (aka nominal categorical) or _ordered_ (aka ordinal categorical). Since some shift detection algorithms work best for a specific type of random variables, this information can be added to the definition of a _Base Algorithm_.

```
FeatureStore{
    features
        wait_duration:continuous,
        service_duration:continuous
    labels 
        is_happy:categorical
}

BaseAlgorithm kstest{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    accepts only continuous
parameters pValue:Real
}

BaseAlgorithmExecution exec1{
    algorithm kstest
    live data is_happy
    historic data is_happy
    actions 1->retrainCallcenterLinear
    parameter values pValue = 0.05
}
```
The above PDL script wil once again cause an error because the _kolmogorovSmirnov_ _Base Algoririthm_ only accepts _continuous_ random variables as input but is passed a _categorical_ one. We can make the check less strict by ommiting the _only_ keyword. This will result in a warning instead of an error which still informs the user but allows the deployment of the PDL script to production.