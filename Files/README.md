# Generative AI Pipeline with Pathway and OpenAI

## Overview

This project demonstrates how to create a Generative AI pipeline using Pathway and OpenAI. The application can ingest and process data from multiple sources, such as a local filesystem, Google Drive, or SharePoint, and respond to user queries through a Retrieval-Augmented Generation (RAG) system. The pipeline leverages Pathway’s connectors to continuously track data changes, processes unstructured data, and indexes them using embeddings from OpenAI’s models.

### Demo Video

[View Demo Video](https://drive.google.com/file/d/17A41z-RW65vW2UCvN71TXD3Hs0ZzE-Rs/view?usp=drive_link)  
*Click the link above to watch the demo video.*


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

### Verify Docker Installation

Ensure Docker is installed by running the following command:

```bash
docker --version
```

In order to let the pipeline get updated with each change in local files, you need to mount the folder onto the docker. The following commands show how to do that.

```bash
# Make sure you are in the right directory.
cd yourDirectory #where files are there

# Build the image in this folder
docker build -t rag .

# Run the image, mount the `data` folder into image and expose the port `8000`
docker run -v `%cd%`/data:/app/data -p 8000:8000 qa
```

### Query the application
You will see the logs for parsing & embedding documents in the Docker image logs. (attach below)
Give it a few minutes to finish up on embeddings, you will see `0 entries (x minibatch(es)) have been...` message.
If there are no more updates, this means the app is ready for use!

To test it, let's query the stats:
```bash
Invoke-WebRequest -Uri 'http://localhost:8000/v1/pw_list_documents' `
                  -Method POST `
                  -Headers @{ "accept"="/"; "Content-Type"="application/json" } `
                  -Body '{}'
```
![Docker logs](https://github.com/user-attachments/assets/ba8adb18-3b0a-4dc2-8539-c591b7b13ab5)


### Asking questions to LLM

```bash
Invoke-RestMethod -Method POST `
  -Uri 'http://localhost:8000/v1/pw_ai_answer' `
  -Headers @{ "accept"="/"; "Content-Type"="application/json" } `
  -Body '{"prompt": "What is the author name?"}'
```
#### Following is the Output of my prompt given to LLM app
![Prompt](https://github.com/user-attachments/assets/7b8d8ca6-0135-4d2b-91c4-f78b0ca6a0d9)



#### Note:

More files can be added to the `data` folder or (Google Drive) simply upload your files there.
