This page provides an overview of the core concepts that Panoptes uses and the PDL syntax used to express them.

# Platform
A _Platform_ describes the underlying infrastructure/Platform used for the training, deployment, and monitoring of ML models. Every PDL script implicitly includes a _Platform_ instance so there's no PDL syntax to create one.

# Feature Store
Every _Platform_ must contain a _Feature Store_ where the information about all available _Features_ and _Labels_ is contained. 

<code><b>FeatureStore</b>{
	<b>features</b>
	    wait_duration
		service_duration
	<b>labels</b> 
	    is_happy
}</code>

# Model
To describe trained ML models that are ready for deployment, users can create _Model_ instances.


    Model "callcenter-linear"{
        uses wait_duration, service_duration
        outputs p1
        predicts is_happy
    }


# Algorithm
An _Algorithm_ can be used to detect dataset shift. There are two kinds of _Algorithms_ that a user can define. _Base Algorithms_ and _Higher Order Algorithms_. A _Base Algorithm_ receives as input historical data (ie. data that was used to train the ML model) and live data (ie. recent data seen in inference requests). A _Higher Order Algorithm_ receives as input, a set of outputs from previous executions of other _Algorithms_.

<pre><code><b>BaseAlgorithm</b> kstest{
	<b>codebase</b> "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
    <b>runtime</b> pythonFunction
    <b>severity levels</b> 2
    <b>parameters</b> pValue
}</code></pre>

- As PDL is used for specifying the monitoring process at a high level, the algorithm itself is implemented in a general purpose programming language and stored in the git repository specifed with the _codebase_ keyword. How the algorithm is implemented depends on the _Algorithm Runtime_ that is used to execute it.

- When an _Algorithm_ is executed it must return a number in [0,n), where n is the number specified with the _severity levels_ keyword. This can express different severity levels of dataset shift if the _Algorithm_ supports it. Users can specify different _Actions_ taken in response to different _severity levels_.

- An _Algorithm_ can be parameterized. The names of the _Algorithm's_ parameters are specified with the _parameters_ keyword.

# Algorithm Runtime
An _Algorithm Runtime_ represents the capability of the underlying platform to execute _Algorithms_ that are implemented using a specific technology (eg. An _Algorithm_ implemented as a Python function). As with _Algorithms_, there are two kinds of _Algorithm Runtimes_, _Base Algorithm Runtimes_ and _Higher Order Algorithm Runtimes_.

<pre><code><b>BaseAlgorithmRuntime</b> pythonFunction</code></pre>

<pre><code><b>HigherOrderAlgorithmRuntime</b> HoPythonFunction</code></pre>

# Algorithm Execution
An _Algorithm_ can detect dataset shift in a variety of diffent scenarios. On the other hand, an _Algorithm Execution_ is the application of an _Algorithm_ in a specific scenario. 

<pre><code><b>BaseAlgorithmExecution</b> exec1{
		<b>algorithm</b> kstest
		<b>live data</b> wait_duration
		<b>historic data</b> wait_duration
		<b>actions</b> 1->retrainCallcenterLinear
		<b>parameter values</b> pValue = "0.05"
	}</code></pre>

For _Base Algorithm Executions_, the user has to specify:
- The _Algorithm_ to be executed.
- Which features will to be used as input from the historic and the live dataset.   
- A mapping from the potential results of the _Algorithm Execution_ to the _Action_ that should be triggered.
- Values for the parameters of the _Algorithm_ if there are any.

# Action
An _Action_ is a functionality of the underlying platform that can be triggered in response to dataset shift.

<pre><code><b>Action</b> retrainAction{
    <b>parameters</b>
        ioNames,
        containerImage
}</code></pre>

# Action Execution
In line with _Algorithm Executions_, _Action Executions_ are the application of a generic _Action_ in a specific scenario.

<pre><code><b>ActionExecution</b> retrainCallcenterLinear{
    <b>action</b> retrainAction
    <b>parameter values</b>
        ioNames="wait_duration,service_duration,is_happy",  
        containerImage="registry.docker.nat.bt.com/panoptes/callcenter-model-training:latest"
	}</code></pre>