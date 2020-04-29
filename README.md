# **Hyperledger Fabric Certificate Authority on Kubernetes (k8s)**

## Watch this tutorial on [YouTube](https://youtu.be/PbMxqH6bNB8) before you do it on your local machine.

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

![The Docker Desktop About window print](/assets/dockerdesktop.png?raw=true "Docker Desktop About")

## **Step 1:** Clone the repo and cd into the directory
```
git clone https://github.com/denali49/fabric-ca-k8s.git && cd fabric-ca-k8s
```
NOTE: All of the commands in this tutorial must be run from this directory or they will not work!

## **Step 2:** Install Minikube on your local machine or enable it in Docker Desktop
If you run Docker Desktop, there is a setting that allows you to enable Kubernetes in Docker Desktop.

![Docker Desktop Preferences Screenshot](/assets/k8sdockerenable.png?raw=true "Docker Desktop Preferences")


![Docker Desktop Kubernetes Enable button in Docker Desktop Preferences Screenshot](/assets/dockerdesktopk8s.png?raw=true "Docker Desktop Enable Kubernetes")

If you do not have Docker Desktop with Kubernetes enabled, install minikube.

[Install Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

In the terminal, run the following command.
```
minikube start
```
Expected output should be similar to this.  ***NOTE: As of this update, the hyperkit driver was depracated so use the flag --driver=virtualbox if you get an error message.*** Check [here](https://kubernetes.io/docs/tasks/tools/install-minikube/) for details on which vitualization drivers are available.

![Screenshot of output from the minikube start command](/assets/minikubestartoutput.png?raw=true "output from minikube start command")

## **Step 3:** Verify minikube installation and that it is running on your local machine
Run the following command in your terminal.
```
kubectl get all
```
Expected output is similar to below, note your ClusterIP will be different.

![Screenshot of output from the kubectl get all command](/assets/minikubeconfirm.png?raw=true "output from `kubectl get all` command")


## **Step 4:** Set up the persistent volume claim and provision storage
Now run the following kubectl command in the terminal to provision a persistent volume claim (PVC).
```
kubectl apply -f setup-pvc.yaml
```
The expected output is similar to below.

![Screeshot of expected output setup-pvc.yaml](/assets/createpvcexpectedoutput.png?raw=true "output from command")

Now run the following command in the terminal to confirm your persistent volume claim has been set up.
```
kubectl get pvc
```
The expected output from the above command should be similar to:

![Screenshot of kubectl get pvc output](/assets/getpvcexpectedoutput.png?raw=true "output from `kubectl get pvc` command")

In the terminal, execute the following command to set up a storage volume that is connected to the PVC we created in the previous step.  This is one method of persisting data in the cloud even if your Kubernetes pods restart.  ***Important Note: Data will NOT persist if you delete the persistent volume claim!  If you plan to start over and you run `minikube delete` your persistent volume and persistent volume claim will be deleted!***

```
kubectl apply -f redis-storage.yaml
```
Expected output after running the above command is similar to below.

![Screenshot of pod creation terminal output](/assets/redispodcreated.png?raw=true "storage pod created")

In the terminal, run the following command to check the status of the pod.
```
kubectl get pod
```
Expected output after running the above command is similar to below.

![Screenshot of redis pod status](/assets/redispodrun.png?raw=true "redis pod status")

## **Step 5:** Init the fabric-ca-server and modify the fabric-ca-server-config.yaml file 
In the terminal, execute the following command to run a kubernetes job that will 'init' the Fabric CA Server and generate a template file that we can customize.  
```
kubectl apply -f fabric-ca-server-initJob.yaml
```
To check the status of the kubernetes job, execute the following command:
```
kubectl get pod
```
If successful, the job should indicated that it completed with an output similar to below.

![Screenshot of init job completion status](/assets/initjobcompletion.png?raw=true "Init job completion status")

Note that a Kubernetes Job runs to completion if successful, and a deployment stays running.  

Next we are going to copy the fabric-ca-server-config.yaml file from the container to our local machine, modify it, then copy it back to the container so that when we start the server our customized variables will be read.  To do this, we use a `kubectl cp` command, specifying the container and location of the target file in the container.
Note the convention of the command structure is "***pod-name:path-to-target-file***" followed by a space, then the target location you would like to copy the file to. 
```
kubectl cp redis:/data/redis/hyperledger/fabric-ca/k8s/fabric-ca-server-config.yaml $PWD/fabric-ca-server-config.yaml
```
Edit the CSR section of the fabric-ca-server-config.yaml file by changing the State (S) from ***"North Carolina"*** to ***"Texas"*** and the Organization (O) to ***Hyperchain Labs*** and the Organizational Unit (OU) to ***Energy*** and then save your changes.  Next, run the following command in the terminal to copy our modified file back to the container so we can use it to start the Fabric CA Server.
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
## **Step 6:** Deploy the fabric-ca-server and perform identity management tasks
Now we are ready to start our Fabric CA Server and interact with it by runnning the following command in the terminal:
```
kubectl apply -f fabric-ca-deployment.yaml
```
Then make sure it is running by executing:
```
kubectl get pod
```
You can then get the logs of the CA Server by running:

kubectl logs [paste your pod name from the output of the get pod command, paste it here, and hit enter]
So in my case, it was 'kubectl logs fabric-ca-k8s-696566c87f-xz9hx'.  Make sure you copy the name of the fabric-ca-k8s deployment that is 'running' and ***NOT*** the fabric-ca-k8s job that is 'completed'.

Your the last few lines of the log indicating successful server start should be similar to this.
![Screenshot of successful server start](/assets/successlog.png?raw=true "Successful Fabric CA Server start")

Near the top of the log output, you should see the custom values you entered into the fabric-ca-server-config.yaml file.
![Screenshot of new config values after edit](/assets/configeditoutput.png?raw=true "New config values")

Now we are ready to interact with the Fabric CA Server.  First we will 'enroll' the CA admin that was listed in the registry section of the fabric-ca-server-config.yaml file that we modified.  We could have changed it to something else, but note that it was admin:adminpw.  Since this identity was 'registered' automatically by the start up of the server reading from the config file, we simply need to enroll the admin identity.  

For all other idenities we wish to add, we will need to register them first, then enroll them.  For more information on identity management, please see the [Hyperledger Fabric CA Server and Client documentation](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/users-guide.html).

Let's get started by getting into the fabric CA container and exporting some environment variables that our client will need.  Find out the pod name of the running pod for the CA by entering the following command:
```
kubectl get pod
```
Next, copy the full pod name because we are going to paste it into our next command when we do `kubectl exec -it [YOUR POD NAME PASTED HERE] -- /binbash`.
In my case ***(for reference only, do not copy and paste this!!)*** it was:
```
kubectl exec -it fabric-ca-k8s-696566c87f-xz9hx -- /bin/bash
```
***Again, make sure you copy the running pod and NOT the completed job as they have similar names.***

Once you are in the container, make a directory for where we want to store our certs, then export the FABRIC_CA_CLIENT_HOME environment variable that points to this location, then enroll the admin identity that was registered during the server start.
***NOTE: If TLS were enabled, you would also have to export the location of the FABRIC_CA_CLIENT_TLS_CERTFILES***
```
mkdir -p /shared/artifacts/org1/ca/admin
export FABRIC_CA_CLIENT_HOME=/shared/artifacts/org1/ca/admin
fabric-ca-client enroll -d -u http://admin:adminpw@0.0.0.0:7054
```
You should see an output similar to this:

![Screenshot of output from enroll admin command](/assets/enrolladminoutput.png?raw=true "Enroll admin ouput")

Now that you have a running Fabric CA in Kubernetes, let's register and enroll a peer node, org-admin, and user. Then we will practice modifying the user credentials, and listing and storing the user certs!
For this part of the tutorial, it is recommended that you refer to the [Hyperledger Fabric CA documentation](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/users-guide.html) to see and become familiar with the various commands and attributes of managing cryptographic identities.  

Still in the fabric-ca container, run the following command to register an org-admin with specific attributes. 
Note: Type and attributes are important to understand, so I encourage you to thoroughly review the [Hyperledger Fabric CA docs](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/users-guide.html)!
```
fabric-ca-client register -d --id.name admin-org1 --id.secret org1AdminPW --id.type admin --id.attrs "hf.Registrar.Roles=*,hf.Registrar.Attributes=*,hf.Revoker=true,hf.GenCRL=true,admin=true:ecert,abac.init=true:ecert" -u http://0.0.0.0:7054
```
Note the output will contain the listing of the identity password and attributes.

Now let's register a peer node:
```
fabric-ca-client register -d --id.name org1peer1 --id.secret org1peer1PW --id.type peer -u http://0.0.0.0:7054
```
And finally, we'll register a user:
```
fabric-ca-client register -d --id.name user --id.secret userpw --id.type client -u http://0.0.0.0:7054
```

Now that we have registered our idenities and peer node, we need to enroll them.  We will need the server output to store the credentials in what will become our msp folder.  To do this, we point our FABRIC_CA_CLIENT_HOME export variable at it.
Let's start by enrolling the admin identity:
```
export FABRIC_CA_CLIENT_HOME=/shared/artifacts/org1/org1admin
fabric-ca-client enroll -d -u http://admin-org1:org1AdminPW@0.0.0.0:7054
```
Now let's enroll the peer node identity:
```
export FABRIC_CA_CLIENT_HOME=/shared/artifacts/org1/peer1
fabric-ca-client enroll -d -u http://org1peer1:org1peer1PW@0.0.0.0:7054
```
And finally, let's enroll the user identity:
```
export FABRIC_CA_CLIENT_HOME=/shared/artifacts/org1/user
fabric-ca-client enroll -d -u http://user:userpw@0.0.0.0:7054
```
Now let's practice with some fabric-ca-client commands.  First, let's list all the identities:
```
fabric-ca-client identity list
```
**Expected output is an error!**  Why? The last variable export we did was for the user identity, so we need to prove that we have the access and authority to run these commands because the user identity is not authorized to.  In short, it is a security measure made possible by cryptographic identities and is the foundation of a secure blockchain network.  We should be worried if we did not get this error.

How do we list the identities? By exporting the variable that proves we have access to the admin's signing certificate.  We do this by exporting the variable that points back to the admin cert location BEFORE we start running the fabric-ca-client commands.
```
export FABRIC_CA_CLIENT_HOME=/shared/artifacts/org1/ca/admin
fabric-ca-client identity list
```
The expected output is a listing of the idenities and their attributes registered with this Certificate Authority.  
If you inspect closely the output of the identity list command we just ran, you'll notice that none of our idenities are affilliated with an org.  Let's change that by modifying the identities to add an affiliation to org1.
```
fabric-ca-client identity modify admin-org1 --affiliation org1
```
The expected output is that we 'Successfully modified identity'.  Now we can run the identity list command again and inspect the output to make sure that org1admin identity is indeed affiliated to org1.
```
fabric-ca-client identity list --id admin-org1
```
Inspect the output of the command and you will see that admin-org1 is now affiliated with org1!
Now let's repeat the steps for the peer node and the user by running each command in the terminal one at a time.
```
fabric-ca-client identity modify org1peer1 --affiliation org1
fabric-ca-client identity list --id org1peer1
```
The node org1peer1 is now affiliated with org1.  

Finally, let's affiliate the user identity with org1.department1
```
fabric-ca-client identity modify user --affiliation org1.department1
fabric-ca-client identity list --id user
```
The user identity is now affiliated with org1.department1.  

## **Congratulations!** You have successfully set up your own Hyperledger Fabric Certificate Authority on Kubernetes, modified the Fabric CA Server configuration file, registered and enrolled identities, and modified identities and inspected the result.  You are now ready to explore running your own Fabric Certificate Authority in production systems without using Cryptogen! 

Feel free to continue referencing the [Hyperledger Fabric CA documentation](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/users-guide.html) and practicing the fabric-ca-client commands against this running instance.  

## **Cleanup**
If you are ready to cleanup, run the following commands:
Type 'exit' to exit the running pod session, then:
```
minikube delete
```