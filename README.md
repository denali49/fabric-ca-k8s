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

### **Step 1:** Clone the repo and cd into the directory
```
git clone https://github.com/denali49/fabric-ca-k8s.git && cd fabric-ca-k8s

```
### **Step 2:** Install Minikube [Install Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

### **Step 3:** Verify minikube installation and that it is running on your local machine

### **Step 4:** Set up the persistent volume claim and provision storage

### **Step 5:** Init the fabric-ca-server and modify the fabric-ca-server-config.yaml file 

### **Step 6:** Deploy the fabric-ca-server and perform idenity management tasks