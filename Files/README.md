# Generative AI Pipeline with Pathway and OpenAI

## Overview

This project demonstrates how to create a Generative AI pipeline using Pathway and OpenAI. The application can ingest and process data from multiple sources, such as a local filesystem, Google Drive, or SharePoint, and respond to user queries through a Retrieval-Augmented Generation (RAG) system. The pipeline leverages Pathway’s connectors to continuously track data changes, processes unstructured data, and indexes them using embeddings from OpenAI’s models.

### Demo Video

[![Demo Video](https://link_to_demo_gif_or_thumbnail_image)](https://link_to_demo_video)  
*Click the image above to watch the demo video.*

## Purpose

The goal of this project is to create a scalable question-answering system using the capabilities of Pathway, along with models from OpenAI for embedding and language understanding. The application retrieves data from various sources, processes the content into structured formats, and makes it queryable by users through an HTTP API.

This project demonstrates:
- **Pathway** for real-time data processing and change tracking.
- **Langchain** for chaining multiple LLM-powered operations (optional).
- **LlamaIndex** to manage large-scale data retrieval (optional).
- **Ollama** for model inference (optional).
- **OpenAI** for embeddings and language model inference.

The project highlights the use of these technologies for efficient data management and intelligent query handling.

## Features

- Ingest and index data from multiple sources (local, Google Drive, SharePoint).
- Query large datasets using OpenAI embeddings for Retrieval-Augmented Generation (RAG).
- Continuous real-time monitoring of data changes with low-latency processing.
- HTTP API for querying the system, returning intelligent answers from indexed content.

## Folder Structure

This folder contains the following components:

- **`app.py`**: The core application code, written in Python, which leverages Pathway.
- **`app.yaml`**: Configuration file for data sources, the OpenAI LLM model, and the web server. Customize this file to modify the LLM model, switch to a Google Drive data source, or adjust the filesystem directories for indexing.
- **`requirements.txt`**: Lists the pipeline dependencies. Use `pip install -r requirements.txt` to install all necessary packages for running the pipeline locally.
- **`Dockerfile`**: Configuration file for Docker, enabling the pipeline to run inside a container.
- **`.env`**: A simple configuration file where the OpenAI API key should be stored as an environment variable.
- **`data/`**: A folder containing sample files for testing the pipeline.

### Key Steps:
1. **Data Ingestion**: Pathway connectors read files from various sources such as the local drive, Google Drive, and SharePoint. These files are read into a single Pathway Table as binary objects.
2. **Content Parsing**: The binary objects are parsed using the `unstructured` library, and then the parsed content is split into smaller, manageable chunks.
3. **Embeddings Generation**: The chunks are passed to OpenAI's API to generate embeddings, which are vector representations of the text.
4. **Indexing**: The embeddings are indexed using Pathway's machine-learning library, making the data searchable and enabling fast query responses.
5. **Querying**: Users can make simple HTTP requests to the defined endpoints to retrieve information from the indexed data. The system supports querying using RESTful API endpoints.


## Setup Instructions

### Prerequisites

- Docker installed on your system.
- OpenAI API key (for generating embeddings and language model inference).
- Python installed locally (for installing dependencies).

### Step 1: Verify Docker Installation

Ensure Docker is installed by running the following command:

```bash
docker --version
```



## OpenAI API Key Configuration

This example relies on the usage of OpenAI API, which is crucial to perform the embedding part.

**You need to have a working OpenAI key stored in the environment variable OPENAI_API_KEY**.

Please configure your key in a `.env` file by providing it as follows: `OPENAI_API_KEY=sk-*******`. You can refer to the stub file `.env` in this repository, where you will need to paste your key instead of `sk-*******`.

You can also set the key in the `app.py` while initializing the embedder and chat instances as follows;

```python
chat = llms.OpenAIChat(api_key='sk-...', ...)

embedder = embedders.OpenAIEmbedder(api_key='sk-...', ...)
```

## How to run the project

### With Docker

In order to let the pipeline get updated with each change in local files, you need to mount the folder onto the docker. The following commands show how to do that.

```bash
# Make sure you are in the right directory.
cd yourDirectory #where files are there

# Build the image in this folder
docker build -t qa .

# Run the image, mount the `data` folder into image and expose the port `8000`
docker run -v `%cd%`/data:/app/data -p 8000:8000 qa
```

### Query the documents
You will see the logs for parsing & embedding documents in the Docker image logs. 
Give it a few minutes to finish up on embeddings, you will see `0 entries (x minibatch(es)) have been...` message.
If there are no more updates, this means the app is ready for use!

To test it, let's query the stats:
```bash
curl -X 'POST'   'http://localhost:8000/v1/statistics'   -H 'accept: */*'   -H 'Content-Type: application/json'
```

For more information on available endpoints by default, see [API docs](https://pathway.com/solutions/ai-pipelines).

We provide some example `curl` queries to start with.

The general structure is sending a request to `http://{HOST}:{PORT}/{ENDPOINT}`.

Where HOST is the `host` variable you specify in your app configuration. PORT is the `port` number you are running your app on, and ENDPOINT is the specific extension for endpoints. They are specified in the application code, and they are listed with the versioning as `/v1/...`.

Note that, if you are using the Pathway hosted version, you should send requests to `https://...` rather than `http://...` and emit the `:{PORT}` part of the URL.

You need to add two headers, `-H 'accept: */*'   -H 'Content-Type: application/json'`.

Finally, for endpoints that expect data in the query, you can pass it with `-d '{key: value}'` format.

#### Listing inputs
Get the list of available inputs and associated metadata.

```bash
curl -X 'POST'   'http://localhost:8000/v1/pw_list_documents'   -H 'accept: */*'   -H 'Content-Type: application/json'
```

#### Searching in your documents

Search API gives you the ability to search in available inputs and get up-to-date knowledge.
`query` is the search query you want to execute.

`k` (optional) is an integer, the number of documents to be retrieved. Documents in this case means small chunks that are stored in your vector store.

`metadata_filter` (optional) String to filter results with Jmespath query.

`filepath_globpattern` (optional) String to filter results with globbing pattern. For example `"*"` would result in no filter, `"*.docx"` would result in only `docx` files being retrieved.


```bash
curl -X 'POST' \
  'http://0.0.0.0:8000/v1/retrieve' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "What is the start date of the contract?",
  "k": 2
}'
```

#### Asking questions to LLM (With and without RAG)

- Note: The local version of this app does not require `openai_api_key` parameter in the payload of the query. Embedder and LLM will use the API key in the `.env` file. However, Pathway hosted public demo available on the website [website](https://pathway.com/solutions/ai-pipelines/) requires a valid `openai_api_key` to execute the query.

- Note: All of the RAG endpoints use the `model` provided in the config by default, however, you can specify another model with the `model` parameter in the payload to use a different one for generating the response.

For question answering without any context, simply omit `filters` key in the payload and send the following request.

```bash
curl -X 'POST' \
  'http://0.0.0.0:8000/v1/pw_ai_answer' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "prompt": "What is the start date of the contract?"
}'
```

Question answering with the knowledge from files that have the word `Ide` in their paths.
```bash
curl -X 'POST' \
  'http://0.0.0.0:8000/v1/pw_ai_answer' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "prompt": "What is the start date of the contract?",
  "filters": "contains(path, `Ide`)"
}'
```

- Note: You can limit the knowledge to a folder or, to only Word documents by using ```"contains(path, `docx`)"```
- Note: You could also use a few filters separated with `||` (`or` clause) or with `&&` (`and` clause).

You can further modify behavior in the payload by defining keys and values in `-d '{key: value}'`.

If you wish to use another model, specify in the payload as `"model": "gpt-4"`.

For more detailed responses add `"response_type": "long"` to payload.

#### Summarization
To summarize a list of texts, use the following `curl` command.

```bash
curl -X 'POST' \
  'http://0.0.0.0:8000/v1/pw_ai_summary' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "text_list": [
    "I love apples.",
    "I love oranges."
  ]
}'
```

Specifying the GPT model with `"model": "gpt-4"` is also possible.

This endpoint also supports setting different models in the query by default.

To execute similar curl queries as above, you can visit [ai-pipelines page](https://pathway.com/solutions/ai-pipelines/) and try out the queries from the Swagger UI.


#### Adding Files to Index

First, you can try adding your files and seeing changes in the index. To test index updates, simply add more files to the `data` folder.

If you are using Google Drive or other sources, simply upload your files there.
