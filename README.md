# Ansible SDK Kafka Connect Operator Proof of Concept

## Introduction

This repository contains a proof of concept for implementing an operator using the Ansible SDK to configure connectors interacting with an external Kafka Connect through its REST API.

> :warning: This project is a proof of concept and is not intended for production use. The code may lack necessary error handling, security features, and robustness required for production environments.

## Install

1. Build and push the operator image

```
make docker-build docker-push IMG="example.com/external-connect-operator:v0.0.1"
```

2. Deploy the operator 

```
make deploy IMG="example.com/external-connect-operator:v0.0.1"
```

For more detailed instructions see the official [Operator SDK documentation](https://sdk.operatorframework.io/docs/)

## Usage

Once the operator has been installed, you can define `KafkaConnector` CR to configure connector on externally managed Kafka Connect cluster. 

```
apiVersion: external.saiello.github.io/v1alpha1
kind: KafkaConnector
metadata:
  labels:
    app.kubernetes.io/name: external-connect-operator
    app.kubernetes.io/managed-by: kustomize
    
    # Use labels to provide any required configuration
    external.saiello.github.io/api-schema: 'http'
    external.saiello.github.io/api-host: 'my-connect-cluster-connect-api.kafka'
    external.saiello.github.io/api-port: '8083'

  name: kafkaconnector-file-source-sample
spec:
  

  # The Class for the Kafka Connector.
  class: org.apache.kafka.connect.file.FileStreamSourceConnector
           
  # The maximum number of tasks for the Kafka Connector.
  tasksMax: 2

  # Automatic restart of connector and tasks configuration.
  autoRestart:

  # The Kafka Connector configuration. The following properties cannot be set: name, connector.class, tasks.max.
  config:
    file: /opt/kafka/LICENSE
    topic: connect-test
    type: source

  # The state the connector should be in. (one of [running, paused, stopped]) Defaults to running.
  state: running
```




## Testing Locally with Molecule and Kind

### Prerequisites

Before you begin, ensure you have the following prerequisites installed:

- Python 3.8+
- Podman
- Kubernetes Cluster (Kind for local development)


### Walktrhough


1. Clone the repository:

```
git clone https://github.com/saiello/external-kafka-connect-operator.git
cd external-kafka-connect-operator
```


2. Create a virtual environment and activate it:
```
python3 -m venv venv
source venv/bin/activate
```

3. Install the dependencies:

```
pip install -r requirements.txt
```

4. Create a kind cluster and install the operator

```
molecule -s kind converge 
```

5. Export kind context

```
kind export kubeconfig --name osdk-test
```

6. Apply the sample CR 

```
kubectl apply -f config/samples/external_v1alpha1_kafkaconnector.yaml
```

7. Verify source connector has been configured and worked as expected
```
kubectl exec -it -n kafka my-cluster-kafka-0 -- ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
```




> :warning: For sake of simplicity here I setup a strimzi managed connect cluster to interact with through the REST API. If you need to manage KafkaConnectors on strimzi connect cluster, please use the builtin KafkaConnector CRD.
