# Generative AI Pipeline with Pathway and OpenAI

## Overview

This project demonstrates how to create a Generative AI pipeline using [Pathway](https://pathway.com) and OpenAI. The application can ingest and process data from multiple sources, such as a local filesystem, Google Drive, or SharePoint, and respond to user queries through a Retrieval-Augmented Generation (RAG) system. The pipeline leverages Pathway’s connectors to continuously track data changes, processes unstructured data, and indexes them using embeddings from OpenAI’s models.

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

### Data sources

You can configure the data sources in the `config.source` part of the `conf.yaml`.
You can add as many data sources as you want, but the demo supports only three kinds: `local`, `gdrive`, and `sharepoint`. You can have several sources of the same kind, for instance, several local sources from different folders.
The sections below describe the essential parameters that need to be specified for each of those sources.

By default, the app uses a local data source to read documents from the `data` folder.

You can use other kind of data sources using the different [connectors](https://pathway.com/developers/user-guide/connecting-to-data/connectors) provided by Pathway.
To do so, you need to add them in `data_sources` in `app.py`


#### Local Data Source

The local data source is configured by setting the `kind` parameter to `local`.

The section `config` must contain the string parameter `path` denoting the path to a folder with files to be indexed.

#### Google Drive Data Source

The Google Drive data source is enabled by setting the `kind` parameter to `gdrive`.

The section `config` must contain two main parameters:
- `object_id`, containing the ID of the folder that needs to be indexed. It can be found from the URL in the web interface, where it's the last part of the address. For example, the publicly available demo folder in Google Drive has the URL `https://drive.google.com/drive/folders/1cULDv2OaViJBmOfG5WB0oWcgayNrGtVs`. Consequently, the last part of this address is `1cULDv2OaViJBmOfG5WB0oWcgayNrGtVs`, hence this is the `object_id` you would need to specify.
- `service_user_credentials_file`, containing the path to the credentials files for the Google [service account](https://cloud.google.com/iam/docs/service-account-overview). To get more details on setting up the service account and getting credentials, you can also refer to [this tutorial](https://pathway.com/developers/user-guide/connectors/gdrive-connector/#setting-up-google-drive).

Besides, to speed up the indexing process you may want to specify the `refresh_interval` parameter, denoted by an integer number of seconds. It corresponds to the frequency between two sequential folder scans. If unset, it defaults to 30 seconds.

For the full list of the available parameters, please refer to the Google Drive connector [documentation](https://pathway.com/developers/api-docs/pathway-io/gdrive#pathway.io.gdrive.read).

#### Using the Provided Demo Folder

We provide a publicly available folder in Google Drive for demo purposes; you can access it [here](https://drive.google.com/drive/folders/1cULDv2OaViJBmOfG5WB0oWcgayNrGtVs).

A default configuration for the Google Drive source in `config.yaml` is available and connects to the folder: uncomment the corresponding part and replace `SERVICE_CREDENTIALS` with the path to the credentials file.

Once connected, you can upload files to the folder, which will be indexed by Pathway.
Note that this folder is publicly available, and you cannot remove anything: **please be careful not to upload files containing any sensitive information**.

#### Using a Custom Folder

If you want to test the indexing pipeline with the data you wouldn't like to share with others, it's possible: with your service account, you won't have to share the folders you've created in your private Google Drive.

Therefore, all you would need to do is the following:
- Create a service account and download the credentials that will be used;
- For running the demo, create your folder in Google Drive and don't share it.

#### SharePoint Data Source

This data source is the part of commercial Pathway offering. You can try it online in one of the following demos:
- The real-time document indexing pipeline with similarity search, available on the [Hosted Pipelines](https://pathway.com/solutions/ai-pipelines) webpage;
- The chatbot answering questions about the uploaded files, available on [Streamlit](https://chat-realtime-sharepoint-gdrive.demo.pathway.com/).

## How to run the project

### Locally
If you are on Windows, please refer to [running with docker](#With-Docker) section below.

To run locally, change your directory in the terminal to this folder. Then, run the app with `python`.

```bash
cd examples/pipelines/demo-question-answering

python app.py
```

Please note that the local run requires the dependencies to be installed. It can be done with a simple pip command:
`pip install -r requirements.txt`

### With Docker

In order to let the pipeline get updated with each change in local files, you need to mount the folder onto the docker. The following commands show how to do that.

You can omit the ```-v `pwd`/data:/app/data``` part if you are not using local files as a source. 
```bash
# Make sure you are in the right directory.
cd examples/pipelines/demo-question-answering

# Build the image in this folder
docker build -t qa .

# Run the image, mount the `data` folder into image and expose the port `8000`
docker run -v `pwd`/data:/app/data -p 8000:8000 qa
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
