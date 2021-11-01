# IBM_Kubernetes
Working through the IBM learning material to gain an understanding of kubernetes.

video Link: https://www.crowdcast.io/e/operating-kubernetes?utm_source=crowdcast&utm_medium=email&utm_campaign=followers

Presentation Link: **Presentation-1-operating-iks-essentials.pdf**

# Lab 1

Follow the instuction in the file **Lab-1-create-iks-cluster.pdf** to create an IBM kubernetes cluster. 

# Lab 2 

Follow the instuction in the file **Lab-2-iks-essentials.pdf**. The lab objectives are as follows: 

- Understand core concepts of Kubernetes
- Build a Docker image and deploy an application to Kubernetes run by the IBMKubernetes Service (IKS)
- Control application deployments, while minimizing your time with infrastructure management
- Add AI services to extend your app
- Secure and monitor your cluster and app

# Table of contents
1. [Exercise 1: Deploy container image](#Deploy-container-image)
2. [Exercise 2: Using Replicas, Updates and rollback apps](#Using-Replicas-Updates-and-rollback-apps)
3. [Exercise 3: Using Configuration files](#Using-Configuration-files)



# <a name="Deploy-container-image"></a>Exercise 1: Deploy container image 

- A container image for this app is already built and uploaded to DockerHub under the name ibmcom/guestbook:v1.
- When deploying this app, we will be using Kubernetes artifacts such as **deployments**, **pods** and **services**. 
- This video gives a very good overview of kubectl, control master, worker nodes, etc: https://www.youtube.com/watch?v=aSrqRSk43lY
- A Kubernetes deployment specifies where to download the container image from, and how many copies of the container to start. 
- These running containers are called **pods**. 

## Step 1: Start by creating a guestbook deployment:
Setup your shell enviroment by following the instruction https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install#cloud-shell. 

Type the following command in the shell window from your IBM cloud account. 
```
kubectl create deployment guestbook --image=ibmcom/guestbook:v1
```
To check the status
```
kubectl get pods
```
The end result of the create deployment command is not just the pod containing ourapplication containers, but a Deployment resource that manages the lifecycle of thosepods.If you navigate to the Kubernetes dashboard you will see the status of your deployments, pods and Replica sets

![image](https://user-images.githubusercontent.com/11243960/139295242-d026bb0b-276a-4455-a9bf-401d5aed1056.png)

## Step 2: Expose the deployment as a service 

Once the status reads Running, we need to expose that **deployment** as a **service** so we can access it. The guestbook application listens on port 3000. Run:
```
kubectl expose deployment guestbook --type="NodePort" --port=3000
```

This created a new resource in Kubernetes -- a service. 

---
A service and deployment are two essential building blocks for creating an application and accessing it.
---

## Step 3: Find the port used on worker nodes

To find the port used in that worker node, examine your new service:

```
kubectl get service questbook
```
In column PORT(S), you can see that internal port 3000 is mapped to the external port ```<number greater than 30000>```. The ```<nodeport>``` is always in the 30000 range, it is automatically created, and will be different in your case. Remember this commmand, you will need it later in order to find out ports of your other services.
  
## Step 4: Accessing the service

guestbook is now running on your cluster, and exposed to the internet. We need tofind out where it is accessible. The worker nodes running in the container service get external IP addresses. Run the following command and replace ```<your-cluster>``` with the name of your IKS cluster, for example mycluster-free

```  
ibmcloud ks workers --cluster <your-cluster>
```
Note the public IP listed in the ```<public-IP>``` column. 
  
## Step 5 Accessing the application

Now that you have both the address and the port, you can access the application inthe web browser at ```<public-IP>:<nodeport>```. You have deployed an application to kubernetes.

![image](https://user-images.githubusercontent.com/11243960/139424746-48ec0181-af67-4d62-8376-02dd15579f20.png)


# <a name="Using-Replicas-Updates-and-rollback-apps"></a>Exercise 2: Using Replicas, Updates and rollback apps

This part will cover how to update the number of instances a deployment has and how to safely rollout an update of your application on kubernetes.

# Scale apps and Replicas

- A *replica* is a copy of a pod that contains a running service. 
- By having multiple replicas of a pod, you can ensure your deployment has the available resources to handle increasing loadon your application. 
- Multiple replicase also increase availability of your app.

## Step 1: Creating replicas of a pad

```kubectl``` provides a ```scale``` subcommand to change the size of an existingdeployment. Let's increase our capacity from a single running instance of guestbook up to10 instances:

```
kubectl scale --replicas=10 deployment guestbook
```

![image](https://user-images.githubusercontent.com/11243960/139424967-7d23bec2-5dd2-464f-a3a4-d5bc32de9bd3.png)

Kubernetes will matvh the required replica numbers and start 9 new pods with the same configuration as the first. 

## Step 2: viewing the changes 
To see the changes being rolled out run:

```
kubectl rollout status deployment guestbook
```
![image](https://user-images.githubusercontent.com/11243960/139432279-b90ec0d3-5ef3-4d29-9207-7b9faa24b400.png)

## Step 3: check pods are running
Once the rollout has finished, ensure your pods are running:
```
kubectl get pods
```
![image](https://user-images.githubusercontent.com/11243960/139432680-a346c39d-4a30-425f-8696-988faa5e14de.png)

Tip: Another way to improve availability is to use multizone clusters, spreadingyour application over multiple data centers in the same region, as shown in the following diagram:

![image](https://user-images.githubusercontent.com/11243960/139433076-008e44fd-b679-4c13-b403-aa13f7623f55.png)

# Update and Roll Back Apps 

- Kubernetes allows you to do a rolling upgrade of your application to a new container image.
- With this, you can easily update the running image and also easily undo a rollout (roll back) if aproblem is discovered during or after deployment.

### Step 1: Update 
In the previus lab, am image with the tag v1 was used. For the update, we'll use the image with the tag v2. Run the following:
```
kubectl set image deployment/guestbook guestbook=ibmcom/guestbook:v2
```
![image](https://user-images.githubusercontent.com/11243960/139434621-6d7224be-9556-49bc-aed3-9fcf24da82c0.png)

Note: a pod could have multiple containers, each with its own name. Each image can be changed individually or all at once by referring to the name. In the case of our ```guestbook``` Deployment, the container name is also ```guestbook```.

## Step 2: checking rollout status
To check the status of therollout run:
```
kubectl rollout status deployment/guestbook
```
![image](https://user-images.githubusercontent.com/11243960/139441867-f90b27f8-569d-44a0-ba91-54a1308d99ea.png)

## Step 3: Load the applicaiton

Accessing ```<public-IP>:<nodeport>``` in the browser to confirm your new code is active. To verify that you're running "v2" of guestbook, look at the title of the page, it should now be Guestbook - v2. You may need to do a"cache-less" reload of the web-page to refresh the cache -- ```Ctrl+Shift+R``` (on Windows) or ```Cmd+Shift+R``` (on Mac).

![image](https://user-images.githubusercontent.com/11243960/139442372-6ca00b15-f414-45c7-afaf-081683f221fa.png)

Note: to get your IP addresss and port can type the following: 
```
kubectl get svc
ibmcloud ks workers --clusters mycluster-free
```

## Step 4: Undo a rollout

```
kubectl rollout undo deployment guestbook 
```
To check the status of the rollback run:
```
kubectl rollout status deployment/guestbook
```
![image](https://user-images.githubusercontent.com/11243960/139443938-76ad94de-b349-44f5-87bd-b662a03f1e4e.png)

## Step 5: Old and new replicas 

When doing a rollout, you see references to old replicas and new replicas.
- The **old** replicas are the original 10 pods deployed when we scaled the application.
- The **new** replicas come from the newly created pods with the different image.

All of these pods are owned by the deployment. The deployment manages these two sets of pods with a resource called a ```ReplicaSet```.
```
kubectl get replicaset
```
![image](https://user-images.githubusercontent.com/11243960/139444943-cecad713-acd5-46f9-83c6-2912a4eb0edd.png)

## Step 6: Removing services
Before we continue, let's delete the application so we can learn about a different way toachieve the same results. 

Remove ```deployment```: 
```
kubectl delete deployment guestbook
```

Remove ```service```:
```
kubectl delete service guestbook
```

# <a name="Using-Configuration-files"></a>Exercise 3: Using Configuration files 

In this exercise we will learn how to deploy the same guestbook application we deployed in the previous labs, however, instead of using ```kubectl``` command line helper functions we'll be deploying the application using confuguation files. 

The configuation file mechanism allows you to have a more fine-grained control over all of resources being created within the Kubernetes clusters. 


Before working on the application we need to clone a github repo:
```
git clone https://github.com/IBM/guestbook.git
```

The directory has multiple versions of the guestbook as well as the configuration files we'll use to deploy the pieces of the application. 

```
cd guestbook
cd v1
```

# Scale apps natively

Kubernetes can deploy an individual pod to run an application, but when you need to scale it to handle a large number of requests, a ```Deployment``` is the resource you want to use. 

- A Deployment manages a collection of similar pods. When you ask for a specific number of replicas the **Kubernetes Deployment Controller** will attempt to maintain that number of replicas at all times.

- Every Kubernetes object we create contains two nested object fields that govern the object’s configuration: the field ```spec``` and the field ```status```.

- Field ```spec``` defines the desired state of an object (what we want).
- Filed ```status``` shows the current state (what is now).
- Filed ```status``` is provided by the kubernetes system, we don't define it by ourselves. 
- Kubernetes will attepmt to reconcile your desired state with the actual state of the system. 

A configuation file for every object we create, must contain these four fields: 
 - ```apiVersion```
 - ```kind```
 - ```metadata```
 - ```spec```

guestbook-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:  name: guestbook-v1
  labels:
    app: guestbook
    version: "1.0"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
        version: "1.0"
    spec:
      containers:
      - name: guestbook
        image: ibmcom/guestbook:v1
        ports:
        - name: http-server
          containerPort: 3000
```

