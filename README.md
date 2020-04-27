# **Hyperledger Fabric Certificate Authority on Kubernetes (k8s)**
Deploy a Hyperledger Fabric Certificate Authority using Native Kubernetes (k8s) for Production Hyperledger Fabric Networks Deployed on Cloud Infrastructure.

## **Purpose**
Learn how to configure and deploy a Hyperledger Fabric Certificate Authority on Kubernetes for a production network and manage cryptographic identities without using cryptogen.

## **Key Concepts**
Note: This tutorial assumes a novice level of experience with Kubernetes and kubectl.  We are going to make it easy enough that if you do not have this experience, you can learn it on the fly.  Please refer to the Kubernetes links throughout this tutorial for additional information if you get stuck, see errors, or have difficulties.

**Kubernetes** (a.k.a. k8s) [Kubernetes Documentation](https://kubernetes.io/)
- minikube (for deployment of a k8s cluster on your local machine for testing and this tutorial) 
- kubectl cli commands and help
- kubernetes persistent volumes
- kubernetes persistent volume claims
- deploying storage

**Docker**

**Hyperledger Fabric Certificate Authority**
 - Fabric CA Server
 - Fabric CA Client
 - Membership Service Provider (MSP)

## Step-by-Step Tutorial 
### At the time of this update, this tutorial was developed using:
- Mac OS Catalina 10.15.4
- minikube version: v1.9.2
- Docker version 19.03.8

![Alt text](/assets/dockerdesktop.png?raw=true "Docker Desktop About")

### **Step 1:** Clone the repo and cd into the directory
```
git clone https://github.com/denali49/fabric-ca-k8s.git && cd fabric-ca-k8s
```
NOTE: All of the commands in this tutorial must be run from this directory or they will not work!

### **Step 2:** Install Minikube on your local machine or enable it in Docker Desktop
If you run Docker Desktop, there is a setting that allows you to enable Kubernetes in Docker Desktop.

![Alt text](/assets/k8sdockerenable.png?raw=true "Docker Desktop Preferences")


![Alt text](/assets/dockerdesktopk8s.png?raw=true "Docker Desktop Enable Kubernetes")

If you do not have Docker Desktop with Kubernetes enabled, install minikube.

[Install Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

In the terminal, run the following command.
```
minikube start
```
Expected output should be similar to this.  NOTE: As of this update, the hyperkit driver was depracated so use the flag 
--driver=virtualbox if you get an error message. Check [here](https://kubernetes.io/docs/tasks/tools/install-minikube/) for details on which vitualization drivers are available.

![Alt text](/assets/minikubestartoutput.png?raw=true "output from minikube start command")

### **Step 3:** Verify minikube installation and that it is running on your local machine
Run the following command in your terminal.
```
kubectl get all
```
Expected output is similar to below, note your ClusterIP will be different.

![Alt text](/assets/minikubeconfirm.png?raw=true "output from `kubectl get all` command")


### **Step 4:** Set up the persistent volume claim and provision storage
Now run the following kubectl command in the terminal to provision a persistent volume claim (PVC).
```
kubectl apply -f setup-pvc.yaml
```
The expected output is similar to below.

![Alt text](/assets/createpvcexpectedoutput.png?raw=true "output from `kubectl get all` command")

Now run the following command in the terminal to confirm your persistent volume claim has been set up.
```
kubectl get pvc
```
The expected output from the above command should be similar to:

![Alt text](/assets/getpvcexpectedoutput.png?raw=true "output from `kubectl get all` command")

In the terminal, execute the following command to set up a storage volume that is connected to the PVC we created in the previous step.  This is one method of persisting data in the cloud even if your Kubernetes pods restart.  Important Note: Data will NOT persist if you delete the persistent volume claim!  If you plan to start over and you run `minikube delete` your persistent volume and persistent volume claim will be deleted!

```
kubectl apply -f redis-storage.yaml
```
Expected output after running the above command is similar to below.

![Alt text](/assets/redispodcreated.png?raw=true "storage pod created")

In the terminal, run the following command to check the status of the pod.
```
kubectl get pod
```
Expected output after running the above command is similar to below.

![Alt text](/assets/redispodrun.png?raw=true "redis pod status")

### **Step 5:** Init the fabric-ca-server and modify the fabric-ca-server-config.yaml file 
In the terminal, execute the following command to run a kubernetes job that will 'init' the Fabric CA Server and generate a template file that we can customize.  
```
kubectl apply -f fabric-ca-server-initJob.yaml
```
To check the status of the kubernetes job, execute the following command:
```
kubectl get pod
```
If successful, the job should indicated that it completed with an output similar to below.

![Alt text](/assets/initjobcompletion.png?raw=true "Init job completion status")

Note that a Kubernetes Job runs to completion if successful, and a deployment stays running.  

Next we are going to copy the fabric-ca-server-config.yaml file from the container to our local machine, modify it, then copy it back to the container so that when we start the server our customized variables will be read.  To do this, we use a `kubectl cp` command, specifying the container and location of the target file in the container.
Note the convention of the command structure is "pod-name:path-to-target-file" followed by a space, then the target location you would like to copy the file to. 
```
kubectl cp redis:/data/redis/hyperledger/fabric-ca/k8s/fabric-ca-server-config.yaml $PWD/fabric-ca-server-config.yaml
```
Edit the CSR section of the fabric-ca-server-config.yaml file by changing the State (S) from "North Carolina" to "Texas" and the Organization (O) to Hyperchain Labs and the Organizational Unit (OU) to Energy.  Next, run the following command in the terminal to copy our modified file back to the container so we can use it to start the Fabric CA Server.
```
kubectl cp $PWD/fabric-ca-server-config.yaml redis:/data/redis/hyperledger/fabric-ca/k8s/fabric-ca-server-config.yaml
```
Next, we need to exec into the file container and delete the 'ca-cert' file and the 'msp' directory located at 'redis:/data/redis/hyperledger/fabric-ca/k8s/' so when we start the fabric-ca-server our certs are regenerated using our new custom variables in the fabric-ca-server-config.yaml file we just copied into the container.  In the terminal, exec into the container by running the following command:
```
kubectl exec -it redis -- /bin/bash
```
Once inside the container, change directory to our target location so we can delete the file and directory.
```
cd /data/redis/hyperledger/fabric-ca/k8s/
```
Then delete the ca-cert.pem file and the msp directory by running the following command inside the container:
```
rm ca-cert.pem && rm -rf msp
```
Leave the container by running the following command while still inside the container:
```
exit
```
### **Step 6:** Deploy the fabric-ca-server and perform identity management tasks
