# Kafka on Kubernetes

Before we get into running Kafka on Kubernetes, you need to have some idea about Kubernetes operators. Head over to the [operators topic](../StatefulSets101/operators.md) to get a quick refresher on the concept. The reason we are focused on Kubernetes operators is that we are going to use them to set up the Kafka cluster.

As you can imagine, setting up Kafka from scratch is no simple task and would take developers weeks if not months to set up from scratch. You would need to deploy StatefulSets (since data needs to be persisted) and replicasets. You would also need to configure access for various endpoints and other configurations to set up Kafka. Once that is done, you would need to manually set up various logging and monitoring systems that allow you to monitor each Kafka broker. When a new version of Kafka comes along, you would have to update it yourself. This is all a lot of work.

This is why multiple operators exist that help you set up the cluster with a few quick commands. There are three main operators for this. [Strimzi](https://strimzi.io), which is free and open source, and is what we are going to use now. [Banzai Cloud Operator (Koperator)](https://banzaicloud.com/docs/supertubes/kafka-operator/install-kafka-operator/#manual-install) which is also open source and newer than the other operators. [Confluent Operator](https://docs.confluent.io/operator/current/overview.html#co-long) which is a commercial product. If you are in a large organization, you will likely be using the Confluent version which has extensive support and constantly updating features.

All these operators have the configuration needed to run a Kafka cluster and allow you to simplify what would have taken weeks into a couple of seconds. You can take a look at the scripts Strizmi uses [here](https://strimzi.io/install/latest?namespace=kafka) which should give you some idea of the work that is automated for you. Now, let's get to setting up the operator.

## Lab

If you don't want to manually set up Kafka on your system, there is now a [Terraform config for Kafka](https://github.com/Phantom-Intruder/infrastructure-configs/tree/master/terraform/kafka). You will need Terraform, Helm, Kubectl and a Kubernetes cluster such as [Minikube](https://minikube.sigs.k8s.io/docs/). Note that the cluster should have at least 4GB of memory. If you are using Minikube, start Minikube with this command:

```
minikube start --memory=4096
```

You can then use two simple commands and have the entire lab automatically set up on your machine.

If you would like to manually set up Kafka on your Kubernetes cluster, you can go ahead and follow the rest of the lab. You can use Helm to install the Strimzi chart which will install all the required operators. Start by adding the Helm repo:

```
helm repo add strimzi https://strimzi.io/charts/
```

Then install the chart:

```
helm install strimzi-release strimzi/strimzi-kafka-operator
```

This will start deploying the resources needed to get a Kafka cluster running on your Kubernetes cluster. Setting up all the resources will take time. Keep track of the deployment with:

```
kubectl get po --watch
```

There should be a pod named `strimzi-cluster-operator` which needs to get into a ready state for you to continue. Now, you need to create a Kubernetes resource file specifying what kind of cluster you want. The resource is of a custom `kind`, which is `Kafka`. You can also specify the name, version, replicas, storage type (ephemeral or persistent), etc... Luckily, Strimzi has provided us with a [number of sample yamls](https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/kafka) that we can use without having to set up each resource manually. In our case, we will be creating a persistent cluster with 3 replicas. You can get the confirmation for that [here](https://strimzi.io/examples/latest/kafka/kafka-persistent.yaml). You can also compare it to the ephermeral example they have provided [here](https://strimzi.io/examples/latest/kafka/kafka-persistent.yaml). You will notice that there is a persistent volume claim in the persistent version that is supposed to help retain data.

Once you deploy the resource, it should start creating pods. Use the watch command again to keep track of the pods that get created and wait until they are all ready.

And that's essentially it! You now have a fully functional Kafka cluster running within Kubernetes. Let's test it out.

We will be doing the same thing we did with Kafka running on our local machine: start a producer, produce logs, and view them in the consumer. Open a new terminal window and start the producer with:

```
kubectl run kafka-producer -ti --image=quay.io/strimzi/kafka:0.32.0-kafka-3.3.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic
```

You will note that the command looks similar to a command that is used to start the producer in a local machine. You need to specify the bootstrap-server by setting the broker and the port followed by the topic you will be producing to. Additionally, a Docker image containing Kafka 3.3.1 is used which contains a full instance of Kafka including the scripts used to run the producer and consumer.

Now, let's run the consumer in the same way you ran the producer. Note that you need to run this in a new terminal window.

```
kubectl run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.32.0-kafka-3.3.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
```

Type something in the producer window, and you should see it appear in the consumer.

In order for you to remove the consumer and pod, simply use `ctrl+c` which will stop the consumer/producer and also remove the respective pods.

## Producer in Python

Now that we've look at how the producer can send messages to the consumer using the command line, let's take a look at how you can implement this in code with Python.  What we will be doing is pulling an image from Docker Hub that contains a simple python producer set to run run when the container is started. The Dockerfile and Python code for this image can be found [here](https://github.com/Phantom-Intruder/infrastructure-configs/tree/master/terraform/kafka/image). This image will be used to deploy a pod into your existing Kubernetes cluster.

First, make sure to run the consumer:

```
kubectl run kafka-consumer -n kafka -ti --image=quay.io/strimzi/kafka:0.32.0-kafka-3.3.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-external-bootstrap:9094--topic my-topic --from-beginning
```

If you automated the lab set up with Terraform and have the terraform repo checked out, open a terminal instance in the python-producer folder and run:

```
terraform init
```

```
terraform apply
```

Otherwise you can start the container manually. Use:

```
docker pull ccdockerrgu/pyproducer:1.0
```

This will pull and start the producer which will send messages to the consumer. The code for the Python producer can be found in [this file](https://github.com/Phantom-Intruder/infrastructure-configs/blob/master/terraform/kafka/image/kafka-python.py).

What you've seen so far is the most basic implementation of Strimzi. You could just as easily use this in a production cluster. If you have a huge multi-node cluster, you could use taints and tolerations to run Kafka on dedicated nodes. If you are having a cluster in the cloud with a different node in different regions, you can spread your brokers across those regions for better fault tolerance. If you need to expose Kafka, you could easily do it in a secure manner. All of this is fully supported out of the box with Strimzi.