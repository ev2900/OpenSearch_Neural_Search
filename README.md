# OpenSearch Neural Search

<img width="275" alt="map-user" src="https://img.shields.io/badge/cloudformation template deployments-22-blue"> <img width="85" alt="map-user" src="https://img.shields.io/badge/views-1077-green"> <img width="125" alt="map-user" src="https://img.shields.io/badge/unique visits-456-green">

The OpenSearch [Neural Search plugin](https://opensearch.org/docs/latest/search-plugins/neural-search/) brings machine learning models to OpenSearch and allows for the transformation of text into vectors during data ingestion. This simplifies vector search use cases.

Having a machine learning model run directly on OpenSearch eliminates the need to pre-process text into vectors before indexing the data on OpenSearch. Using the Neural Search plugin, you can convert text into vector as you index data on OpenSearch.

For an example follow the step in the section below.

## Example of Neural Search on Amazon OpenSearch Service

1. Deploy OpenSearch Domain

If you do not already have an OpenSearch domain to run the example on you can click the button below to deploy a domain

[![Launch CloudFormation Stack](https://sharkech-public.s3.amazonaws.com/misc-public/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=neural-search&templateURL=https://sharkech-public.s3.amazonaws.com/misc-public/OpenSearch_Neural_Search.yaml)

The login information for the OpenSearch dashboard is provided in the CloudFormation outputs

2. By default machine learning (ML) tasks and models only run on specific ML nodes. Running the API command below allows ML tasks and models to run on data nodes

```
PUT /_cluster/settings
{
  "transient":{
    "plugins.ml_commons.only_run_on_ml_node": false
  }
}
```

3. Upload the BERT model to the OpenSearch domain

```
POST /_plugins/_ml/models/_upload
{
  "name": "huggingface/sentence-transformers/all-MiniLM-L12-v2",
  "version": "1.0.1",
  "model_format": "TORCH_SCRIPT"
}
```

Note down the task id from the API response. This will be used in subsequent steps. After uploading the model you will notice that indexes *.plugins-ml-model* and *.plugins-ml-task* now exist.

4. Get the model id

Using the task id from the previous step, retrieve the model id of the BERT model that was uploaded in the previous step

```
GET /_plugins/_ml/tasks/{task_id}
```
Note down the model id from the API response. This will be used in subsequent steps

5. Load the model to memory for inference

```
POST /_plugins/_ml/models/{model_id}/_load
```

Loading the model reads the model’s chunks from the model index and then creates an instance of the model to load into memory

You can check the status of loading the model by passing the task id from the load model API to the API call below. The larger the model the longer it will take to load it to memory

```
GET /_plugins/_ml/tasks/{task_id}
```

6. Create a pipeline

OpenSearch pipelines are used to transform data before it is indexed. We can create a pipeline that uses the BERT model to transform text to embeddings (vector representation of the text) on ingestion

```
PUT _ingest/pipeline/nlp-pipeline
{
  "description": "An example neural search pipeline",
  "processors" : [
    {
      "text_embedding": {
        "model_id": "{model_id}",
        "field_map": {
          "question": "question_vector"
        }
      }
    }
  ]
}
```

7. Create an index that uses the pipeline to convert the *question* field into embeddings using the pipeline

The *question_vector* field in the index will be generated via. the pipeline we created in the previous step. The pipeline will use the BERT model to convert the *question* field into embeddings ie. vector representations of the question text

```
PUT /nlp_pqa
{
  "settings": {
      "index.knn": true,
      "index.knn.space_type": "cosinesimil",
      "default_pipeline": "nlp-pipeline",
      "analysis": {
        "analyzer": {
          "default": {
            "type": "standard",
            "stopwords": "_english_"
          }
        }
      }
  },
  "mappings": {
      "properties": {
          "question_vector": {
              "type": "knn_vector",
              "dimension": 384,
              "method": {
                  "name": "hnsw",
                  "space_type": "l2",
                  "engine": "faiss"
              },
              "store": true
          },
          "question": {
              "type": "text",
              "store": true
          },
          "answer": {
              "type": "text",
              "store": true
          }
      }
  }
}
```

8. Load sample data

The sample data for this example is the opensource Amazon product question answer data set. Copy and paste the content of the [load_sample_data.json](https://github.com/ev2900/OpenSearch_Neural_Search/blob/main/load_sample_data.json) to the dev tools console of the OpenSearch dashboard. Run the API call.

This will load the index *nlp_pqa* with a field *question* and the *question_vector* field will automatically be created via. the pipeline we created in previous steps.

<img width="1000" alt="bulk_load" src="https://github.com/ev2900/OpenSearch_Neural_Search/blob/main/README/bulk_load.png">

9. Search data with neural search

We can search the question vector field using the API call below. We use the model to convert our *query_text* into a vector representation

```
GET /nlp_pqa/_search
{
  "_source": [ "question" ],
  "size": 30,
  "query": {
    "neural": {
      "question_vector": {
        "query_text": "does this work with xbox?",
        "model_id": "{model_id}",
        "k": 30
      }
    }
  }
}
```

10. *Optional* Compare the results of a neural search and a key word based search

OpenSearch offers a search relevancy feature that can help compare the result of two different searches. Navigate to the search relevancy section in the OpenSearch dashboard.

Copy, paste the following into the Query 1 box

```
{
  "_source": [ "question" ],
  "size": 30,
  "query": {
    "neural": {
      "question_vector": {
        "query_text": "does this work with xbox?",
        "model_id": "{model_id}",
        "k": 30
      }
    }
  }
}
```

Copy, paste the following into the Query 2 box

```
{
  "_source": [ "question" ],
  "size": 30,
  "query": {
      "match": {
          "question":"does this work with xbox?"
      }
  }
}
```

Click the search button and compare the results

<img width="1000" alt="search_relevancy" src="https://github.com/ev2900/OpenSearch_Neural_Search/blob/main/README/search_relevancy_1.png">
