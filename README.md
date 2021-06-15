# AWS-Kops-ClusterSetup

This repostory contains all the detailed instruction to create a production grade kubernetes cluster using Kops in AWS. To create a k8s cluster, please follow the following steps -

### 1. Launch Linux EC2 instance in AWS (It will act as Kubernetes Client)
### 2. Create and attach IAM role to EC2 Instance.
	Please attach these policies to IAM role created as these are required by Kops
		 AmazonS3FullAccess
		 IAMFullAccess
		 AmazonRoute53FullAccess
		 AmazonEC2FullAccess
		 AmazonVPCFullAccess
 		 
### 3. Install Kops on EC2
```sh
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

### 4. Install kubectl
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
### 5. Create S3 bucket in AWS
S3 bucket is used by kubernetes to persist cluster state, lets create s3 bucket using aws cli
**Note:**  Make sure you choose bucket name that is uniqe accross all aws accounts

```sh
aws s3 mb s3://abhi.in.k8s --region us-east-2
```
### 6. Create private/pubic hosted zone in AWS Route53
#### For private hosted zone
 1. Head over to aws Route53 and create hostedzone
 2. Choose name for example (abhi.in.k8s)
 3. Choose type as privated hosted zone for VPC
 4. Select default vpc in the region you are setting up your cluster
 5. Hit create

#### For public hosted zone 
 1. Head over to aws Route53 and create hostedzone
 2. Choose your domain name for example (abhi.in)
 3. Choose type as public hosted zone for VPC
 4. Give description
 5. Hit create

### 7 Configure environment variables.
Open .bashrc file 
```
	vi ~/.bashrc
```
Add following content into .bashrc file, and make sure bucket name matches the one you created in step 5.

```sh
export KOPS_STATE_STORE=s3://abhi.in.k8s
```
Then run the command to reflect variables added to .bashrc file
```
	source ~/.bashrc
```
### 8. Create ssh key pair
This keypair will be used for ssh into kubernetes cluster

```sh
ssh-keygen
```

### 9. Create a Kubernetes cluster from the config file. 
Make sure to change config file with your cluster config file (if any) and make neccessary changes in config file as per your requirements.

```sh
kops create cluster -f cluster-setup.yaml
```

### 10. Create secret to start with installation process

```sh
kops create secret --name test.devtron.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub
```

### 11. Create kubernetes cluster, please don't forget to replace cluster name with your cluster name given.

```sh
kops update cluster test.domain.name --yes 
```
Above command may take some time to create the required infrastructure resources on AWS. Execute the validate command to check its status and wait until the cluster becomes ready

```sh
kops validate cluster
```
For the above above command, you might see validation failed error initially when you create cluster and it is expected behaviour, you have to wait for some more time and check again.

### 12 To check all clusters created
```sh
kops get clusters
```

### 13. To connect to the master, replace with your cluster name after api.
```sh
ssh admin@api.test.doamin.name
```
After getting into the master, you can run all your kubectl commands, create deployments, get pods, etc and istall tools having kubernetes cluster as dependencies. To come out of master, type ```exit	```

# Destroy the kubernetes cluster
```sh
kops delete cluster test.domain.name --yes
```


