### DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER

- In this project, you will build upon your knowledge of Kubernetes architecture, and begin to deploy applications on a K8s cluster. Kubernetes has a lot of moving parts; it operates with several layers of abstraction between your application and host machines where it runs. So many terms, and capabilities that is not realistic to learn it all at once. Hence, you will be introduced to as many concepts as possible, but gradually.

- Within this project we are going to learn and see in action following 

1.  Deployment of software applications using YAML manifest files with following K8s objects:

    *  Pods
    *  ReplicaSets
    *  Deployments
    *  StatefulSets
    *  Services (ClusterIP, NodeIP, Loadbalancer)
    *  Configmaps
    *  Volumes
    * PersistentVolumes
    * PersistentVolumeClaims …and many more

2. Difference between stateful and stateless applications

    * Deploy MySQL as a StatefulSet and explain why

3. Limitations of using manifests directly to deploy on K8s

    * Working with Helm templates, its components and the most important parts – semantic versioning
    * Converting all the .yaml templates into a helm chart


4. Deploying more tools with Helm charts on AWS Elastic `Kubernetes Service (EKS)`

    * Jenkins
        * MySQL
        * Ingress Controllers (Nginx)

    * Cert-Manager
    * Ingress for Jenkins
    * Ingress for the actual application
5. Deploy Monitoring Tools

     * Prometheus
     * Grafana

6. Hybrid CI/CD by combining different tools such as: `Gitlab CICD`, Jenkins. And, you will also be introduced to concepts around `GitOps` using `Weaveworks Flux`.

### Choosing the right Kubernetes cluster set up

- If you need something more robust, suitable for a production workload and with more advanced capabilities such as horizontal scaling of the worker nodes, then you can consider building own Kubernetes cluster from scratch just as you did in Project 21. If you have been able to automate the entire bootstrap using Ansible, you can easily spin up your nodes with Terraform, and configure the cluster with your Ansible configuration scripts.

- Tools responsible for different parts of your applications:

    * Terraform for infrastructure provisioning
    * Ansible for cluster master and worker nodes configuration
    * Kubernetes for deploying your containerized application and orchestrating the deployment

![image of tools](./p22-architecture-tools.PNG)

- Other options will be to leverage a `Managed Service` Kubernetes cluster from public cloud providers such as: `AWS EKS`, `Microsoft AKS`, or `Google Cloud Platform GKE`. 


- However, there is usually strong reasons why organisations with very strict compliance and security concerns choose to build their own Kubernetes clusters. 

- Most of the companies that go this route will mostly use on-premises data centres. When there is need to store data privately due to its sensitive nature, companies will rather not use a public cloud provider. Because, if they do, they have no idea of the physical location of the data centre in which their data is being persisted. Banks and Governments are typical examples of this.

- Some setup options can combine both public and private cloud together. For example, the `master nodes`, `etcd clusters`, and some `worker nodes` that run `stateful applications` can be configured in private datacentres, while `worker nodes` that require heavy computations and `stateless` applications can run in `public clouds`. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.

![image of public and private cloud infra](./Images/p22-Public-Private-cloud-infra.PNG)

###  Deploying the `Tooling app` using Kubernetes objects

- In this section, you will begin to write configuration files for Kubernetes objects (they are usually referred as `manifests`) in the form of files with yaml syntax and deploy them using kubectl console. But first, let us understand what a `Kubernetes object` is.

- `Kubernetes objects` are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

    * What containerized applications are running (and on which nodes)
    * The resources available to those applications
    * The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

- These objects are `"record of intent"` – once you create the object, the Kubernetes system will constantly work to ensure that the object exists. By creating an object, you are effectively telling the Kubernetes system what you want your cluster’s workload to look like; this is your cluster’s desired state.

- To work with Kubernetes objects – whether to create, modify, or delete them – you will need to use the `Kubernetes API`. When you use the `kubectl command-line interface,` for example, the `CLI` makes the necessary Kubernetes `API calls for you`. It is also possible to use `curl` to directly interact with the `Kubernetes API`


### UNDERSTANDING THE CONCEPT

- Let us try to understand a bit more about how the `service object` is able to route traffic to the Pod.

- If you run the below command:

        kubectl get service nginx-service -o wide

- You will get the output similar to this:

        NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
        nginx-service   ClusterIP   10.100.71.130   <none>        80/TCP    4d    app=nginx-pod



- As you already know, the service’s type is `ClusterIP`, and in the above output, it has the IP address of `10.100.71.130` – This IP works just like an `internal loadbalancer`. It accepts requests and forwards it to an IP address of any Pod that has the respective `selector label`. In this case, it is `app=nginx-pod`. If there is more than one Pod with that label, service will distribute the traffic to all theese pods in a Round Robin fashion.

- Now, let us have a look at what the Pod looks like:

        kubectl get pod nginx-pod --show-labels

- Output:


        NAME        READY   STATUS    RESTARTS   AGE   LABELS
        nginx-pod   1/1     Running   0          31m   app=nginx-pod


- Notice that the IP address of the Pod, is NOT the IP address of the server it is running on. Kubernetes, through the implementation of network plugins assigns `virtual IP adrresses to each Pod.


        kubectl get pod nginx-pod -o wide


- Output:

        NAME        READY   STATUS    RESTARTS   AGE   IP               NODE                                              NOMINATED NODE   READINESS GATES
        nginx-pod   1/1     Running   0          57m   172.50.197.236   ip-172-50-197-215.eu-central-1.compute.internal   <none>           <none>
        Therefore, Service with IP 10.100.71.130 takes request and forwards to Pod with IP 172.50.197.236

### Self Side Task:

1. Build the Tooling app Dockerfile and push it to Dockerhub registry

2. Write a Pod and a Service manifests, ensure that you can access the Tooling app’s frontend using port-forwarding feature.

### Expose a Service on a server’s public IP address & static port

Sometimes, it may be needed to directly access the application using the public IP of the server (when we speak of a K8s cluster we can replace ‘server’ with ‘node’) the Pod is running on. This is when the `NodePort service type `comes in handy.

- A Node port service type exposes the service on a static port on the node’s IP address. NodePorts are in the 30000-32767 range by default, which means a NodePort is unlikely to match a service’s intended port (for example, 80 may be exposed as 30080).

- Update the nginx-service yaml to use a NodePort Service.

        apiVersion: v1
        kind: Service
        metadata:
        name: nginx-service
        spec:
        type: NodePort
        selector:
            app: nginx-pod
        ports:
            - protocol: TCP
            port: 80
            nodePort: 30080


- What has changed is:

    * Specified the type of service (Nodeport)
    * Specified the NodePort number to use.


- To access the service, you must:

    * Allow the inbound traffic in your EC2’s Security Group to the NodePort range 30000-32767
    * Get the public IP address of the node the Pod is running on, append the nodeport and access the app through the browser.

_ You must understand that the port number 30080 is a port on the node in which the Pod is scheduled to run. If the Pod ever gets rescheduled elsewhere, that the same port number will be used on the new node it is running on. So, if you have multiple Pods running on several nodes at the same time – they all will be exposed on respective nodes’ IP addresses with a static port number.





### How Kubernetes ensures desired number of Pods is always running?

- When we define a Pod manifest and apply it – we create a Pod that is running until it’s terminated for some reason (e.g., error, Node reboot or some other reason), but what if we want to declare that we always need at least 3 replicas of the same Pod running at all times? 

- Then we must use an ResplicaSet (RS) object – it’s purpose is to maintain a stable set of Pod replicas running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

- Note: In some older books or documents you might find the old version of a similar object – ReplicationController (RC), it had similar purpose, but did not support set-base label selectors and it is now recommended to use ReplicaSets instead, since it is the next-generation RC.

Let us delete our nginx-pod Pod:

        kubectl delete -f nginx-pod.yaml

- Output:

        pod "nginx-pod" deleted

