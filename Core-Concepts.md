This page provides an overview of the core concepts that Panoptes uses. For the PDL syntax that can be used to create instances of these concepts in a way that fits each individual scenario take a look at the [PDL syntax section](PDL Syntax).

# Platform
A _Platform_ describes the underlying infrastructure/Platform used for the training, deployment, and monitoring of ML models. Every PDL script implicitly includes a _Platform_ instance so there's no PDL syntax to create one.

# Model
To describe trained ML models that are ready for deployment, users can create _Model_ instances.

```
Model "callcenter-linear"{
    uses wait_duration, service_duration
    outputs p1 predicts is_happy
}
```