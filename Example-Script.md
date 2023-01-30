The example script can be seen with syntax highlighting in the [web editor](http://editor.panoptes.uk/). Please copy/paste it from below to make sure you are looking at the right version. There is no need to press apply for this step.
```
FeatureStore{
    entities
        call{keys callID}
	features
	    wait_duration:continuous{requires entities call},
	    service_duration:continuous{requires entities call},
	    is_solved:categorical{requires entities call}
	labels 
	    is_happy:categorical
}

Model callcenter-tree{
    uses wait_duration, service_duration, is_solved
    outputs hapiness_prediction
    predicts is_happy
}

BaseAlgorithmRuntime pythonFunction

BaseAlgorithm kolmogorov-smirnov{
	codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/example-algorithm-repo"
	runtime pythonFunction
	severity levels 2
	accepts only continuous
	parameters pValue:Real
}

BaseAlgorithm accuracy-check{
	codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/accuracy-algorithm-repo"
	runtime pythonFunction
	severity levels 2
	parameters threshold:Real
}

HigherOrderAlgorithmRuntime higherOrderPythonFunction

HigherOrderAlgorithm exponential-moving-average{
    codebase "https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/ema-algorithm-repo"
    runtime higherOrderPythonFunction
    parameters 
        alpha:Real,
        mandatory threshold:Real
    severity levels 2
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
        inputs call.callID
	model callcenter-tree
	
	BaseAlgorithmExecution service-duration-shift{
		algorithm kolmogorov-smirnov
		live data service_duration
		historical data service_duration
		actions 1->emailMe
		parameter values pValue = 0.05
	}
	
	BaseAlgorithmExecution callcenter-accuracy{
		algorithm accuracy-check
		live data is_happy, callcenter-tree.hapiness_prediction
		parameter values threshold = 0.80
	}

	HigherOrderAlgorithmExecution ema-accuracy{
	    algorithm exponential-moving-average
	    observed execution callcenter-accuracy
	    min observations 3
	    max  observations 3
	    parameter values alpha = 0.5, threshold = 0.8
	    actions 1->emailMe
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
	
	Trigger t1{
	every
	1000 samples
	or
	every
	one day
	execute service-duration-shift
	}
	
	Trigger t2{
	every
	100 labels
	execute callcenter-accuracy
	}

}
```