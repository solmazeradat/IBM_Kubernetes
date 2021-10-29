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

# Exercise 1: Deploy container image 

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

## Step 3 Find the port used on worker nodes

To find the port used in that worker node, examine your new service:

```
kubectl get service questbook
```

In column PORT(S), you can see that internal port 3000 is mapped to the external port <number greater than 30000>. The <nodeport> is always in the 30000 range, it is automatically created, and will be different in your case. Remember this commmand, you will need it later in order to find out ports of your other services.
  
## Step 4 Accessing the service

guestbook is now running on your cluster, and exposed to the internet. We need tofind out where it is accessible. The worker nodes running in the container service get external IP addresses. Run the following command and replace <your-cluster> with the name of your IKS cluster, for example mycluster-free

```  
ibmcloud ks workers --cluster <your-cluster>
```
Note the public IP listed in the <public-IP> column. 
  
## Spep 5 Accessing the applicaiton

Now that you have both the address and the port, you can access the application inthe web browser at <public-IP>:<nodeport>. You have deployed an application to kubernetes.
 
