In this scenario, we want to run a simulation with concept shift. In essence, the average wait/service time will stay the same but the customers' tolerance to waiting times and failures will be lower. As a result, the accuracy of the ML model that was trained on the initial customer interactions will now be significantly lower.

## PDL script
In this scenario, we ask the participants to modify the script from scenario 2 so that the concept shift is detected and we receive an email notification.

High-level Guide:
- You can utilise the _accuracy-check_ _Algorithm_ to detect the concept shift. The _Algorithm's_ [git repo](https://gitlab.agile.nat.bt.com/BETALAB/research/panoptes/accuracy-algorithm-repo) documents how it should be used.
- You also have to make an addition to the script to ensure that the _Algorithm_ is going to be executed after 100 calls.

## Simulation settings
Apply the following settings in the [simulation dashboard](https://ui.digitaltwin.callcentre.panoptes.betalab.rp.bt.com/):

### Call Configuration
- Call Interval: minimum
- Average Call Difficulty: minimum
- Wait Time Until Call Gets Bounced: maximum
- Average Service Time: 5s

### Customer Configuration
- Expected Wait Time: minimum
- Average Patience During Waiting: maximum (fully impatient)
- Average Patience During Service: maximum (fully impatient)
- Average Understanding for Failure: maximum (fully intolerant)

### Worker Configuration
- Number of Workers: 15
- Average Worker Skill: maximum
- Average Worker Speed: maximum

After applying the settings, start the simulation and let it run for 100 calls. The number of calls served can be seen in the [user dashboard](https://ui.dashboard.callcentre.panoptes.betalab.rp.bt.com/).

## Expected Result
Due to the concept shift, the performance of the model will be significantly lower. An algorithm execution using the _accuracy-check_ algorithm or similar should detect this and send us an email notification.