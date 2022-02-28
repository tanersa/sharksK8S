# KUBERNETES (K8S) & AWS EKS

<br />

**K8S**

&nbsp; &nbsp; Whenever it comes to docker containers, they should be orchestrated. Currently, **K8S** is the best **container orchestration tool** in the market. 
K8S is designed by Google and now managed by **Cloud Native Computing**(CNC). K8S was **released** first time in 2014. 


   ![alt text](https://github.com/tanersa/sharksK8S/blob/feature/k8s-eks/K8s-Cluster.png)
   
&nbsp; &nbsp; As seen from the image, **Master Node** would provision **multiple Worker Nodes** for configuration need. Therefore, **Master Node** orchestrate everything. 

<br />

**Let's create our first K8S Cluster:**

**Pre-requisite** for K8S Cluster:
   -  AWS CLI
   -  KUBECTL AGENT (In order to talk to K8S Cluster)

Let's continue to build our cluster..

   -  Create an instance on AWS (Ubuntu instance)
   -  Check if **"aws cli"** is installed on your macchine if not do it so.
   -  Install **kubectl** on your Ubuntu machine.
   -  Install **Kops** (Kubernetes Operations) tool for installing, operating, and deleting Kubernetes clusters in the cloud.
 
Next, in order to create K8S cluster we need to create **IAM Role** or a **user** on AWS Console.

This time we create an IAM Role for our cluster.

   -  Create a role on AWS.
   -  Choose EC2 type
   -  Add Permissions for:
         EC2FullAcesss
         AmazonRoute53FullAccess
         AmazonS3FullAccess
         AmazonVPCFullAccess
         
   -  Role Name: K8S
   -  Then go to EC2 instances and choose K8S instance you just created.
   -  EC2 > K8S > Actions > Security > Modify IAM Role
   -  Choose K8S role and SAVE
   -  We just added K8S role to our EC2 instance so we can do actions that we have as permissions.
   -  We also need to store all K8S Configuration in S3 bucket. That is the reason we created IAM Role with S3 bucket permissions.
   -  You may create your S3 bucket using CLI or AWS Console.
   -  During the creation of your S3 bucket, you may enable the "Versioning" for your S3 bucket under Properies to prevent accidental deletion.
   -  We need to store K8S configuration files in S3 bucket.
   -  Encryption would also be enabled for your bucket to increase security.
   
Ideally, we would like to deploy **worker nodes** to different Availability Zones **(AZs)** in order to achieve High Availability **(HA)**.

Lets check our environments created:

   -  export NAME
   -  export KOPS_STATE_STORE

   -  Then, we can create our first cluster with CLI.
   
            kops create cluster --zones:us-east1a,us-east1b ${NAME} 
            
            In this case, NAME will be pulled from our env. variable "NAME"
  
**Output:** We got an ERROR!

   -  Let's check if kops is installed.

            kops
              installed
              
   -  Run the command again to create a cluster
 
            kops update cluster --name sharksCluster-k8s-local --yes --admin
            
Go to EC2 machine on AWS, then you would see **One Master Node** Instance and **Two Worker Nodes** instances.

Cluster still is **not created...**

We may verify cluster with below command:

            kops validate cluster
            
Still  we get **ERROR** message. 

**Lets change our solution using different approach...**

<br />

**AWS Elastic Kubernetes Service (EKS)**

This time we will leverage EKS for creating a cluster.

To start, we will follow the below steps:

   -  In order to leverage EKS cluster, we need to install eksctl utility.
   -  Go to root directory on your instance and check if a user created to leverage.

            aws sts get-caller-identity
                  (Unable to locate credentials)
                  
   -  To create credentials for the user

            aws configure
                  (gather your Access Key and Secret Key from .csv file for your user and paste to 
                  necessary fields )
                  
  Again in order to create resources on behalf of **eksctl**, we  need to create **user** or **user role**
  
  <br />

**All AWS services are integrated to each other with "Permissions"**
  
   -  User must have permissions to talk to different AWS services.
   -  Let's create role for EKS on AWS Console. 
   -  Choose EKS use case
   -  Skip Permissions
   -  Install eksctl on your instance
   -  Install kubectl (to talk to Kubernetes cluster)
   -  Make kubectl executable:

            chmod +x ./kubectl
            
Let's create our **cluster-config.yaml** file 

            apiVersion: eksctl.io/v1alpha5
            kind: ClusterConfig

            metadata:
              name: Sharks-EKS-cluster
              region: us-east-2

            nodeGroups:
              - name: Sharks-ng
                instanceType: t2.small
                desiredCapacity: 2
                ssh:
                  publicKeyName: eks
                        
   -  Create the cluster using CLI
   
            eksctl create cluster -f cluster-config.yaml
            
This config file will pull CloudFormation Template **(CFT)** in background and will **create EKS Cluster.**

<br />

For **AWS EKS,**

Master Node   ----->   **Control Plane**

Worker Node   ----->   **Node Group**

   -  To see the nodes created for EKS

             kubectl get nodes
                 (2 nodes are created)
                 (One for Control Plane and the other for Node Group)
                 
   -  Nodes get all configuration from Control Plane.
   -  To see all node groups:

              eksctl get nodegroup --cluster=Sharks-EKS-cluster
             
   -  We do not see Control Plane instance on AWS Console because it's managed by AWS. We only see Worker Node instance.
  
Now, we can **deploy Docker image using K8S**

   -  Create a new manifest yaml file called **image-deploy.yaml** to deploy an image.
   
               apiVersion: apps/v1
               kind: Deployment 
               metadata:
                 name: tomcat-deployment
               spec:
                 selector:
                   matchLabels:
                     app: tomcat-deployment
                 replicas: 2

                 template:
                   metadata:
                     labels:
                       app: tomcat-deployment
                   spec:
                     containers:
                     - name: tomcat-deployment-container
                       image: sharksdocker/simple-devops-image
                       imagePullPolicy: Always 
                       ports:
                       - containerPort: 8080               
                 
   -  We added this manifest yaml file for eksctl user
   -  Deploy your yaml file for image deployment

               kubectl apply -f image-deploy.yaml
                     (Deployment is created)
                     
   -  To see all deployments, you may run one of below CLIs:

               -kubectl get deploy
               -kubectl get deployment
               -kubectl get deployments
               
   -  To watch deployments live
 
               -kubectl get deployments -w
               
**Note:** 
**POD** is the smallest unit in Kubernetes. Pods are placed in **Nodes** and containers are placed in **PODs.**

   - In order to see PODs in our EKS cluster...

               kubectl get pods   OR    kubectl get po
               
   -  If you do not specify "Replica" in your "Deployment" file, there will only be one POD created. Therefore, there would
      be only one container and one pod created.
      
   -  All PODs would have deployment name plus random string so they can be differentiated from each other.  

               Example: tomcat-deployment-f6d272f83363bn8
               
   -  Lets increase the replica to 4 then we would see 4 PODs.

   -  Deploy yaml file again

                 kubectl apply -f image-deploy.yaml
                 kubectl get po -w
                    (There will be 4 PODs)

<br />

&nbsp; &nbsp; &nbsp; &nbsp; **Difference between PODs and Deployments**

**PODs** would get re-created as long as Deployment exists.

Once, **Deployment** is deleted, you would delete all PODs.

<br />

**This is all about High Availability (HA)**

Let's proof this **self-healing** feature of K8S:

   -  Let's delete the pod and see what happens

               kubectl delete pod <POD Name>
                      (POD is deleted)
                      
               kubectl get po
                       (There are still 4 PODs, so it created one more POD after deletion)
                       
   -  Now, let's delete the deployment now

               kubectl delete deploy <Deployment Name> 
                      (Nothing exists)
               
               kubectl get po
                       (It is terminating all the PODs)
     
   -  For testing purposes, we can delete and create PODs over and over and complete our tests in DEV. However, we do not want to delete PODs 
      in PROD. 
      
   -  To see what is inside the POD:

               kubectl exec -it <POD Name> /bin/bash
               (Now we are inside the POD)
               
It doesn't matter which POD you access because all PODs are **pulling same image**. However, each pod is **not identical**.            
               
   -  Now, lets go to other VM for docker and go to webapps direct.

               cd webapps/
               cat index.jsp
                   <h1> Hello, Welcome to K8S </h1>
                   
   -  Go back to ansible instance 

               cd index.jsp
               cat index.jsp
                   (We can see the file)
                   
   -  Below CLI will give you **IP Adresses** with **PODs** and **instances**.

               kubectl get nodes -o wide
               
We deployed everything in Ohio region but our instances are in Northern Virginia region.

&nbsp; &nbsp; We have **two worker nodes** in Ohio region. We want to make sure whenever we deploy everything, all should get deployed to 
**different PODs** and and **different worker nodes**.

Let's say if one of worker nodes goes down, other worker node will be up and running.

We put...

       Desired Capacity: 2

       Minimum Capacity: 2
       
       Maximum Capacity: 2
       
&nbsp; &nbsp; If one of instances goes down, of course **Auto Scaling Group** will create another instance, but control plane and 
worker node creation will take time. During this time, this worker node will also be in traffic because we want to have zero down time.

&nbsp; &nbsp; Now, we created our **Deployment** and we would like to serrve our traffic to public. That's why we will create another yaml file.

               service-image.yaml
               
               apiVersion: v1
               kind: Service 
               metadata:
                 name: tomcat-service
                 labels:
                   app: tomcat-deployment
                 spec:
                   selector:
                     app: tomcat-deployment
                   type: LoadBalancer
                   ports:
                   - port: 8080
                     targetPort: 8080
                     nodePort: 31200
                     
**Note:** Target port is always container port.

   -  To deploy service:

               kubectl apply -f service-image.yaml

   -  To see services run one of CLIs:

               kubectl get services    
               kubectl get service
               kubectl get svc
               
   -  Go to AWS Console > Load Balancer  

               We should be able to see Load Balancers.
               
   -  Copy DNS Name and paste it to your browser.

               You should be able to see application (In this case TomCat Application)
               
<br />

**Let's say we want this application always up and running.**

   -  To see all deployments and services

               kubectl get all 
                       ( We see everything service and deployment )
                       
**Deployments** create Replicas in backend which are the **PODs** on the screen.    

   -  Let's verify if application will be up and running all the time.

               kubectl get pods
                  (Two pods running)
                  
   -  Delete the pods

               kubectl delete pod <POD1>   
               kubectl delete pod <POD2> 
               
   -  Copy DNS Name and port 8080 on browser and refresh the page    

               You can see application is still up and running
               And pods are recreated again.
               
**That's the Beauty of Kubernetes**!!!

   -  To get very detail output:

               kubectl describe pod <POD Name>     
               kubectl logs <POD Name>  
               
                     You will see the EVENTS in output which is the most important section for debugging.
                     
<br />

**NOTES:**

Containers are inside the PODS and they get networking and everything from PODs.
                     
By default, K8S and containers are Stateless (there is no persistent volume attached).

Containers get those volumes from PODs.

K8S cluster tree for an application goes as follows...

               K8S Cluster > Worker Node > Deployment > POD > Container > Application


                 


               

       
       
       
               
               
                                 
                  
               
               


                       
                      
                      
          
                       
                       
                      

            
                 
          
                 

             

                          
               
               
               
             
             
             
            
            
            
            
  

                            
                  
                  
            
            




      




              
   















