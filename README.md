# OpenSearch Neural Search
 
## Example on Amazon OpenSearch Service

 1. By default machine learning (ML) tasks and models only run on specific ML nodes. Running the API command below allows ML tasks and models to run on data nodes

```
PUT /_cluster/settings
{
  "transient":{
    "plugins.ml_commons.only_run_on_ml_node": false
  }
}
```

 2. Upload the BERT model to the OpenSearch domain

```
POST /_plugins/_ml/models/_upload
{
  "name": "huggingface/sentence-transformers/all-MiniLM-L12-v2",
  "version": "1.0.1",
  "model_format": "TORCH_SCRIPT"
}
```

Note down the task id from the API response. This will be used in subsequent steps

3. Get the model id

Using the task id from the previous step, retrieve the model id of the BERT model that was uploaded in the previous step 

```
GET /_plugins/_ml/tasks/{task_id}
```
Note down the model id from the API response. This will be used in subsequent steps

4. Load the model to memory for inference 

```
POST /_plugins/_ml/models/{model_id}/_load
```

Loading the model reads the model’s chunks from the model index and then creates an instance of the model to load into memory

You can check the status of loading the model by passing the task id from the load model API to the API call below. The larger the model the longer it will takes to load it to memory

```
GET /_plugins/_ml/tasks/{task_id}
```

5. Load sample data

6. 
