In this scenario, we want to run a simulation with covariate shift. In essence, the average service time will be increased but customers' tolerance to waiting times and failures will stay the same.

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
	codebase "https://github.com/pkourouklidis/kolmogorov-smirnov-algorithm"
	runtime pythonFunction
	severity levels 2
	accepts only continuous
	parameters pValue:Real
}

BaseAlgorithm accuracy-check{
	codebase "https://github.com/pkourouklidis/accuracy-algorithm"
	runtime pythonFunction
	severity levels 2
	parameters threshold:Real
}

Action email{
	parameters mandatory email:String
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
	
	ActionExecution emailMe{
		action email
		parameter values email=panagiotis.kourouklidis@bt.com
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
- Average Service Time: 10s

### Customer Configuration
- Expected Wait Time: minimum
- Average Patience During Waiting: minimum (fully patient)
- Average Patience During Service: minimum (fully patient)
- Average Understanding for Failure: minimum (fully tolerant)

### Worker Configuration
- Number of Workers: 20
- Average Worker Skill: maximum
- Average Worker Speed: maximum

After applying the settings, start the simulation and let it run for 100 calls. The number of calls served can be seen in the [user dashboard](https://ui.dashboard.callcentre.panoptes.betalab.rp.bt.com/).

## Expected Result
This simulation run produces service times that are significantly higher compared to the ones in the training set. The service_duration_shift algorithm execution should detect this and send us an email notification.