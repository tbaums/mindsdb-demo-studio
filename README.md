# mindsdb-demo-studio

# Docker Compose File

This Docker Compose file includes the following services:
- **MindsDB**
- **RedPanda** (single broker deployment)
- **Cassandra** (single node deployment)
- **Postgres**

## Connectivity

Each service has connectivity to each other over named networks.

## Ports

Where possible, default ports have been preserved.

## Persistence

Because this is designed to be a quick way to launch demos for MindsDB, each service (including MindsDB) does not persist **anything** between container restarts.

## Setup and Launch Instructions


1. Clone this repo and navigate to the repo directory
2. Run:
```bash
docker-compose up -d
`````
3. Navigate to http://localhost:47344