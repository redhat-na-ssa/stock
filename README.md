# Predicting stock prices using Long Short-Term Memory (LSTM)

## Prerequisites
- An Openshift 4.11+ cluster
- The Pipelines Operator
- The `oc` and `tkn` command line tools (See the question mark menu in the Openshift UI)
- An Openshift project to work with

### Files and directories
```
├── src                             Python source for data ingestion and model training
├── pipelines                       Tekton pipeline and tasks 
├── notebooks                       Jupyter experimentation
└── requirements.txt                Python dependencies
```

### Pipeline example

Login to Openshift and create a project.

```
PROJ=pipelines-tutorial
oc new-project ${PROJ}
```

Apply the custom tasks and pipeline resources.
```
oc apply -f pipelines/01-ingest-train-task.yaml
oc apply -f pipelines/02-ingest-train-pipeline.yaml
```

### Create a PVC

-> Use the Openshift UI to manually create a storage persistent volume claim (PVC) and 
pass its name in when starting the pipeline below. I called mine `my-pipeline-claim-01`

### Start a pipeline run
```
tkn pipeline start ingest-and-train -w name=shared-workspace,claimName=my-pipeline-claim-01 -p deployment-name=ingest-and-train -p git-url=https://github.com/redhat-na-ssa/stock.git -p IMAGE='image-registry.openshift-image-registry.svc:5000/$(context.pipelineRun.namespace)/ingest-and-train' --use-param-defaults

tkn pipeline start ingest-and-train -w name=shared-workspace,claimName=my-pipeline-claim-01 -p deployment-name=ingest-and-train -p git-url=https://github.com/redhat-na-ssa/stock.git -p IMAGE='image-registry.openshift-image-registry.svc:5000/$(context.pipelineRun.namespace)/ingest-and-train' -p BOB='abc123xyz' -p ACCESS_KEY='access_key' -p SECRET_KEY='secret_key' -p S3_ENDPOINT='minio-route.com' --use-param-defaults
```

### -> Optional: Auto-create a pvc when starting the pipeline. 

This method requires further investigation as the PVCs don't get deleted when the pipeline gets deleted.

```
tkn pipeline start ingest-and-train -w name=shared-workspace,volumeClaimTemplateFile=00-persistent-volume-claim.yaml -p deployment-name=ingest-and-train -p git-url=https://github.com/redhat-na-ssa/stock.git -p IMAGE='image-registry.openshift-image-registry.svc:5000/$(context.pipelineRun.namespace)/ingest-and-train' --use-param-defaults
```

### TODOs
- Integrate s3 storage into the `ingest` and `training` tasks.
  - Ingest the csv file from s3 vs. the yahoo finance service.
  - Save the trained model artifact to s3 storage so the Triton server can find it.

### References
[Data streamer sample](https://github.com/redhat-na-ssa/ml_data_streamer/blob/main/source-eip/src/test/resources/samples/MUFG-1.csv)

[Custom Notebook Builder](https://github.com/redhat-na-ssa/rhods-custom-notebook-example.git)

[Pipeline examples](https://github.com/rh-datascience-and-edge-practice/kubeflow-examples/blob/main/pipelines/11_iris_training_pipeline.py)
