# Churn users analysis using Apache Spark

An Apache Pyspark analysis of churn users, using a user-log of Sparkify music app. 

<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Background" data-toc-modified-id="Background-1"><span class="toc-item-num">1&nbsp;&nbsp;</span>Background</a></span><ul class="toc-item"><li><span><a href="#What-is-a-Churn-user?" data-toc-modified-id="What-is-a-Churn-user?-1.1"><span class="toc-item-num">1.1&nbsp;&nbsp;</span>What is a Churn user?</a></span></li></ul></li><li><span><a href="#About-data" data-toc-modified-id="About-data-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>About data</a></span></li><li><span><a href="#Task" data-toc-modified-id="Task-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Task</a></span></li><li><span><a href="#Workflow" data-toc-modified-id="Workflow-4"><span class="toc-item-num">4&nbsp;&nbsp;</span>Workflow</a></span><ul class="toc-item"><li><span><a href="#Cleaning" data-toc-modified-id="Cleaning-4.1"><span class="toc-item-num">4.1&nbsp;&nbsp;</span>Cleaning</a></span></li><li><span><a href="#Exploring" data-toc-modified-id="Exploring-4.2"><span class="toc-item-num">4.2&nbsp;&nbsp;</span>Exploring</a></span></li><li><span><a href="#Defining-the-Churn" data-toc-modified-id="Defining-the-Churn-4.3"><span class="toc-item-num">4.3&nbsp;&nbsp;</span>Defining the Churn</a></span></li><li><span><a href="#Feature-Engineering" data-toc-modified-id="Feature-Engineering-4.4"><span class="toc-item-num">4.4&nbsp;&nbsp;</span>Feature Engineering</a></span></li><li><span><a href="#Modeling" data-toc-modified-id="Modeling-4.5"><span class="toc-item-num">4.5&nbsp;&nbsp;</span>Modeling</a></span></li></ul></li><li><span><a href="#Results" data-toc-modified-id="Results-5"><span class="toc-item-num">5&nbsp;&nbsp;</span>Results</a></span></li></ul></div>

## Background

Sparkify is music app that offers a paid premium membership along with a free subscription. The company providing the service holds a log of the interactions between each user and the app. This database is a treasure to the marketing team. 

With the right analysis of the log data, we can see the patterns of interactions between our users and the app. Although the app's free tier is great investement for our company, still we would like focus attention on the patterns of bevaviour of the users who bought the premium subscription. 

Studying the patterns of interaction between premium subscription users and our app could help us in two ways:

 - Making the experience of using our service more satisfying
 
 - Studying features of "Churn" users in the aim of decreasing their number.


### What is a Churn user?

In the customer-oriented domains, It's difficult to measure success if you don't measure the inevitable failures, too. While you must aim for 100% of customers to stick with your company since day one until the end of time, that could be unrealistic. That's where customer churn comes in.

**A churn customer is the individual that stopped using your company's product or service**.


## About data

The data is composed of a log of all events or actions taken by the users during the usage of app. Within every instance in the dataset, the following features are recorded:

 - artist: string 
 - auth: string 
 - firstName: string 
 - gender: string 
 - itemInSession: long 
 - lastName: string 
 - length: double 
 - level: string 
 - location: string 
 - method: string 
 - page: string 
 - registration: long 
 - sessionId: long 
 - song: string 
 - status: long 
 - ts: long 
 - userAgent: string 
 - userId: string 


## Task

The aim of the study is to identify a *potential churn user* using the provided database. In the given aim, it would be reasonable to train a supervised learning model to identify a potential churn user based on engineered features from the user log database provided. 

## Workflow

### Cleaning

Before engaging with the learning process using the dataset, a cleaning process is performed. In this process, there are three main issues that needed to be resolved:

 - Some instances in the user log does not have a respective user id associated, either a null value or an empty string. In each of the cases, the instance is dropped from the dataset as it might be redundant for our calculations.
 
 - Some sessions id's are missing as null values. These sessions could be redundant. I have dropped them from the dataset.
 
 - The timestamps provided for registration each user and interaction by each user is corrupted, hence a correction process is performed.
 
The three processes are coded as a function with name of **clean** with takes the original dataframe and returns a clean dataframe with the three issues corrected. The number of rows in the original dataframe was 286500, the number of rows is the resultant dataframe is 278154. The number of features in the original dataframe was 18, now is 20

### Exploring

In this step, deeper understanding of the features and provisional reasons a user to stop using the service is gained. However, within the dataframe, there are not obvious label for a user if they are considered churn or not. Hence, a new feature needs to be created to define users are churn or not.

### Defining the Churn

Before starting labeling users as churn or not, we have to mark the certain interactions that leads for a user to be churn. In our case, interactions with pages like **"Cancellation Confirmation"** or **"Submit Downgrade"** labeled as a churn interaction.

I have created a column within the dataframe labeling instances in the user log as churn/non-churn based on interactions with the churn pages, which are aggregated with maximum value for each user. This step results in labeling all instances of interaction of a user as churn if they have interacted with these pages, otherwise they are not churn users. 

### Feature Engineering

To train a model to identify potential churn user, the dataset has to be user-oriented. A user-oriented dataset would present each instance (row) as a user outlining their respective features that could be used to induce a statistical supervised system to predict potential churn users.

In the feature engineering process, a user-oriented dataframe is created various features are created using SPARK like:

 - Total number of songs listened by each user
 - Total time spent for each user
 - Total number of thumbs-up given by each user
 - Total number of thumbs-down given by each user
 - Number of adds to playlists by each user
 - Length of subscription lifetime of the user
 - Total number of friend add requests sent by each user
 - Gender of the user  
 - Number of help asked by each user
 - Number of rolladvert by each user
 - Average number of songs listened per session by each user
 - Total number of artists the user has listened to
 
Moreover, using vector assembler, features are assembled into a single column, combining the features created previously. The combined columns is split into training and testing sets to prepare for modelling with the target column as "Churn".

Lastly, a standard scaler is intialised and fitted using the training set and transformed the training and testing sets.

At this point, training and testing sets are ready for modelling.

### Modeling

In this step, three supervised learning models are trained and tested for predicting potential churn users.

 - Logistic Regression
 - Support Vector Machine 
 - Gradient Boosted Trees
 
The three models has been trained and tested using two techniques:

 - Paramgrid: for optimizing hyperparamters for each model.
 - cross validation: to find testing the performance of the model on the training set.
 
After trials, it has been proven that the Gradient Boosted Trees are the best performing model on the training and testing sets with a f1 score of 0.65.

## Results

The first and main resultant of the project is a model for predicting potential churn users.

The second result of the project is understanding the difference importances of our created features in predicting if the user is a potential churn or not.

The most important features in predicting a potential churn users are: 

 - Number of rolladvert by each user
 - Length of subscription lifetime of the user
 - Total number of thumbs-up given by each user
