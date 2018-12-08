# Lab 2: Run an experiment in Azure Machine Learning Services
In the second lab we're going to run an experiment to train a customer churn model.
At the end of this lab you will have learned the following:

 * How to create a new experiment inside an Azure Notebook
 * How to use pandas and numpy to load data
 * How to use scikit-learn to preprocess data for your model
 * How to train a random-forest classifier to predict customer churn
 * How to register a trained model in the machine learning workspace

## About the dataset
This lab uses the Customer churn dataset [originally published by IBM](https://www.ibm.com/communities/analytics/watson-analytics-blog/predictive-insights-in-the-telco-customer-churn-data-set/).

It's a dataset that contains contract data from customers of a telecom company.
Alongside a lot of features that describe the customer, there's a label indicating
whether a customer unsubscribed from services offered by the telecom company.

We're going to use the data to build a model that is capable of predicting customer
churn. 

Predicting when a customer is going to unsubscribe can be relevant for the marketing
department and customer service department. They can use this model to find customers
that are likely to leave. They can use the same model with modified subscription parameters
to see whether a change in subscription is likely to result in the customer staying longer.

## Step 1: Upload the dataset to your notebook environment
Before we can start to build our model we need to make the data available in the notebook
environment. 

Download [customerchurn_train.csv](./data/customerchurn_train.csv) file. And save it to disk.

Within the [Azure notebooks site](https://notebooks.azure.com) open your
project and upload the `customerchurn_train.csv` file so that it becomes available in your notebook.

## Step 2: Create a new experiment in the notebook
Open up the `Experiment.ipynb` file in your Azure Notebooks project and add a new Cell.
You can either select the first cell in the notebook and press <kbd>B</kbd> or choose *Insert* -> *Cell below*.

Copy and paste the following code into the newly created cell:

```
from azureml.core import Run, Experiment

experiment_name = 'customer-churn'
experiment = Experiment(ws, experiment_name)
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.

This will create a new experiment instance that we can use to track metadata and output of our experiment.

## Step 2: Load and preprocess the dataset for the experiment
Now that we have an experiment, let's load the dataset so we can preprocess it.
Create another cell in your notebook and copy and paste the following code into the new cell:

```
import pandas as pd
import numpy as np

df_churn = pd.read_csv('customerchurn_train.csv', na_values=[' '])
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.
This load the dataset from disk and put it into the `df_churn` variable.

You can visualize this dataset by adding a new cell and place the following code in it:

```
df_churn.head()
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.
This will render the first 5 rows of the dataset.

Our model will only work on numeric values. But not all columns in our dataset have numeric values in them.
There are quite a few categorical columns in the dataset that we need to convert first.

We're going to use a small piece of python code just for that. Create a new cell in the notebook.
Now copy and paste the following code into the new cell:

```
from sklearn.preprocessing import LabelEncoder

categorical_cols = [
    'gender',
    'Partner',
    'Dependents',
    'PhoneService',
    'MultipleLines',
    'InternetService',
    'OnlineSecurity',
    'OnlineBackup',
    'DeviceProtection',
    'TechSupport',
    'StreamingTV',
    'StreamingMovies',
    'Contract',
    'PaperlessBilling',
    'PaymentMethod',
]

encoders = {}

for col in categorical_cols:
    encoder = LabelEncoder()
    df_churn[col] = encoder.fit_transform(df_churn[col])
    
    encoders[col] = encoder
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.

Executing this cell will iterate over all categorical columns in the `categorical_cols` list and
convert them to a numeric representation. We keep the encoders in a separate dictionary.
This makes it easier when we need to encode more data later on when we want to make a prediction with our model.

The `TotalCharges` column in the dataset contains empty values. The machine learning model will be unable to use
rows that contain empty values so we need to make sure that the dataset doesn't have any empty cells.

Add a new cell to the notebook with the following code:

```
df_churn['TotalCharges'] = df_churn['TotalCharges'].fillna(df_churn['TotalCharges'].mean())
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.
This will fill any empty cells in the `TotalCharges` column with the mean of the `TotalCharges` column.

## Step 3: Build and train the model
With the data preprocessed, let's take a look at training a model with it.
For this we're going to need to first split our dataset in a training and validation set.

Add a new cell to the notebook and put the following code in the cell:

```
from sklearn.model_selection import train_test_split

X = df_churn.iloc[:, 2:-1].values
y = df_churn.iloc[:, -1:].values.ravel()

X_train, X_test, y_train, y_test = train_test_split(X,y, test_size=0.2)
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.

When you execute the cell, you're first splitting the dataset in a set of features and a set of labels.
Then the dataset gets split in a set training samples and a set of testing samples.

Now that we have a training and test set, let's build the model. Add the following code
to a new cell in your notebook:

```
from sklearn.ensemble import RandomForestClassifier
from sklearn.externals import joblib

n_estimators = 25
classifier = RandomForestClassifier(n_estimators=n_estimators)

run = experiment.start_logging()

model = classifier.fit(X_train, y_train)
accuracy = model.score(X_test, y_test)

run.log('training samples', X_train.shape[0])
run.log('validation samples', X_test.shape[0])
run.log('accuracy', accuracy)
run.log('estimators', n_estimators)

with open('outputs/model.pkl', 'wb') as model_file:
    joblib.dump(classifier, model_file)
    
with open('outputs/encoders.pkl', 'wb') as encoders_file:
    joblib.dump(encoders, encoders_file)

run.upload_file('model.pkl', 'outputs/model.pkl')
run.upload_file('encoders.pkl', 'outputs/encoders.pkl')

run.complete()
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.

First we configure our classifier with a defined number of estimators. We then
start a new run by invoking the `start_logging` method on the experiment.

Once the run is started, we can start the training and validation process.
First we invoke `fit` on the classifier to train it on the training set.

When we've trained the model, we can score it using the test set.

At the end of the run we record the settings and metrics of the model. We also store our models on 
disk and upload them to the machine learning workspace. After that we complete the run.

## Step 4: Visualize the run results
You can visualize run details in your Azure Notebook using a specialized set of widgets.
Add a new cell to the notebook with the following code:

```
from azureml.widgets import RunDetails
RunDetails(run).show()
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.

You can click on the link at the bottom of the widget to explore the output of your experiment on the portal.

**Tip** try modifying the `n_estimators` setting and run the experiment again. This will generate a new run.
Check out the portal for some interesting visualizations of your experiment.

## Step 5: Register your model in the workspace
If you're happy with your model you can register it in the workspace so you can deploy it at a later stage.
Add a new cell to your notebook and include the following code:

```
stored_model = run.register_model(model_name='customer_churn', model_path='model.pkl')
stored_encoders = run.register_model(model_name='customer_churn_encoders', model_path='encoders.pkl')
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>.

First we dump the model and the encoders to disk. Then we register both models with the machine learning workspace.
Navigate to the machine learning workspace on the Azure Portal and explore the models to see what was stored.

## Summary
This concludes the second lab of the workshop. In this lab you've seen how to use scikit-learn and pandas to build
a machine learning model. In this lab you've also learned how to track metrics with experiments
and discovered how to register models with the machine learning workspace.

In the next lab we will explore how to deploy your model on Azure. [Continue to lab 3](../lab3/README.md).