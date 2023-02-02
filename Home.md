Welcome to the Panoptes evaluation study. Thank you for participating! The goal of this study is to evaluate the usability and usefulness of Panoptes in a context that is as realistic as possible under laboratory conditions.

To simplify the process, I have included in this wiki all the information that the participants are going to need. Here is the step-by-step process of this study.

It is kindly requested that you read through the steps and try to go as far as you can before our meeting. I have tried to make the process as straightforward as possible. Your time and effort are greatly appreciated!

## Step 0: Preparation
To avoid presenting a lot of material during our meeting, I thought it would be good to create some documentation that the participants can read at their own pace. This way we can hopefully have more time for discussion. Below are the links to the relevant material.
- [High-level overview of this project.](Introduction)
- [Explanation of the demo system that we've built to showcase the functionality of Panoptes.](Demo System)
- [The core concepts of Panoptes Description Language (PDL).](Core Concepts)
- [Additional features of PDL related to validity checks.](Validity Checks)
- [Adding custom algorithms to detect dataset shift.](Algorithm Runtimes)
- [How Panoptes integrates with the underlying platform (optional).](Platform Integration)

## Step 1: Evaluating comprehensibility
We would like to evaluate how easy it is for data scientists to understand PDL scripts. For this, we have created an example PDL script targeting the call center demo system and a small questionnaire to check that the participants understand what the script defines.

If you feel that you have got a good enough understanding of PDL after reading the documentation, please feel free to fill in the questionnaire right away. I am always available for questions/clarifications through email/Teams. 
- [Example script](Example Script)
- [Questionnaire](Questionnaire)

## Step 2: Evaluating usability
We would also like to evaluate how easy it is for users to use PDL to specify new behaviors. For this, we start by showcasing the behavior of the system with the example script that is already provided.

- [Scenario 1: No dataset shift.](Scenario 1)
- [Scenario 2: Covariate shift.](Scenario 2)

We now ask the participant to modify the script provided such that the system can detect concept shift and respond by sending us an email notification. Participants are free to provide their own algorithms or use the one implemented by us.
- [Scenario 3: Concept shift](Scenario 3)

## Step 3: Evaluating Utility
In this final step, we would like to ask you to estimate the potential of Panoptes to reduce the effort required for the monitoring of ML model performance. You can base your estimate on your interaction with the demo provided and your previous experience with deploying ML models in production.
- Utility evaluation form