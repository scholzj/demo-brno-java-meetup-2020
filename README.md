# Stateful applications on Kubernetes  with the operator pattern

This repository contains the demo from the _Stateful applications on Kubernetes  with the operator pattern_ talk in the [Brno Java Meetup]()
Tento repository obsahuje demo z prednasky _Stateful aplikace na Kubernetes pomocí operátor patternu_ ze [září 2020 pro Brno Java Meetup](https://www.meetup.com/Brno-Java-Meetup/events/273090874/). The slides used in the talk can be found [here](https://docs.google.com/presentation/d/1RS9xnHdFC3Z-pY_SqxXcj7obUO0HoY56t0e2K8PLY6M/edit?usp=sharing).

## Prerequisites

* Install [`kafkacat` utility](https://github.com/edenhill/kafkacat) for sending / receiving Kafka messages

## Demo

### Operator installation

* Create a namespace `myproject`

```
kubectl create ns myproject
```

* Install the operator using the YAML files in this repo:

```
kubectl apply -f 01-operator/ -n myproject
```

* Wait until the operator pod is up and running:

```
kubectl get pods -n myproject
```

### New Kafka cluster

* Check the first simple Kafka custom resource [02-simple-kafka.yaml](./02-simple-kafka.yaml)
* And deploy it:

```
kubectl apply -f 02-simple-kafka.yaml -n myproject
```

* Look at how the pods are being started:

```
kubectl get pods -w -n myproject
```

* Or just wait for the cluster to get ready:

```
kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n myproject
```

* Once it is ready, get the cluster address:

```
kubectl get kafka my-cluster --output=jsonpath='{.status.listeners[?(@.type=="external")].bootstrapServers}{"\n"}' -n myproject
```

* Use the address with `kafkacat` to send and receive messages:

```
kafkacat -b <address> -t test-topic -P
Hello World
Strimzi rulez!
<Ctrl-D>

kafkacat -b <address> -t test-topic -C
...
<Ctrl-D>
```

### Update the cluster

* Check the more complicated custom resource in [03-production-ready-kafka.yaml](./03-production-ready-kafka.yaml)
    * It configures lot more things including security, metrics for monitoring using Prometheus, TLS, etc.
* Apply the changes:

```
kubectl apply -f 03-production-ready-kafka.yaml -n myproject
```

* And watch how they are applied

```
kubectl get pods -w -n myproject
```

### User and Topic operators

* Check the application deployment in [04-users-topics-applications.yaml](./04-users-topics-applications.yaml)
    * Notice the KafkaTopic and KafkaUser resources
    * Check the user of the generated users in the deployments
* Deploy the app with all topics and users all in one go

```
kubectl apply -f 04-users-topics-applications.yaml
```

* Check the secrets created for the users with the TLS authentication certificates
* Check the pod logs to see they work