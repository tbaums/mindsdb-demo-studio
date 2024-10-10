# MindsDB Demo Studio
- Author: Michael Tanenbaum 
- License: Apache 2.0

This repository provides a Docker Compose file for launching a MindsDB demo environment that includes MindsDB, RedPanda (Kafka), Cassandra, and Postgres.

The intention of this demo platform is to provide a **quick** and **reliable** way to launch a MindsDB demo environment that includes a variety of data sources and sinks. 

These demonstrations are intended to be delivered by technical sales/field engineering teams to prospects and customers. This is not intended for production use.

# Roadmap 

- Add end-to-end examples that include front-end applications so that prospects/customers can understand how projects using MindsDB should be integrated into customer operational environments and workflows. For example,
    - a simple web application that demonstrates how to use MindsDB to deliver RAG capabilities to internal teams. 
    - a simple web application that demonstrates incorporating new data coming off a Kafka topic into a RAG model running on MindsDB. 
    - a simple web application that demonstrates how to use MindsDB to deliver predictive maintenance capabilities to internal teams.
- Replace the current Ollama container image with one that has `` pre-loaded. 

# Docker Compose File

This Docker Compose file includes the following services:
- **MindsDB**
- **Ollama** 
- **RedPanda** (single broker deployment)
- **Cassandra** (single node deployment)
- **Postgres**

## Connectivity

Each service has connectivity to each other over a Docker bridge network named `demo`.

You can confirm this network is created by running `docker network ls` and looking for the `demo` network. You can inspect the network by running `docker network inspect demo`.


## FQDN and Endpoints

Where possible, default ports have been preserved.

Each service can be accessed using the following Fully Qualified Domain Names (FQDN) and endpoints:

- **MindsDB**
    - FQDN: `mindsdb.mindsdb-demo-studio_demo`
    - Endpoint: `http://mindsdb.mindsdb-demo-studio_demo:47334`

- **Ollama**
    - FQDN: `ollama.mindsdb-demo-studio_demo`
    - Endpoint: `http://ollama.mindsdb-demo-studio_demo:11434`

- **RedPanda**
    - FQDN: `redpanda.mindsdb-demo-studio_demo`
    - Endpoint: `http://redpanda.mindsdb-demo-studio_demo:9092`

- **Cassandra**
    - FQDN: `cassandra.mindsdb-demo-studio_demo`
    - Endpoint: `http://cassandra.mindsdb-demo-studio_demo:9042`

- **Postgres**
    - FQDN: `postgres.mindsdb-demo-studio_demo`
    - Endpoint: `http://postgres.mindsdb-demo-studio_demo:5432`

## Persistence

Because this is designed to be a quick way to launch demos for MindsDB, each service (cluding MindsDB) does not persist **anything** between container restarts.

## Setup and Launch Instructions


1. Clone this repo and navigate to the repo directory
2. Run: `docker-compose up -d`
3. Navigate to http://localhost:47334


# How to uninstall/clean up the MindsDB Docker Desktop Extension

This demo provides its own copy of the MindsDB Studio container, integrated into the Docker bridge network `demo` that all the other data services in this demo also use.

If you installed the MindsDB Docker Desktop Extension, you will have port conflict with the MindsDB container provided here **even if Docker Desktop and `docker ps -a` show no running containers**! Uninstalling the extension from the Docker Desktop leaves a Docker Nework service running that includes a MindsDB container bound to port 47334. 

Below are the steps for removing these artifacts. 

## Steps
1. Uninstall the MindsDB Docker Desktop extension from Docker Desktop UI.
2. Run `docker network ls` - you should see output similar to the below.
```
➜  mindsdb-demo-studio git:(main) ✗ docker network ls
NETWORK ID     NAME                                                         DRIVER    SCOPE
f20993b36fb1   bridge                                                       bridge    local
a8de80732472   host                                                         host      local
10253c21557c   mindsdb_mindsdb-docker-extension-desktop-extension_default   bridge    local
8c703939d9fd   none                                                         null      local
```
3. Next, run `docker network inspect <NETWORK ID>` with the network ID of the `mindsdb_mindsdb-docker-extension-desktop-extension_default`
4. Find the ID of the container in the "Containers" section of the Docker network JSON specification for the "mindsdb_service" (in this case, the long string starting `017e` below.)
```
        "Containers": {
            "017e000ca05b61efb19a3e5c4fab59561b5506783091f6ee0ef610d24ba70d2e": {
                "Name": "mindsdb_service",
                "EndpointID": "527a0112704aff45842b56fc83bae430ce3fce6b125eddfc73553a9e77fbc15f",
                "MacAddress": "02:42:ac:15:00:02",
                "IPv4Address": "172.21.0.2/16",
                "IPv6Address": ""
            }
        },
```
5. Run `docker stop <Container ID>`
6. Run `docker rm <Container ID>`
7. Run `docker network rm <Network ID>`
8. Confirm network was removed. Run: `docker network ls`.

# Manual Setup Instructions


## Set up Ollama

1. The Ollama container does not come pre-loaded with a model. Here, we'll use `llama3.2`. We must first pull the model into the Ollama container. (Note, this will take a while.)
```
curl http://localhost:11434/api/pull -d '{
  "name": "llama3.2"
}'
```
2. Next, we'll serve the model from the Ollama container. 
3. Last, test that the model is available by running the following command:
```
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Answer in exactly 10 words. Why is the sky blue?",
  "stream": true 
}'
```

## Set up MindsDB

1. Navigate to http://localhost:47334
2. Create a new ML_ENGINE and MODEL using the MindsDB Studio UI and the command below:
```
CREATE ML_ENGINE ollama_engine
FROM ollama;

CREATE MODEL llama_32
PREDICT completion
USING
   engine = 'ollama_engine',   -- engine name as created via CREATE ML_ENGINE
   model_name = 'llama3.2',  -- model run with 'ollama run model-name'
   ollama_serve_url = 'http://ollama.mindsdb-demo-studio_demo:11434';
```
3. Check on the progress of the new model's training by running the following command. The model is ready when the `status` column reads `complete`.:
``` 
select * from mindsdb.models;
```


4. Test the model by running the following command:
```
SELECT text, completion
FROM mindsdb.llama_32 
WHERE text = 'Why is the sky blue?'
USING
   prompt_template = 'Answer using maximum 15 words: {{text}}:';
```

## Set up Postgres