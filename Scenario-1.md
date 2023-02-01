In this scenario, we want to run a simulation with no dataset shift.

## PDL script
In the [web editor](http://editor.panoptes.uk/), delete the existing script(if any) and paste the PDL script below. Afterward, press the apply button and wait for the "success" message.
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
    outputs happiness_prediction
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
		algorithm kolmogorov-smirnov
		live data service_duration
		historical data service_duration
		actions 1->emailMe
		parameter values pValue = 0.05
	}
	
	BaseAlgorithmExecution callcenter-accuracy{
		algorithm accuracy-check
		live data is_happy, happiness_prediction
		actions 1->emailMe
		parameter values threshold = 0.80
	}
	
	ActionExecution emailMe{
		action email
		parameter values email=panagiotis.kourouklidis@bt.com
	}
	
	ActionExecution retrainCallcenter{
	    action retrain
	    parameter values ioNames="wait_duration,service_duration,is_solved,is_happy",  
	        containerImage="registry.docker.nat.bt.com/panoptes/callcenter-model-training:latest"
	}
	
	Trigger t1{
	every
	100 samples
	execute service_duration_shift
	}
}
```

## Simulation settings
Apply the following settings in the [simulation dashboard](https://ui.digitaltwin.callcentre.panoptes.betalab.rp.bt.com/):

### Call Configuration
- Call Interval: minimum
- Average Call Difficulty: minimum
- Wait Time Until Call Gets Bounced: maximum
- Average Service Time: minimum

### Customer Configuration
- Expected Wait Time: minimum
- Average Patience During Waiting: minimum (fully patient)
- Average Patience During Service: minimum (fully patient)
- Average Understanding for Failure: minimum (fully tolerant)

### Worker Configuration
- Number of Workers: 15
- Average Worker Skill: maximum
- Average Worker Speed: maximum

After applying the settings, start the simulation and let it run for 100 calls. The number of calls served can be seen in the [user dashboard](https://ui.dashboard.callcentre.panoptes.betalab.rp.bt.com/).

## Expected Result
This simulation run produces data that are similar to the ones used to train the ML model. Therefore there will be no dataset shift detected and no email notification.
 