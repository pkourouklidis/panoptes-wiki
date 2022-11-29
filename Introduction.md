# What is Panoptes?
Panoptes is a low-code platform that helps you monitor the performance of deployed ML models. It does so by providing a coordination layer on top of your existing infrastructure so it can work on-prem or in the cloud provider of your choice.

Data scientists can interact with Panoptes via Panoptes Description Language (PDL). Using PDL, data scientists can define how they would like their ML models to be monitored without worrying about infrastructure-specific technical details. More details about PDL can be found in the [syntax section](syntax). 

# How does it work?
To help us understand the inner workings of Panoptes, let's first look at a relatively simple set of resources being used to train ML models and serve their predictions.

![panoptesGCP_-_noPanoptes.drawio.svg](uploads/67a13fc6cebf6c54bb6625a01b606cd2/panoptesGCP_-_noPanoptes.drawio.svg)

The figure demonstrates a scenario where we:
- Pull raw data from BigQuery.
- Extract our features and store them in a feature store.
- Pull our feature data into a notebook server to train our ML model.
- Push the resulting ML model artifact to a model registry.
- Pull the ML model artifact into a model server and start serving inference requests

Now imagine that for every 1000 inference requests served, we would like to execute an algorithm that checks that the statistical distribution of newly received features closely follows the distribution seen in the training set. We would have to either develop that functionality from scratch, which would require considerable effort, or use a managed service and be constrained to the functionality offered by the service and locked into a specific platform.

Panoptes bypasses this problem by providing a coordination layer that can be configured via an easy-to-use domain-specific language, PDL. In the figure below, we can see the additional components that are added to the infrastructure of our example.

