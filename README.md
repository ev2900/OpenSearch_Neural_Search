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

Note down the task id from the API response. This will be used in subsequent steps. After uploading the model you will notice that indexes *.plugins-ml-model* and *.plugins-ml-task* now exist.

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

The sample data for this example is the opensource Amazon product question answer data set. Copy and past the content of the [load_sample_data.json](https://github.com/ev2900/OpenSearch_Neural_Search/blob/main/load_sample_data.json) to the dev tools console of the OpenSearch dashboard. Run the API call. 

This will create an index *nlp_pqa* with a field *question*. We will convert the text in the question field into vector representations - using the BERT model we loaded - in the next steps 

6. 

7. 
