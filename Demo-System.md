To demonstrate the features of Panoptes we have developed an example application that simulates a call center.

Supposedly, there is a call center that customers can contact when they are having an issue. Initially, the customer is put in a wait queue until a call center worker is available to answer the call. When a worker becomes available, they answer the call and speak with the customer for an amount of time, and then the call ends. There is a chance that the worker is not able to resolve the customer's issue. Based on the interaction, the customer is either happy or not.

After the end of each call, the system requests a prediction about the customer's happiness from an endpoint that hosts a trained ML model. The prediction input is the wait time, the service time, and whether the customer's issue was resolved or not.

The purpose of this example system is to adjust the various simulation variables to introduce dataset shift/affect the performance of the model and then see if we can use Panoptes to detect that and apply corrective measures.

The following are the two user interfaces that you can use to interact with the application.

## User Dashboard
The user Dashboard can be accessed [here](https://ui.dashboard.callcentre.panoptes.betalab.rp.bt.com).

## Simulation Dashboard
The simulation dashboard can be accessed [here](https://ui.digitaltwin.callcentre.panoptes.betalab.rp.bt.com).
