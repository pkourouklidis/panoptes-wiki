Out of the box, the [Panoptes web editor](http://editor.panoptes.uk) can help users catch many errors but these are mainly related to syntax. To help ensure that the semantics of users' PDL scripts are also correct, the language provides a number of optional features.

## Parameter Type Correctness
When creating _Algorithms_  and _Actions_ you can add type annotations to any defined parameters. _Algorithm Execution_ and _Action Executions_ then must provide parameter values that respect the annotated parameters' types.
```
BaseAlgorithm kolmogorov-smirnov{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    parameters pValue:Real
}

BaseAlgorithmExecution wait_duration_shift{
    algorithm kolmogorov-smirnov
    live data wait_duration
    historical data wait_duration
    actions 1->retrainCallcenter
    parameter values pValue = true
}
```
The above PDL script will cause the web editor to output an error because the parameter _pValue_ of the _kolmogorov-smirnov Base Algorithm_ was expecting a real number value but received a _boolean_.

Here are the parameter types supported in PDL:
- Real (e.g 0.1, 1, 10.554)
- Integer (e.g 2, -3, 15)
- Boolean (true, false all lowercase)
- String (with quotes any character is ok "sadfdfs fasdfasd://**". Without quotes only A-Z/a-z/@/_/. 0-9 and - are ok but not at the beginning of the string AcbD-08_7@gmail.com)

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

BaseAlgorithm kolmogorovSmirnov{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    accepts only continuous
    parameters pValue:Real
}

BaseAlgorithmExecution wait_duration_shift{
    algorithm kolmogorovSmirnov
    live data is_happy
    historical data is_happy
    actions 1->retrainCallcenterLinear
    parameter values pValue = 0.05
}
```
The above PDL script will once again cause an error because the _kolmogorovSmirnov_ _Base Algoririthm_ only accepts _continuous_ random variables as input but is passed a _categorical_ one. We can make the check less strict by omitting the _only_ keyword. This will result in a warning instead of an error which still informs the user but allows the deployment of the PDL script to production.

## Deployment Input Correctness
Panoptes can also check if inputs of a _Deployment_ are compatible with the inputs of the _Deployment's_ _Model_. For example, if a _Model_ needs a call's wait_duration and service_duration to calculate a prediction then the _Deployment_ needs to receive these two features for every prediction request. Alternatively, if given the ID of a call, the _Deployment_ can retrieve the wait_duration and service_duration of that call, then the _Deployment_ only needs to receive the call's ID in a prediction request.

Here's an example PDL script that illustrates the point
```
FeatureStore{
    entities
        call{keys callID}
    features
	wait_duration{requires entities call},
	service_duration{requires entities call}
    labels 
	is_happy
}

Model callcenter-tree{
    uses wait_duration, service_duration
    outputs happiness_prediction
    predicts is_happy
}

Deployment callcenter{
    inputs callID
    model callcenter-tree
}
```
As we see there is additional information added to the FeatureStore definition. For every feature, we have added the _entity_ that the _feature_ is associated with. In addition, for every _entity_ we have defined the _keys_ that one needs in order to uniquely identify it. Lastly, for every _Deployment_ we have added the _inputs_ that it receives with prediction requests.

In summary, a _callID_ uniquely identifies a _call_. Knowing the _call_ we can retrieve its _wait_duration_ and _service_duration_. The _Model_ that we use only needs these two _features_ as input. Therefore a _callID_ is all that the _Deployment_ needs to receive in a prediction request to successfully produce a prediction.

For completeness, there is also a way to express the fact that some features are not associated with any entity but are rather computed on the fly from the data that is received in a prediction request.

```
FeatureStore{
    entities
        call{keys callID}
    request data additional_data
    features
	wait_duration{requires entities call},
	service_duration{requires entities call},
	additional_feature{requires request data additional_data}
    labels 
	is_happy
}

Model callcenter-tree{
    uses wait_duration, service_duration, additional_feature
    outputs happiness_prediction
    predicts is_happy
}

Deployment callcenter{
    inputs callID, additional_data
    model callcenter-tree
}
```
Above we can see that the _Model_ now also requires _additional_feature_ as input. This _feature_ can be calculated on the fly from the input _additional_data_. Therefore the _Deployment_ now needs _additional data_ as input as well as _callID_ in order to retrieve/calculate all of the inputs that the _Model_ requires. 