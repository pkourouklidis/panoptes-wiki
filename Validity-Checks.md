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
Similarly to _parameters_, _features_ and _labels_ can also be annotated with a type. From a statistical point of view, a random variable can be:
- _continuous_ (e.g. 0.1,0.25,1.5)
- _categorical_ (aka nominal categorical like cat, dog, mouse)
- _ordered_ (aka ordinal categorical like small medium large)

Some shift detection algorithms work better with a specific type of random variables. To prevent errors, this information can be added to the definition of a _Base Algorithm_ with the _accepts_/_accepts only_ keyword.

```
FeatureStore{
    features
        wait_duration:continuous,
        service_duration:continuous
    labels 
        is_happy:categorical
}

BaseAlgorithm kolmogorov-smirnov{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    runtime pythonFunction
    severity levels 2
    accepts only continuous
    parameters pValue:Real
}

BaseAlgorithmExecution wait_duration_shift{
    algorithm kolmogorov-smirnov
    live data is_happy
    historical data is_happy
    actions 1->retrainCallcenterLinear
    parameter values pValue = 0.05
}
```
The above PDL script will cause an error because the _kolmogorov-smirnov_ _Base Algoririthm_ only accepts _continuous_ random variables as input but is passed a _categorical_ one. We can make the check less strict by omitting the _only_ keyword. This will result in a warning instead of an error which still informs the user but allows the deployment of the PDL script to production.

## Deployment Input Correctness
A common function of _feature stores_ is that they _associate_ each feature with an _entity_. For example, the _features_ _wait_duration_ and _service_duration_ can be associated with the _entity_ _call_. Each _entity_ is also given a _key_ that can uniquely identify it. We can then, given an _entity's_ _key_ retrieve from the _feature store_ all of the _features_ that are associated with it.

The PDL script below shows how we can define the _feature_-_entity_ relationships described above.
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
```

With this setup in our _feature store_, we no longer need to receive the values of _wait duration_ and _service duration_ with each prediction request. We only need the value of the _callID_ and we can retrieve the values of _wait duration_ and _service duration_ from the _feature store_.

To avoid errors, Panoptes can automatically check if a _Deployment_ receives the right _keys_ as input to retrieve all of the _features_ that the deployed _Model_ requires for a prediction. All we need to do is add information about which _keys_ the _Deployment_ receives with every prediction request.

Here is what needs to be added to the PDL script:
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

In summary, a _callID_ uniquely identifies a _call_. Knowing the _call_ we can retrieve its _wait_duration_ and _service_duration_. The _Model_ that we use only needs these two _features_ as input. Therefore a _callID_ is all that the _Deployment_ needs to receive in a prediction request to successfully produce a prediction.

For completeness, there is also a way to express the fact that some _features_ are not associated with any _entity_ but are rather computed on the fly from a data field that is received in a prediction request.

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