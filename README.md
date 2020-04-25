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

### **Step 5:** Init the fabric-ca-server and modify the fabric-ca-server-config.yaml file 

### **Step 6:** Deploy the fabric-ca-server and perform identity management tasks