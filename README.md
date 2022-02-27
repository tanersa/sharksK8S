# KUBERNETES (K8S) & AWS EKS

<br />

**K8S**

&nbsp; &nbsp; Whenever it comes to docker containers, they should be orchestrated. Currently, **K8S** is the best **container orchestration tool** in the market. 
K8S is designed by Google and now managed by **Cloud Native Computing**(CNC). K8S was **released** first time in 2014. 


   ![alt text](https://github.com/tanersa/sharksK8S/blob/feature/k8s-eks/K8s-Cluster.png)
   
&nbsp; &nbsp; As seen from the image, **Master Node** would provision **multiple Worker Nodes** for configuration need. Therefore, **Master Node** orchestrate everything. 

<br />





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
   -  You may create your S3 bucket using cli or AWS Console.
   -  During the creation of your S3 bucket, you may enable the "Versioning" for your S3 bucket under Properies to prevent accidental deletion.
   -  We need to store K8S configuration files in S3 bucket.
   -  Encryption would also be enabled for your bucket to increase security.
   
Ideally, we would like to deploy **worker nodes** to different Availability Zones **(AZs)** in order to achieve High Availability **(HA)**.

        















