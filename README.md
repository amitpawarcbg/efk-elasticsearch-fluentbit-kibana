**EFK Stack Setup (Elasticsearch, Fluent-bit and Kibana) for Kubernetes
Log Management**

EFK stack is **Elasticsearch**, **Fluent bit** and **Kibana** UI, which
is gaining popularity for Kubernetes log aggregation and management. The
\'**F**\' is EFK stack can be **Fluentd **too, which is like the big
brother of Fluent bit. Fluent bit being a lightweight service is the
right choice for basic log management use case.

## **Why we need EFK Stack?**

Well if you know about Kuberenetes then you must be thinking that you
can use the kubectl logs command to easily check logs for any Kuberneted
pod running. But what if there are 100 pods or even more, in that case
it will be very difficult, on top of this, **Kibana dashboard UI** can
be configured as you want to continuosly monitor logs in runtime, which
makes it easier for someone with no experience of **running linux
commands to check logs**, monitor the Kubernetes cluster and
applications running on it.

If you have a large application with 100 pods running along with **logs
coming in from kubernetes system, docker container**, etc, if you do not
have a** centralised log aggregation and management** system, you will,
sooner or later, regret big time, hence the EFK stack is a good choice.

Also, using Fluent bit we can parse logs from various different input
sources, filter them to add more info. or remove unwanted info, and then
store the data in Elasticsearch.

## **How does it Work?**

![EFK stack setup in
Kubernetes](./media/image1.png){width="6.268055555555556in"
height="4.477083333333334in"}

**Fluent bit is run as a DaemonSet**, which means each node in the
cluster will have one pod for Fluent bit, and it will read logs from
the **/var/log/containers** directory where log files are created for
each Kubernetes namspace.

**Elastcisearch** service runs in a separate pod while **Kibana runs in
a separate pod**. They can be on the same cluster node too, depending
upon the resource availability. But usually both of the them deman high
CPU and memory so their pods get started on different cluster nodes.

The there will be some pods running your applications, which are shown
as **App1**, **App2**, in the above picture.

The Fluent bit service will read logs from these Apps, and push the data
in JSON document format in Elasticsearch, and from there Kibana will
stream data to show in the UI.

## **Step 1: Create a Namespace**

It\'s good practice to create a separate namespace for every functional
unit in Kubernetes as this makes the management of pods running within a
particular namespace easy. To see the existing namespaces, you can use
the following command:

kubectl get namespaces

and you will see the list of existing namespaces:

We will be creating a new namespace with name **kube-logging** for us.
To do so create a new file and name it **kube-logging.yaml** using your
favorite editor like **vim**:

vi kube-logging.yaml

Press **i** to enter the **INSERT** mode and then copy the following
text in it.

kind: Namespace

apiVersion: v1

metadata:

name: kube-logging

To create the namespace using the YAML file created above, run the
following command:

kubectl create -f kube-logging.yaml
