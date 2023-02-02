Welcome to the Panoptes evaluation study. Thank you for participating! The goal of this study is to evaluate the usability and utility of Panoptes in a context that is as realistic as possible under laboratory conditions.

As opposed to the previous study in which you may have participated, this is a self-directed study. You are kindly asked to follow the step-by-step instructions below. Of course, I am always available if something goes wrong and you cannot proceed. On step 2, in particular, which is run on a live system, you might need my assistance.  

## Step 0: Preparation
Below are the links to Panoptes' documentation. It is necessary that you carefully read it as the next steps of the study rely on your understanding of Panoptes.
- [High-level overview of this project.](Introduction)
- [Explanation of the demo system that we've built to showcase the functionality of Panoptes.](Demo System)
- [The core concepts of Panoptes Description Language (PDL).](Core-Concepts)
- [Additional features of PDL related to validity checks.](Validity-Checks)
- [Adding custom algorithms to detect dataset shift.](Algorithm-Runtimes)
- [How Panoptes integrates with the underlying platform (optional).](Platform-Integration)

## Step 1: Evaluating comprehensibility
We would like to evaluate how easy it is for data scientists to understand PDL scripts. For this, we have created an example PDL script targeting the call center demo system and a small questionnaire that checks your understanding of what the script expresses.

If you feel that you have got a good enough understanding of PDL after reading the documentation, please feel free to fill in the questionnaire right away. Otherwise, I am always available for questions/clarifications through email/Teams (panagiotis.kourouklidis@bt.com). 
- [Example script](Example-Script)
- [Questionnaire](Questionnaire)

## Step 2: Evaluating usability
We would also like to evaluate how easy it is for users to use PDL to specify new behaviors. For this, we start by showcasing the behavior of the system with the example script that is already provided.

- [Scenario 1: No dataset shift.](Scenario-1)
- [Scenario 2: Covariate shift.](Scenario-2)

We now ask the participant to modify the script provided such that the system can detect concept shift and respond by sending us an email notification. Participants are free to provide their own algorithms or use the one implemented by us.
- [Scenario 3: Concept shift](Scenario-3)

## Step 3: Evaluating Utility
In this final step, we would like to ask you to estimate the potential of Panoptes to reduce the effort required for the monitoring of ML model performance. You can base your estimate on your interaction with the demo provided and your previous experience with deploying ML models in production.
- [Utility evaluation](Utility-Evaluation)