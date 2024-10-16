# MLflow with S3 Deployment using Docker Compose
(this guide was based on https://github.com/sachua/mlflow-docker-compose)

Guide on how to deploy MLflow with AWS S3 as the artifact store.


## How to start

1. Clone (download) this repository

    ```bash
    git clone https://github.com/benjad/mlflow-docker-s3.git
    ```

2. `cd` into the `mlflow-docker-compose` directory

3. complete de .env file with your AWS credentials. You need  to also update  AWS_S3_BUCKET with the name of your actual bucket.  Remember that the credentials needs to have full access to the S3 repo.

3. Build and run the containers with `docker-compose`

    ```bash
    docker compose up -d --build
    ```

4. Access MLflow UI on http://localhost:5000 , with the initial credentials:  `user=admin password=password`


## Containerization

The MLflow tracking server is composed of 2 docker containers:

* MLflow server
* MySQL database server

## Update admin password

It's recommended to update the default credentials. You can do it using the Python API.

```python
import os
from mlflow.server.auth.client import AuthServiceClient

#default init credential for admin
os.environ["MLFLOW_TRACKING_USERNAME"] = "admin"
os.environ["MLFLOW_TRACKING_PASSWORD"] = "password"

client = AuthServiceClient("http://localhost:5000")

client.update_user_password("admin", "your_new_password")
```

## track a toy model and upload a file (artifact)
Here there is a example of how to log a run on your Mlflow tracking server. You can also upload files to the artifact store, which in our case is located on a S3 bucket. 

```python
import os
from random import random, randint
import mlflow
os.environ["MLFLOW_TRACKING_USERNAME"] ="admin"
os.environ["MLFLOW_TRACKING_PASSWORD"] ="your_new_password"


mlflow.set_tracking_uri("http://localhost:5000")


with mlflow.start_run() as run:
    
    print("Running mlflow_tracking.py")

    mlflow.log_param("param1", randint(0, 100))
    
    mlflow.log_metric("foo", random())
    mlflow.log_metric("foo", random() + 1)
    mlflow.log_metric("foo", random() + 2)

    artifact_uri = mlflow.get_artifact_uri()
    print(f"Artifact uri: {artifact_uri}")

    if not os.path.exists("outputs"):
        os.makedirs("outputs")
    with open("outputs/test.txt", "w") as f:
        f.write("hello world!")
    mlflow.log_artifact("outputs/test.txt")

```


