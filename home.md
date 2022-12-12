## The Problem
The fact that an ML model has achieved good performance for some time is not a guarantee that it will continue to perform well indefinitely. The following are a couple of scenarios to illustrate the point.
- For a model that predicts a patient's illness from observed symptoms: Due to the seasonality of certain viruses, certain symptoms might be observed more often in some months (prior probability shift).
- A video hosting service develops a model that predicts link click-through rate based on the user's demographic: A change in users' preferences can affect the click-through rate even if the demographic stays the same (concept shift).
- Alternatively in the above scenario: The service might suddenly become popular with users that belong to a different demographic and that might also affect performance (covariate shift).
The above scenarios can be grouped under the umbrella term of [_dataset shift_](https://www.sciencedirect.com/science/article/abs/pii/S0031320311002901).

## What is Panoptes?
Panoptes is a low-code tool that helps data scientists safeguard the performance of deployed ML models against the various types of dataset shift. It does so by providing a way for data scientists to use the algorithm of their choice to detect dataset shift and trigger corrective actions if necessary.

To reduce the complexity of platform-specific technical details (e.g on-prem vs Google Cloud Platform), Panoptes provides a high-level view of the underlying platform used for the training, deployment, and monitoring of ML models. Data scientists can use Panoptes Description Language (PDL) to create a complete specification of how they would like their models to be monitored. This description is portable across platforms with Panoptes deployed. Data scientists can therefore focus on the core logic of the ML model monitoring rather than the technical implementation.