# **EFK Stack Setup (Elasticsearch, Fluent-bit and Kibana) for Kubernetes Log Management**

EFK stack is  **Elasticsearch** ,  **Fluent bit**  and  **Kibana**  UI, which is gaining popularity for Kubernetes log aggregation and management. The &#39; **F**&#39; is EFK stack can be  **Fluentd ** too, which is like the big brother of Fluent bit. Fluent bit being a lightweight service is the right choice for basic log management use case.

## **Why we need EFK Stack?**

Well if you know about Kuberenetes then you must be thinking that you can use the kubectl logs command to easily check logs for any Kuberneted pod running. But what if there are 100 pods or even more, in that case it will be very difficult, on top of this,  **Kibana dashboard UI**  can be configured as you want to continuosly monitor logs in runtime, which makes it easier for someone with no experience of  **running linux commands to check logs** , monitor the Kubernetes cluster and applications running on it.

If you have a large application with 100 pods running along with  **logs coming in from kubernetes system, docker container** , etc, if you do not have a ** centralised log aggregation and management**  system, you will, sooner or later, regret big time, hence the EFK stack is a good choice.

Also, using Fluent bit we can parse logs from various different input sources, filter them to add more info. or remove unwanted info, and then store the data in Elasticsearch.

## **How does it Work?**

![1590057793-1](https://user-images.githubusercontent.com/88305831/163329055-1aef1f4b-d55e-4613-8893-a0e11470ef88.png)

**Fluent bit is run as a DaemonSet** , which means each node in the cluster will have one pod for Fluent bit, and it will read logs from the  **/var/log/containers**  directory where log files are created for each Kubernetes namspace.

**Elastcisearch**  service runs in a separate pod while  **Kibana runs in a separate pod**. They can be on the same cluster node too, depending upon the resource availability. But usually both of the them deman high CPU and memory so their pods get started on different cluster nodes.

The there will be some pods running your applications, which are shown as  **App1** ,  **App2** , in the above picture.

### The Fluent bit service will read logs from these Apps, and push the data in JSON document format in Elasticsearch, and from there Kibana will stream data to show in the UI.

### **Step 1: Create a Namespace**

It&#39;s good practice to create a separate namespace for every functional unit in Kubernetes as this makes the management of pods running within a particular namespace easy.

We will be creating a new namespace with name kube-logging for us.

# kubectl create -f kube-logging.yaml

# kubectl get ns

### **Step 2: Setup Elasticsearch**

For Elasticsearch we will setup a  **headless service and a statefulset**  which will get attached to this service. A headless service does not perform load balancing or have a static IP. We are making Elasticsearch a headless service because we will setup a 3 node elastic cluster and we want each node to have all the data stored in it, so we don&#39;t want any load balancing. We will get  **3 Elasticsearch pods**  running once we are done with everything, which will ensure high availability.

### **Creating Elasticsearch Service:**

# kubectl create -f elastic-service.yaml

# kubectl get services -n kube-logging

### **Creating the StatefulSet**

Now let&#39;s define the YAML for creating the statefulset for Elasticsearch service. When we define a statefulset, we provide a lot of information like the  **cluster information**  which includes the cluster  **name** , number of  **replicas** ,  **template ** for replica creation, then along with cluster information, we specify which  **Elasticsearch version to be installed** , we provide the  **resources**  like  **CPU ** and  **Memory ** too in the StatefulSet only.

In the YAML file, we have defined the follwoing:

- The Elasticsearch cluster information like cluster name which is  **es-cluster** , the namespace for it which will be  **kube-logging** , name of the service which we defined in the section above, number of replicas as 3, and the template for those replicas which will be app: elasticsearch.
- We have defined the  **container information** , like the  **version of Elasticsearch**  to be setup, which is  **7.2.0**  in this case, then the  **resource**  allocation, CPU and Memory, the  **limit section**  defines the maximum limit and the  **requests section**  defines how much will be used.
- The Port information to define the  **port numbers**  for REST API and inter-node communication.
- Then we have the  **environment variables**  followed by  **init containers**  which is some pre-setup commands run before Elasticsearch app is run, and at last we have defined the  **storage ** to be allocated for Elasticsearch data which we have kept as  **10 GB** , but you can increase it as per your requriements.

# kubectl create -f elastic-statefulset.yaml

# kubectl get pod -n kube-logging

## **Step 3: Setup Kibana**

For Kibana, we will have a  **kibana service**  and a  **deployment to launch one pod**.

We have two YAML files, one for Kibana service and other for Kibana deployment.

# kubectl create -f kibana-service.yaml

# kubectl create -f kibana-deployment.yaml

Now to access the Kibana UI we need to configure Kibana Service type as NodePort.

# kubectl edit svc kibana -n kube-logging

# kubectl get svc kibana -n kube-logging

Check the port on which the service is running as type NodePort.

Now you can access the Kibana Dashboard from you browser.

http://nodeip:port

## **Step 4: Fluent Bit Service**

For Fluent Bit we will have  **5 YAML files**  and apply them using the kubectl command like we did in the above sections. The YAML files will be:

| **YAML File** | **Purpose** |
| --- | --- |
| **fluent-bit-service-account.yaml** | This is used to create a ServiceAccount with name  **fluent-bit**  in the namespace  **kube-logging** , which Fluent Bit pods will use to access the Kubernetes API. |
| --- | --- |
| **fluent-bit-role.yaml** | This creates a ClusterRole which is used to grant the get, list, and watch permissions to fluent-bit service on the Kubernetes resources like the  **nodes** ,  **pods**  and  **namespaces**  objects. |
| **fluent-bit-role-binding.yaml** | This is to bind the ServiceAccount to the ClusterRole created above. |
| **fluent-bit-configmap.yaml** | This is the main file in which we specify the configurations for the Fluent Bit service like Input plugin, Parser, Filter, Output plugin, etc. We have already covered about [Fluent Bit Service and its Configurations](http://www.studytonight.com/post/what-is-fluent-bit-fluent-bit-beginners-guide). |
| **fluent-bit-ds.yaml** | This defines the DaemonSet for Fluent Bit along with Elasticsearch configuration, along with other basic configurations. |

# kubectl create -f fluent-bit-service-account.yaml

# kubectl create -f fluent-bit-role.yaml

# kubectl create -f fluent-bit-role-binding.yaml

# kubectl create -f fluent-bit-configmap.yaml

# kubectl create -f fluent-bit-ds.yaml

Run the following command to see if the daemonset it created or not:

# kubectl get ds -n kube-logging
