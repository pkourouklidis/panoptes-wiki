## email
This _Action_ simply sends an email containing information regarding the _Algorithm Execution_ that triggered it. _Action Executions_ that use this _Action_ must provide a value for the `email` parameter.

## retrain
This _Action_ will trigger the execution of the container provided in the `containerImage` parameter and provide it with the necessary environment variables. It will also pass, via an environment variable, the value provided in the `ioNames` parameter which should contain the names of the features and label that your model needs during training. Here is [an example](https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/callcenter-model-training/-/tree/master) of a training container that this _Action_ can execute.