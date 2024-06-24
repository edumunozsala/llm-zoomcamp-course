## Homework: Introduction

In this homework, we'll learn more about search and use Elastic Search for practice. 

## Q1. Running Elastic 

Run Elastic Search 8.4.3, and get the cluster information. If you run it on localhost, this is how you do it:

```bash
curl localhost:9200
```

What's the `version.build_hash` value?

**Answer**:42f05b9372a9a4a470db3b52817899b99a76ee73

First we run Elastich ina  container:
```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

Then we run the previous curl command:
```bash
{
  "name" : "a0f36c828197",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "quHB7dpeSUyWlVk0nW4PWg",
  "version" : {
    "number" : "8.4.3",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "42f05b9372a9a4a470db3b52817899b99a76ee73",
    "build_date" : "2022-10-04T07:17:24.662462378Z",
    "build_snapshot" : false,
    "lucene_version" : "9.3.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```


## Getting the data

Now let's get the FAQ data. You can run this snippet:

```python
import requests 

docs_url = 'https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-intro/documents.json?raw=1'
docs_response = requests.get(docs_url)
documents_raw = docs_response.json()

documents = []

for course in documents_raw:
    course_name = course['course']

    for doc in course['documents']:
        doc['course'] = course_name
        documents.append(doc)
```

Note that you need to have the `requests` library:

```bash
pip install requests
```

## Q2. Indexing the data

Index the data in the same way as was shown in the course videos. Make the `course` field a keyword and the rest should be text. 

Don't forget to install the ElasticSearch client for Python:

```bash
pip install elasticsearch
```

Which function do you use for adding your data to elastic?

* `insert`
* `index`
* `put`
* `add`

**Answer**: index

To index every document into the Elastic Search engine we must use the `ìndex` command:
```python
for doc in tqdm(documents):
    es_client.index(index=index_name, document=doc)
```

## Q3. Searching

Now let's search in our index. 

We will execute a query "How do I execute a command in a running docker container?". 

Use only `question` and `text` fields and give `question` a boost of 4, and use `"type": "best_fields"`.

What's the score for the top ranking result?

* 94.05
* 84.05
* 74.05
* 64.05

Look at the `_score` field.

**Answer**: 84.05

We run the following search:
```python
query="How do I execute a command in a running docker container?"

search_query = {
        "size": 5,
        "query": {
            "bool": {
                "must": {
                    "multi_match": {
                        "query": query,
                        "fields": ["question^4", "text"],
                        "type": "best_fields"
                    }
                }
            }
        }
}

response = es_client.search(index=index_name, body=search_query)
response['hits']
```

And the output:
```
{'total': {'value': 865, 'relation': 'eq'},
 'max_score': 84.050095,
 'hits': [{'_index': 'course-questions',
   '_id': 'IqiEQJAB_YoAhkVNgc2r',
   '_score': 84.050095,
   '_source': {'text': 'Launch the container image in interactive mode and overriding the entrypoint, so that it starts a bash command.\ndocker run -it --entrypoint bash <image>\nIf the container is already running, execute a command in the specific container:\ndocker ps (find the container-id)\ndocker exec -it <container-id> bash\n(Marcos MJD)',
    'section': '5. Deploying Machine Learning Models',
    'question': 'How do I debug a docker container?',
    'course': 'machine-learning-zoomcamp'}},
  {'_index': 'course-questions',
   '_id': 'MKiEQJAB_YoAhkVNW8tS',
   '_score': 75.54128,
   '_source': {'text': 'In case running pgcli  locally causes issues or you do not want to install it locally you can use it running in a Docker container instead.\nBelow the usage with values used in the videos of the course for:\nnetwork name (docker network)\npostgres related variables for pgcli\nHostname\nUsername\nPort\nDatabase name\n$ docker run -it --rm --network pg-network ai2ys/dockerized-pgcli:4.0.1\n175dd47cda07:/# pgcli -h pg-database -U root -p 5432 -d ny_taxi\nPassword for root:\nServer: PostgreSQL 16.1 (Debian 16.1-1.pgdg120+1)\nVersion: 4.0.1\nHome: http://pgcli.com\nroot@pg-database:ny_taxi> \\dt\n+--------+------------------+-------+-------+\n| Schema | Name             | Type  | Owner |\n|--------+------------------+-------+-------|\n| public | yellow_taxi_data | table | root  |\n+--------+------------------+-------+-------+\nSELECT 1\nTime: 0.009s\nroot@pg-database:ny_taxi>',
    'section': 'Module 1: Docker and Terraform',
    'question': 'PGCLI - running in a Docker container',
    'course': 'data-engineering-zoomcamp'}},
  {'_index': 'course-questions',
   '_id': 'UaiEQJAB_YoAhkVNmM5e',
   '_score': 72.08518,
   '_source': {'text': 'If you are trying to run Flask gunicorn & MLFlow server from the same container, defining both in Dockerfile with CMD will only run MLFlow & not Flask.\nSolution: Create separate shell script with server run commands, for eg:\n> \tscript1.sh\n#!/bin/bash\ngunicorn --bind=0.0.0.0:9696 predict:app\nAnother script with e.g. MLFlow server:\n>\tscript2.sh\n#!/bin/bash\nmlflow server -h 0.0.0.0 -p 5000 --backend-store-uri=sqlite:///mlflow.db --default-artifact-root=g3://zc-bucket/mlruns/\nCreate a wrapper script to run above 2 scripts:\n>\twrapper_script.sh\n#!/bin/bash\n# Start the first process\n./script1.sh &\n# Start the second process\n./script2.sh &\n# Wait for any process to exit\nwait -n\n# Exit with status of process that exited first\nexit $?\nGive executable permissions to all scripts:\nchmod +x *.sh\nNow we can define last line of Dockerfile as:\n> \tDockerfile\nCMD ./wrapper_script.sh\nDont forget to expose all ports defined by services!',
    'section': 'Module 4: Deployment',
    'question': 'Running multiple services in a Docker container',
    'course': 'mlops-zoomcamp'}},
  {'_index': 'course-questions',
   '_id': 'QaiEQJAB_YoAhkVNg831',
...
```

## Q4. Filtering

Now let's only limit the questions to `machine-learning-zoomcamp`.

Return 3 results. What's the 3rd question returned by the search engine?

* How do I debug a docker container?
* How do I copy files from a different folder into docker container’s working directory?
* How do Lambda container images work?
* How can I annotate a graph?

**Answer**: How do I copy files from a different folder into docker container’s working directory?

This time we need to run the next commands:
```python
query="How do I execute a command in a running docker container?"

search_query = {
        "size": 3,
        "query": {
            "bool": {
                "must": {
                    "multi_match": {
                        "query": query,
                        "fields": ["question^4", "text"],
                        "type": "best_fields"
                    }
                },
                "filter": {
                    "term": {
                        "course": "machine-learning-zoomcamp"
                    }
                }                
            }
        }
}

response = es_client.search(index=index_name, body=search_query)
```
The output:
```
{'total': {'value': 345, 'relation': 'eq'},
 'max_score': 84.050095,
 'hits': [{'_index': 'course-questions',
   '_id': 'IqiEQJAB_YoAhkVNgc2r',
   '_score': 84.050095,
   '_source': {'text': 'Launch the container image in interactive mode and overriding the entrypoint, so that it starts a bash command.\ndocker run -it --entrypoint bash <image>\nIf the container is already running, execute a command in the specific container:\ndocker ps (find the container-id)\ndocker exec -it <container-id> bash\n(Marcos MJD)',
    'section': '5. Deploying Machine Learning Models',
    'question': 'How do I debug a docker container?',
    'course': 'machine-learning-zoomcamp'}},
  {'_index': 'course-questions',
   '_id': 'QaiEQJAB_YoAhkVNg831',
   '_score': 51.04628,
   '_source': {'text': "You can copy files from your local machine into a Docker container using the docker cp command. Here's how to do it:\nTo copy a file or directory from your local machine into a running Docker container, you can use the `docker cp command`. The basic syntax is as follows:\ndocker cp /path/to/local/file_or_directory container_id:/path/in/container\nHrithik Kumar Advani",
    'section': '5. Deploying Machine Learning Models',
    'question': 'How do I copy files from my local machine to docker container?',
    'course': 'machine-learning-zoomcamp'}},
  {'_index': 'course-questions',
   '_id': 'QqiEQJAB_YoAhkVNhM0H',
   '_score': 49.938507,
   '_source': {'text': 'You can copy files from your local machine into a Docker container using the docker cp command. Here\'s how to do it:\nIn the Dockerfile, you can provide the folder containing the files that you want to copy over. The basic syntax is as follows:\nCOPY ["src/predict.py", "models/xgb_model.bin", "./"]\t\t\t\t\t\t\t\t\t\t\tGopakumar Gopinathan',
    'section': '5. Deploying Machine Learning Models',
    'question': 'How do I copy files from a different folder into docker container’s working directory?',
    'course': 'machine-learning-zoomcamp'}}]}
```


## Q5. Building a prompt

Now we're ready to build a prompt to send to an LLM. 

Take the records returned from Elasticsearch in Q4 and use this template to build the context. Separate context entries by two linebreaks (`\n\n`)
```python
context_template = """
Q: {question}
A: {text}
""".strip()
```

Now use the context you just created along with the "How do I execute a command in a running docker container?" question 
to construct a prompt using the template below:

```
prompt_template = """
You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
Use only the facts from the CONTEXT when answering the QUESTION.

QUESTION: {question}

CONTEXT:
{context}
""".strip()
```

What's the length of the resulting prompt? (use the `len` function)

* 962
* 1462
* 1962
* 2462

**Answer**: 1462

Now, we create a function to generate our prompt following the desired format:

```python
def build_prompt(query, search_results):
    prompt_template = """
You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
Use only the facts from the CONTEXT when answering the QUESTION.

QUESTION: {question}

CONTEXT: 
{context}
""".strip()

    context = ""
    
    for doc in search_results:
        context = context + f"Q: {doc['question']}\nA: {doc['text']}\n\n"    
        
    prompt = prompt_template.format(question=query, context=context).strip()
    return prompt
```

And call this function:
```python
prompt = build_prompt(query, result_docs)

print("Length of the prompt: ", len(prompt))
```
```text
Length of the prompt:  1463
```


## Q6. Tokens

When we use the OpenAI Platform, we're charged by the number of 
tokens we send in our prompt and receive in the response.

The OpenAI python package uses `tiktoken` for tokenization:

```bash
pip install tiktoken
```

Let's calculate the number of tokens in our query: 

```python
encoding = tiktoken.encoding_for_model("gpt-4o")
```

Use the `encode` function. How many tokens does our prompt have?

* 122
* 222
* 322
* 422

Note: to decode back a token into a word, you can use the `decode_single_token_bytes` function:

```python
encoding.decode_single_token_bytes(63842)
```

**Answer**: 322

Run the code:
```python
encoding = tiktoken.encoding_for_model("gpt-4o")
tokens=encoding.encode(prompt)
print('Num tokens: ', len(tokens))
```

## Bonus: generating the answer (ungraded)

Let's send the prompt to OpenAI. What's the response?  

Note: you can replace OpenAI with Ollama. See module 2.

**Answer**:

Just need to run the following code:
```python
response = client.chat.completions.create(
        model='gpt-4o',
        messages=[{"role": "user", "content": prompt}]
)
    
output = response.choices[0].message.content
print(output)
```

## Bonus: calculating the costs (ungraded)

Suppose that on average per request we send 150 tokens and receive back 250 tokens.

How much will it cost to run 1000 requests?

You can see the prices [here](https://openai.com/api/pricing/)

On June 17, the prices for gpt4o are:

* Input: $0.005 / 1K tokens
* Output: $0.015 / 1K tokens

You can redo the calculations with the values you got in Q6 and Q7.

**Answer**: 

For 1000 requests, we produce 150,000 input tokens and 250,000 output tokens. then,

150,000 * 0.005 /1000 = 0.75 

250,000 * 0.015 / 1000= 3.75

**Total: 4.5**
