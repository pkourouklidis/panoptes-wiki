Welcome to the Panoptes evaluation study. Thank you for participating! The goal of this study is to evaluate the usability and usefulness of Panoptes in a context that is as realistic as possible under laboratory conditions.

To simplify the process, I have included in this wiki all the information that the participants are going to need. Here is the step-by-step process of this study.

## Step 0: Preparation
To avoid presenting a lot of material during our meeting, I thought it would be good to create some documentation that the participants can read at their own pace. This way we can hopefully have more time. Below are the links to the relevant material.
- [High level overview of this project.](Introduction)
- Explanation of the demo system that we've build to showcase the functionality of Panoptes.
- [The core concepts of Panoptes Description Language (PDL).](Core Concepts)
- [Additional features of PDL related to validity checks.](Validity Checks)
- [Adding custom algorithms to detect dataset shift.](Available Algorithm Runtimes)
- [How Panoptes integrates with the underlying platform (optional).](Platform Integration)

## Step 1: Evaluating comprehensibility
We would like to evaluate how easy it is for experts in the domain of data science to understand PDL. For this we have created an example PDL script targeting the call center demo system and a small questionaire to check that the participants understand what the script defines.
- [Example script](Example Script)
- Questionaire

## Step 2: Evaluating usability
We would also like to evaluate how easy it is for users to use PDL to specify new behaviours. For this, we start by showcasing the behaviour of the system with the example script that is already provided.

- scenario 1: No dataset shift.
- scenario 2: Covariate shift.

We now ask the participant to modify the script provided such that the system can detect concept shift and respond by sending us an email notification. Participants are free to provide their own algorithms or use the one implemented by us.

## Step 3: Evaluating effort-reduction
