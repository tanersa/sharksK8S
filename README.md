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


                          
               
               
               
             
             
             
            
            
            
            
  

                            
                  
                  
            
            




      




              
   















