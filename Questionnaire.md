Please only fill in the questionnaire **after** carefully reading the documentation material and contacting me to answer any questions that have come up. Upon completion, please email me the filled-in questionnaire. Thank you!

- Teams: Panagiotis Kourouklidis
- Email: [panagiotis.kourouklidis@bt.com](mailto:panagiotis.kourouklidis@bt.com)

## Questions

PDL keywords are in _italics_. Identifiers/names are in **bold**

1. What is the name of the _Model_ that the **callcenter** _Deployment_ uses?
1. What are the names of the _AlgorithmExecutions_ defined in the **callcenter** _Deployment_?
1. What is the name of the _Algorithm_ used by the **service-duration-shift** _AlgorithmExecution_?
1. What is the URL of the git repo that contains the code for the _Algorithm_ **accuracy-check**?
1. Which _ActionExecution_ will be triggered, in case dataset shift is detected by the **service-duration-shift** _Algorithm Execution_?
1. Which _parameters_ does the **retrain** _Action_ define?
1. In the **retrainCallcenter** _ActionExecution_ can we omit providing a value to the **ioNames** _parameter_? Please justify.
1. In the **service-duration-shift** _AlgorithmExecution_, is the value `true` appropriate for the **pValue** _parameter_? Please justify.
1. What is the statistical type of the _live data_ input for the **service-duration-shift** _AlgorithmExecution_?
1. Could we have used the _Feature_ **is_solved** as input to the **service-duration-shift** _AlgorithmExecution_, without causing any warnings/errors? Please justify.
1. In a scenario where one day has passed since _trigger_ **t1** has been activated but the _Deployment_ has only served 900 requests, will _AlgorithmExecution_ **service-duration-shift** run? Please justify.
1. In a scenario where no _AlgorithmExecution_ has run yet and **callcenter-accuracy** runs for the first time, will _AlgorithmExecution_ **ema-accuracy** run? Please justify.
1. What are the _inputs_ of the **callcenter** _Deployment_?
1. Based on the _inputs_ of the **callcenter** _Deployment_, which _features_ are allowed as input for the _Model_ that **callcenter** uses.
1. How would we need to modify the _inputs_ of the **callcenter** _Deployment_, if we wanted to substitute the current _Model_ that is being used with _Model_ **callcenter-alternative**, without causing any warnings/errors?