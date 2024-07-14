## Homework: Vector Search

In this homework, we'll experiemnt with vector with and without Elasticsearch

> It's possible that your answers won't match exactly. If it's the case, select the closest one.


## Q1. Getting the embeddings model

First, we will get the embeddings model `multi-qa-distilbert-cos-v1` from
[the Sentence Transformer library](https://www.sbert.net/docs/sentence_transformer/pretrained_models.html#model-overview)

```bash
from sentence_transformers import SentenceTransformer
embedding_model = SentenceTransformer(model_name)
```

Create the embedding for this user question:

```python
user_question = "I just discovered the course. Can I still join it?"
```

What's the first value of the resulting vector?

* -0.24
* -0.04
* 0.07
* 0.27

*Answer**: 0.07

Once we load the model, we run the `enconde` function:
```python
text_encoded = model.encode("I just discovered the course. Can I still join it?")
text_encoded
```

The output vector is:
```text
array([ 7.82226548e-02, -4.01311405e-02,  3.86135913e-02, -1.78966438e-04,
        8.92347097e-02, -5.04591092e-02, -1.05026569e-02,  3.71055678e-02,
       -4.18713912e-02, ...
       ])
```


## Prepare the documents

Now we will create the embeddings for the documents.

Load the documents with ids that we prepared in the module:

```python
import requests 

base_url = 'https://github.com/DataTalksClub/llm-zoomcamp/blob/main'
relative_url = '03-vector-search/eval/documents-with-ids.json'
docs_url = f'{base_url}/{relative_url}?raw=1'
docs_response = requests.get(docs_url)
documents = docs_response.json()
```

We will use only a subset of the questions - the questions
for `"machine-learning-zoomcamp"`. After filtering, you should
have only 375 documents

** Answer**

## Q2. Creating the embeddings

Now for each document, we will create an embedding for both question and answer fields.

We want to put all of them into a single matrix `X`:

- Create a list `embeddings` 
- Iterate over each document 
- `qa_text = f'{question} {text}'`
- compute the embedding for `qa_text`, append to `embeddings`
- At the end, let `X = np.array(embeddings)` (`import numpy as np`) 

What's the shape of X? (`X.shape`). Include the parantheses. 

**Answer**: (375, 768)

Create 
```python
#created the dense vector using the pre-trained model
embeddings = []
for doc in filtered_documents:
    # Transforming the title into an embedding using the model
    qa_text = f"{doc['question']} {doc['text']}"
     #doc["text_vector"] = model.encode(doc["text"]).tolist()
    embeddings.append(model.encode(qa_text).tolist())

X = np.array(embeddings)
print("X shape: ",X.shape)
```

Every embneddings include 768 values, so the shape is 375x768.


## Q3. Search

We have the embeddings and the query vector. Now let's compute the 
cosine similarity between the vector from Q1 (let's call it `v`) and the matrix from Q2. 

The vectors returned from the embedding model are already
normalized (you can check it by computing a dot product of a vector
with itself - it should return something very close to 1.0). This means that in order
to compute the coside similarity, it's sufficient to 
multiply the matrix `X` by the vector `v`:


```python
scores = X.dot(v)
```

What's the highest score in the results?

- 65.0 
- 6.5
- 0.65
- 0.065

**Answer**: 0.65

Run the code:
```python
scores= X.dot(query_encoded)
print("Highest score: ",np.max(scores))
```


## Vector search

We can now compute the similarity between a query vector and all the embeddings.

Let's use this to implement our own vector search

```python
class VectorSearchEngine():
    def __init__(self, documents, embeddings):
        self.documents = documents
        self.embeddings = embeddings

    def search(self, v_query, num_results=10):
        scores = self.embeddings.dot(v_query)
        idx = np.argsort(-scores)[:num_results]
        return [self.documents[i] for i in idx]

search_engine = VectorSearchEngine(documents=documents, embeddings=X)
search_engine.search(v, num_results=5)
```

If you don't understand how the `search` function work:

* Ask ChatGTP or any other LLM of your choice to explain the code
* Check our pre-course workshop about implementing a search engine [here](https://github.com/alexeygrigorev/build-your-own-search-engine)

(Note: you can replace `argsort` with `argpartition` to make it a lot faster)


## Q4. Hit-rate for our search engine

Let's evaluate the performance of our own search engine. We will
use the hitrate metric for evaluation.

First, load the ground truth dataset:

```python
import pandas as pd

base_url = 'https://github.com/DataTalksClub/llm-zoomcamp/blob/main'
relative_url = '03-vector-search/eval/ground-truth-data.csv'
ground_truth_url = f'{base_url}/{relative_url}?raw=1'

df_ground_truth = pd.read_csv(ground_truth_url)
df_ground_truth = df_ground_truth[df_ground_truth.course == 'machine-learning-zoomcamp']
ground_truth = df_ground_truth.to_dict(orient='records')
```

Now use the code from the module to calculate the hitrate of
`VectorSearchEngine` with `num_results=5`.

What did you get?

* 0.93
* 0.73
* 0.53
* 0.33

**Answer**: 0.93

WE can calculate the Hitrate with the following code:
```python
hitrate=[]
for gt_qa in ground_truth:
    query = model.encode(gt_qa['question'])
    top_docs = search_engine.search(query, num_results=5)
    hitrate+=[1 for doc in top_docs if doc['id']==gt_qa['document']]

print("Hitrate metric: ",len(hitrate)/len(ground_truth))
```
Output:
```text
Hitrate metric:  0.9426229508196722
```

## Q5. Indexing with Elasticsearch

Now let's index these documents with elasticsearch

* Create the index with the same settings as in the module (but change the dimensions)
* Index the embeddings (note: you've already computed them)

After indexing, let's perform the search of the same query from Q1.

What's the ID of the document with the highest score?

**Answer**: ee58a693

First, we create the index for the documents in Elastic Search:
```python
index_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"},
            "id": {"type": "text"},
            "text_vector": {"type": "dense_vector", "dims": 768, "index": True, "similarity": "cosine"},
        }
    }
}

index_name = "ml-course-questions"

es_client.indices.delete(index=index_name, ignore_unavailable=True)
es_client.indices.create(index=index_name, body=index_settings)
```

Then, we can index our documents:
```python
vectors = X[:,].tolist()
index_documents=[]
for doc,vector in zip(filtered_documents, vectors):
    doc['text_vector']=vector
    index_documents.append(doc)
print("Documents to index: ",len(index_documents))
```

Now we can execute the search with the same paratemers:
```python
search_term = "I just discovered the course. Can I still join it?"
vector_search_term = model.encode(search_term)

query = {
    "field": "text_vector",
    "query_vector": vector_search_term,
    "k": 5,
    "num_candidates": 10000, 
}

res = es_client.search(index=index_name, knn=query, source=["text", "section", "question", "course","id"])
res["hits"]["hits"]
```

The doc with the highest score is:
```text
{'_index': 'ml-course-questions',
  '_id': '30V5sJABN9O30PCkwXgQ',
  '_score': 0.82532895,
  '_source': {'question': 'The course has already started. Can I still join it?',
   'course': 'machine-learning-zoomcamp',
   'section': 'General course-related questions',
   'text': 'Yes, you can. You won’t be able to submit some of the homeworks, but you can still take part in the course.\nIn order to get a certificate, you need to submit 2 out of 3 course projects and review 3 peers’ Projects by the deadline. It means that if you join the course at the end of November and manage to work on two projects, you will still be eligible for a certificate.',
   'id': 'ee58a693'}}
```

## Q6. Hit-rate for Elasticsearch

The search engine we used in Q4 computed the similarity between
the query and ALL the vectors in our database. Usually this is 
not practical, as we may have a lot of data.

Elasticsearch uses approximate techniques to make it faster. 

Let's evaluate how worse the results are when we switch from
exact search (as in Q4) to approximate search with Elastic.

What's hitrate for our dataset for Elastic?

* 0.93
* 0.73
* 0.53
* 0.33

**Answer**: 0.93

We run the following code to calculate the Hitrate:
```python
hitrate=[]
for gt_qa in ground_truth:
    query = {
        "field": "text_vector",
        "query_vector": model.encode(gt_qa['question']),
        "k": 5,
        "num_candidates": 10000, 
    }
    res = es_client.search(index=index_name, knn=query, source=["text", "section", "question", "course","id"])
    hitrate+=[1 for doc in res["hits"]["hits"] if doc['_source']['id']==gt_qa['document']]

print("Hitrate metric: ",len(hitrate)/len(ground_truth))
```

Thew output:
```text
Hitrate metric:  0.9426229508196722
```

## Submit the results

* Submit your results here: https://courses.datatalks.club/llm-zoomcamp-2024/homework/hw3
* It's possible that your answers won't match exactly. If it's the case, select the closest one.
