# **Hyperledger Fabric Certificate Authority on Kubernetes (k8s)**
Deploy a Hyperledger Fabric Certificate Authority using Native Kubernetes (k8s) for Production Hyperledger Fabric Networks Deployed on Cloud Infrastructure.

## **Purpose**
Learn how to configure and deploy a Hyperledger Fabric Certificate Authority on Kubernetes for a production network and manage cryptographic identities without using cryptogen.

## **Key Concepts**
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

# Step-by-Step Tutorial 
## A note about the environment this tutorial was developed in (your results may differ and you may have to troubleshoot accordingly)
### At the time of this update, this tutorial was developed using:
- Mac OS Catalina 10.15.4
- minikube version: v1.9.2
- Docker version 19.03.8

![Alt text](/assets/dockerdesktop.png?raw=true "Docker Desktop About")

### **Step 1:** Clone the repo and cd into the directory
```
git clone https://github.com/denali49/fabric-ca-k8s.git && cd fabric-ca-k8s
```
### **Step 2:** Install Minikube on your local machine or enable it in Docker Desktop
If you run Docker Desktop, there is a setting that allows you to enable Kubernetes in Docker Desktop.

[Install Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

### **Step 3:** Verify minikube installation and that it is running on your local machine

### **Step 4:** Set up the persistent volume claim and provision storage

### **Step 5:** Init the fabric-ca-server and modify the fabric-ca-server-config.yaml file 

### **Step 6:** Deploy the fabric-ca-server and perform identity management tasks