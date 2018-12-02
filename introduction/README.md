# Introduction
Welcome to the workshop, in this first section we'd like to show you around 
Azure Machine Learning Services. At the end of this section you will have a 
basic understanding of what Azure Machine Learning Services is and what it
offers. 

This section is a rather lengthy article on the need for something like
Azure Machine Learning Service and the benefits it offers. Do you prefer 
the slides? You can download them here: [introduction.pdf](introduction.pdf)

## Challenges when using Machine Learning
Building a machine learning model on your own computer requires that you
know how to use Python and a machine learning library. Once you've mastered
those skills it becomes easier to build more models. 

But there's much more to it than building a model. In a typical DevOps project
you need to do much more. For example, you need to make sure you have a clean 
dataset available to the whole team. You need to manage different versions
of your machine learning model. And finally you need a way to deploy your model
to production using automated builds and containers.

To understand how Azure Machine Learning Services can help in a DevOps environment
we need to first understand what challenges you will likely face when you start
to use Machine Learning in your own project.

### Gathering good data is hard
A lot of time is put into gathering, cleaning and preprocessing data. 
Approximately 80% if your time is put into preparing data. 

If you spend a huge amount of time on a dataset you want to make sure you don't
ever have to repeat this effort on someone else's computer. But that's exactly
what is happening at a lot of companies. 

Your data pipeline may be limited in the beginning when you first start to build
machine learning models. But it should be your goal to have an automated data
pipeline as soon as you can.

It saves you a lot of time and reduces the chance that you introduce problems
in your models because you had the wrong dataset or a broken dataset.

### Versioning your models is important
Often when you start to build a machine learning model to solve a particular 
problem you learn a lot along the way about the model and your data. This leads
to improvements in your models. 

It is very typical for data scientists and other non-software engineering people
to keep copies of their models around on their laptops. This leads to problems
because it is very easy to mixup different experiments. It ultimately will
cause problems on production because you deploy the wrong model.

Models should be versioned just like software is. In fact, I believe that you
should version experiments and keep track of different things you've tried.

This way you can go back and take learnings with you as you move forward with
your model. Also it makes it a lot easier to find the best model so you
can deploy that model to production.

### Deploying a model to production is difficult by hand
Software engineers know that deployment should not be done by hand anymore in
2018. Yet, when it comes to deploying machine learning models we somehow still
deploy them by hand. And that causes quite a few problems.

There is no way you can quickly and easily restore an environment if you need
to deploy your models by hand after a disaster struck your production machines.

Also, when you deploy by hand you increase the chance that you deploy broken 
bits to production. I am pretty sure your customers will have an opinion about 
that. 

When you want to recover from disasters quickly and decrease your time to market
you have to use automated deployment and some form of containerization for 
your machine learning models.

## What is Azure Machine Learning Services
Making sure that you have the right dataset, a properly managed model and
setting up automated deployment can be difficult. There are many tools you can
use. Most of them are open source and free to use. But certainly not easy
to set up and manage. It takes a lot of effort.

Azure Machine Learning Services aims to make this easier by offering a combined
toolset to manage datasets, machine learning experiments and model deployments.

### An experiment management tool
Azure Machine Learning Service is first and fore-most an experiment management
solution. You can track experiments from your Python code with just a few lines.

Experiment tracking in Azure Machine Learning Service allows you to keep
track of an experiment for a specific model. You can track metrics, settings
and outputs of different runs of your experiment. 

You can use specialized widgets or the portal to visualize experiments so that 
you can select the best experiments to deploy into production.

### A model management tool
Once you have a succesful experiment that produces a trained model you need
some way to keep track of this model. Azure Machine Learning Service allows you
to register your model so that it is tracked.

Tracking in model management means you can see which experiment produced the 
model that you're using. It also shows you where the model is deployed. This
makes it easier to track problems back to their source.

### A deployment workflow automation tool
Finally Azure Machine Learning Service offers a number of ways to deploy models
to production. You can use Azure Container Instances, Kubernetes or Virtual
Machines to deploy your models.

You need to provide code to make a prediction with your model, but the rest
is taken care of by the Machine Learning Service tools.

## Getting started
Excited to see how it works in practice? 
[Take a look at the first lab](../lab1/README.md).
