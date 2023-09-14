# OpenSearch Neural Search
 
## Example

 1. By default machine learning (ML) tasks and models can only run on specific ML nodes. Running the API command below we allow tasks and models to also run on data nodes

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

Note down the task id from the API response

3. Get the model id

```
GET /_plugins/_ml/tasks/{task_id}
```

4. Load the model

```
POST /_plugins/_ml/models/{model_id}/_load
```

You can check the status of the loaded model

```
GET /_plugins/_ml/tasks/{task_id}
```

5. 
