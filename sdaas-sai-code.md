# Custom Machine Learning (ML) Pipeline using Kubeflow Pipelines (KFP) and Google Cloud AI Platform Pipelines

## Introduction
The following is a custom Machine Learning (ML) pipeline that trains and deploys a model using Google's Vertex AI platform. The pipeline consists of two major components:

## Training Job Component
This component trains a model using custom-defined parameters and data. It utilizes a docker container for the training job. It also updates a Firestore collection with job details, such as job ID, job name, and training status. If the training job fails, it updates the Firestore collection to reflect the "Failed" status. If the training job is successful, it updates the Firestore collection to reflect the "Completed" status.

## Model Deployment Job Component
This component deploys the trained model to an endpoint for serving predictions. It creates an endpoint, uploads the model to the endpoint, and deploys the model. It also updates the Firestore collection with model deployment details, such as model ID, endpoint name, and deployment status. If the deployment fails, it updates the Firestore collection to reflect the "Failed" status. If the deployment is successful, it updates the Firestore collection to reflect the "Completed" status.

## Pipeline Compilation and Execution
The pipeline is compiled into a YAML file, which is used to create a pipeline job in Vertex AI. The pipeline job runs asynchronously.

## Benefits
Overall, this pipeline encapsulates the end-to-end process of training and deploying a machine learning model, making it easier to manage and monitor the entire process. It also ensures that all the job details are stored in a Firestore collection, providing an easy way to track and manage the jobs.

## Pipeline Execution
In the main pipeline function, these two components are chained together, meaning the output of the training job (the trained model) is used as the input for the deployment job.

After defining the pipeline, the code compiles it into a YAML file. This file can then be used to create and run the pipeline on Vertex AI Pipelines.

## Summary
In summary, this code automates the process of training a model and deploying it as a service on AI Platform. It also keeps track of the status of these processes in a Firestore collection, which can be useful for monitoring and debugging.