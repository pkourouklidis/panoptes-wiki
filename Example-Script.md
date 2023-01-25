The example script can be seen in the [web editor](http://editor.panoptes.uk/). If it has been changed by another participant you can copy/paste from below.
```
FeatureStore{
	features
	    wait_duration:continuous,
	    service_duration:continuous,
	    is_solved:categorical
	labels 
	    is_happy:categorical
}

Model callcenter-tree{
    uses wait_duration, service_duration, is_solved
    outputs hapiness_prediction
    predicts is_happy
}

BaseAlgorithmRuntime pythonFunction

BaseAlgorithm kolmogorovsmirnov{
	codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
	runtime pythonFunction
	severity levels 2
	accepts only continuous
	parameters pValue:Real
}

BaseAlgorithm accuracycheck{
	codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/accuracy-algorithm-repo"
	runtime pythonFunction
	severity levels 2
	parameters threshold:Real
}

Action email{
	parameters mandatory email:String
}

Action retrain{
    parameters
        mandatory ioNames:String,
        mandatory containerImage:String
}

Deployment callcenter{
	model callcenter-tree
	
	BaseAlgorithmExecution service_duration_shift{
		algorithm kolmogorovsmirnov
		live data service_duration
		historical data service_duration
		actions 1->emailMe
		parameter values pValue = "0.05"
	}
	
	BaseAlgorithmExecution callcenter-accuracy{
		algorithm accuracycheck
		live data is_happy, callcenter-tree.hapiness_prediction
		actions 1->emailMe
		parameter values threshold = "0.80"
	}
	
	ActionExecution emailMe{
		action email
		parameter values email=panagiotis.kourouklidis@bt.com
	}
	
	ActionExecution retrainCallcenterLinear{
	    action retrain
	    parameter values ioNames="wait_duration,service_duration,is_solved,is_happy",  
	        containerImage="registry.docker.nat.bt.com/panoptes/callcenter-model-training:latest"
	}
	
	/*Trigger t1{
	every
	100 samples
	or
	every
	one day
	execute service_duration_shift
	}*/
	
	Trigger t2{
	every
	100 labels
	execute callcenter-accuracy
	}

}
```