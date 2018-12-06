# Lab 3: Deploy a model using Azure Container Instances
In the previous lab we built a model that can be predict the chance a customer is going to unsubscribe from 
our telecom services. In this lab we will deploy the model to a production environment.

In this lab you will learn:

 * How to create a scoring script for your model
 * How to set up an Azure container instance with your model
 * How to invoke the deployed model

## Step 1: Create a scoring script
Models in Azure Machine Learning Service are deployed as docker containers. We are going to use
a Azure Container Instance to deploy our model. 

To deploy our model in production we need to create a scoring script. The scoring script
is used to make predictions with our model. It needs to load our model and encoders to do so.

Add a new cell to your notebook and include the following code:

```
%%writefile score.py
import pandas as pd
import json
import os
from sklearn.externals import joblib
from sklearn.ensemble import RandomForestClassifier
from azureml.core.model import Model

def init():
    global model, encoders
    
    model_path = Model.get_model_path('customer_churn')
    encoders_path = Model.get_model_path('customer_churn_encoders')
    
    model = joblib.load(model_path)
    encoders = joblib.load(encoders-path)
    

def run(raw_data):
    data = pd.read_json(raw_data).values.reshape(1,-1)
    
    for col, encoder in encoders.items():
        data[col] = encoder.transform(data[col])
    
    y_hat = model.predict_proba(data.values)[1]
    
    return json.dumps({'score': y_hat})
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>. This will
write the scoring script to disk. 

## Step 2: Create the environment file
The deployment logic will need a special file to build a proper python runtime
environment inside the docker container that we will create. 

Add a new cell to the notebook with the following code:

```
from azureml.core.conda_dependencies import CondaDependencies 

myenv = CondaDependencies()
myenv.add_conda_package("scikit-learn")

with open("myenv.yml","w") as f:
    f.write(myenv.serialize_to_string())
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>. This will create
a new anaconda environment file with the scikit-learn package as requirement.

## Step 3: Create the webservice
Now that we have a scoring script and environment file, let's create a new
service based on these files. Add a new cell to the notebook with the following content:

```
from azureml.core.webservice import AciWebservice

aciconfig = AciWebservice.deploy_configuration(cpu_cores=1, 
                                               memory_gb=1, 
                                               tags={"data": "customer_churn",  "method" : "sklearn"}, 
                                               description='Predict customer churn')
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>. 
This will create a new deployment configuration for our model.

Next create another cell with the following content:

```
%%time
from azureml.core.webservice import Webservice
from azureml.core.image import ContainerImage

# configure the image
image_config = ContainerImage.image_configuration(execution_script="score.py", 
                                                  runtime="python", 
                                                  conda_file="myenv.yml")

service = Webservice.deploy_from_model(workspace=ws,
                                       name='customer-churn-svc',
                                       deployment_config=aciconfig,
                                       models=[stored_model, stored_encoders],
                                       image_config=image_config)

service.wait_for_deployment(show_output=True)
```

Execute the cell by pressing <kbd>Ctrl</kbd>+<kbd>Enter</kbd>. 

In this cell we create a new image configuration for our model. This configuration
points to our scoring script and environment file that we created earlier.

We then create a new service based on our configuration. 

**Note** this step takes approximately 6-8 minutes!

## Step 4: Use the webservice
TODO: Include instructions to invoke the service using powershell.

## Summary
In this lab you've learned how to use your model in a production environment using Azure Container Services.
We explored how to create a scoring script and use it to deploy a new image and service for our model.

That concludes the workshop. We hope you enjoyed it!