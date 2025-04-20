# Elastic Stack + Kafka Integration

A complete deployment solution for Elasticsearch, Kibana, Kafka, and Kafka Connect for log ingestion and visualization.

## Architecture

This setup provides a full data pipeline:
- 3-node Elasticsearch cluster with security enabled
- Kibana for visualization and management
- Kafka and Zookeeper for message streaming
- Fluent Bit for log collection
- Kafka Connect for transferring data from Kafka to Elasticsearch
- Log producer for testing

## Prerequisites

- Docker and Docker Compose installed
- 8GB+ RAM recommended for the full stack
- PowerShell (for Windows users) or Bash (for Linux/Mac users)

## Getting Started

### 1. Configure Environment Variables

Create a `.env` file in the project root with the following variables:

```
STACK_VERSION=7.17.10
ELASTIC_PASSWORD=your_secure_password
KIBANA_PASSWORD=your_secure_password
ES_PORT=9200
KIBANA_PORT=5601
CLUSTER_NAME=elasticsearch-cluster
LICENSE=basic
MEM_LIMIT=1073741824
```

### 2. Start the Stack

```bash
# Launch the containers
docker-compose up -d

# To stop and remove all containers
docker-compose down --remove-orphans
```

### 3. Configure the Elasticsearch Connector

After the stack is running, create the Elasticsearch connector:

```powershell
# Windows
Invoke-RestMethod -Method POST -Uri http://localhost:8083/connectors -ContentType 'application/json' -InFile '.\connectors\es-sink.json'

# Linux/Mac
curl -X POST -H "Content-Type: application/json" --data @./connectors/es-sink.json http://localhost:8083/connectors
```

### 4. Check Connector Status

```powershell
# Windows
Invoke-RestMethod -Uri http://localhost:8083/connectors/elasticsearch-sink/status

# Linux/Mac
curl -X GET http://localhost:8083/connectors/elasticsearch-sink/status
```

## Accessing the Services

- **Elasticsearch:** https://localhost:9200 (Username: elastic, Password: from .env)
- **Kibana:** http://localhost:5601 (Username: elastic, Password: from .env)
- **Kafka Connect:** http://localhost:8083

## Viewing Data in Kibana

1. Access Kibana at http://localhost:5601
2. Navigate to Stack Management → Data Views
3. Create a new data view:
   - Name: logs (or match your index pattern)
   - Time field: timestamp
4. Go to Analytics → Discover to view your data
5. Adjust the time range in the upper right corner if needed

## Troubleshooting

### Connector Issues

If the connector fails to start, check:

```powershell
# List available connectors
Invoke-RestMethod -Uri http://localhost:8083/connector-plugins

# View connector details
Invoke-RestMethod -Uri http://localhost:8083/connectors/elasticsearch-sink
```

### Elasticsearch Certificate Issues

If you encounter SSL certificate errors, ensure the certificates are properly mounted in the containers and accessible to Kafka Connect.

### No Data in Kibana

1. Verify the connector is running
2. Check Kafka topics: `docker exec kafka kafka-topics --list --bootstrap-server kafka:9092`
3. Ensure log producer is generating messages: `docker logs log-producer`

## Customizing the Setup

### Adding Your Own Data Source

To add your own data source, modify the `fluent-bit.conf` file or create additional services in the docker-compose.yml file.

### Scaling Elasticsearch

For production environments, increase the memory limits and consider adding more Elasticsearch nodes.

## File Structure

```
.
├── docker-compose.yml
├── .env
├── connectors/
│   └── es-sink.json
├── fluent-bit/
│   └── fluent-bit.conf
└── logs/
```

### Connector Configuration Example

Here's a sample `es-sink.json` file:

```json
{
  "name": "elasticsearch-sink",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "1",
    "topics": "logs",
    "connection.url": "https://es01:9200",
    "connection.username": "elastic",
    "connection.password": "YOUR_ELASTIC_PASSWORD",
    "key.ignore": "true",
    "schema.ignore": "true",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false",
    "connection.ssl.enabled": "true",
    "elastic.https.ssl.enabled": "true", 
    "elastic.security.protocol": "SSL",
    "elastic.https.ssl.certificate.verification": "false"
  }
}
```

## License

This project uses the Elastic Stack with a Basic license.
