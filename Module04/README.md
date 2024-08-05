# Module 4: Evaluation and Monitoring

In this module, we will learn how to evaluate and monitor our LLM and RAG system.

In the evaluation part, we assess the quality of our entire RAG
system before it goes live.

In the monitoring part, we collect, store and visualize
metrics to assess the answer quality of a deployed LLM. We also
collect chat history and user feedback.

## Homework

See Homework [here](homework/homework.md)

See Homework [Notebook](homework/homework.ipynb)

## 4.1 Introduction to monitoring answer quality 
* Why monitoring LLM systems?
* Monitoring answer quality of LLMs 
* Monitoring answer quality with user feedback
* What else to monitor, that is not covered by this module? 


## 4.2 Offline vs Online (RAG) evaluation
* Modules recap
* Online vs offline evaluation
* Offline evaluation metrics 


## 4.3 Generating data for offline RAG evaluation
Links:

* [notebook](offline-rag-evaluation.ipynb)
* [results-gpt4o.csv](data/results-gpt4o.csv) (answers from GPT-4o)
* [results-gpt35.csv](data/results-gpt35.csv) (answers from GPT-3.5-Turbo)


## 4.4 Offline RAG evaluation: cosine similarity
Content

* A->Q->A' cosine similarity
* Evaluating gpt-4o
* Evaluating gpt-3.5-turbo
* Evaluating gpt-4o-mini

Links:

* [notebook](offline-rag-evaluation.ipynb)
* [results-gpt4o-cosine.csv](data/results-gpt4o-cosine.csv) (answers with cosine calculated from GPT-4o)
* [results-gpt35-cosine.csv](data/results-gpt35-cosine.csv) (answers with cosine calculated from GPT-3.5-Turbo)
* [results-gpt4o-mini.csv](data/results-gpt4o-mini.csv) (answers from GPT-4o-mini)
* [results-gpt4o-mini-cosine.csv](data/results-gpt4o-mini-cosine.csv) (answers with cosine calculated from GPT-4o-mini)


## 4.5 Offline RAG evaluation: LLM as a judge
* LLM as a judge
* A->Q->A' evaluation
* Q->A evaluation


Links:

* [notebook](offline-rag-evaluation.ipynb)
* [evaluations-aqa.csv](data/evaluations-aqa.csv) (A->Q->A evaluation results)
* [evaluations-qa.csv](data/evaluations-qa.csv) (Q->A evaluation results)

## 4.6 Capturing user feedback
> You can see the prompts and the output from claude [here](code.md)

Content

* Adding +1 and -1 buttons
* Setting up a postgres database
* Putting everything in docker compose

```bash
pip install pgcli
pgcli -h localhost -U your_username -d course_assistant -W
```

Links:

* [final code](app/)
* [intermediate code from claude](code.md#46-capturing-user-feedback)

### 4.6.2 Capturing user feedback: part 2 
* adding vector search
* adding OpenAI

Links:

* [final code](app/)
* [intermediate code from claude](code.md#462-capturing-user-feedback-part-2)


## 4.7 Monitoring the system
* Setting up Grafana
* Tokens and costs
* QA relevance
* User feedback
* Other metrics

Links:

* [final code](app/)
* [SQL queries for Grafana](grafana.md)
* [intermediate code from claude](code.md#47-monitoring)

### 4.7.2 Extra Grafana video
* Grafana variables
* Exporting and importing dashboards

Links:

* [SQL queries for Grafana](grafana.md)
* [Grafana dashboard](dashboard.json)


# Notes

* [Notes by slavaheroes](https://github.com/slavaheroes/llm-zoomcamp/blob/homeworks/04-monitoring/notes.md)
