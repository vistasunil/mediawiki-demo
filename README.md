# AWS EKS
## Configure AWS CLI
Use below command to configure AWS CLI using AWS Access and Secret key :

```bash
aws configure
```
## Create IAM role
Create any IAM role for EKS operations, say eksrole with below AWS policies:

 AmazonEKSClusterPolicy
 AmazonEKSServicePolicy

## Install kubectl, helm and aws-iam-authenticator

Kubernetes 1.17:

Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
kubectl version --short --client
```
Install helm

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

Install aws-iam-authenticator

```bash
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
aws-iam-authenticator help
```

## Create Networking
Setup VPC, Security Group and Subnets in AWS region manually or using CloudFormation template from below location:

https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-08-12/amazon-eks-vpc-sample.yaml

## Create EKS cluster either from AWS console or using ekscli

```bash
aws eks create-cluster --region us-east-2 --name mediawiki1 --kubernetes-version 1.16 --role-arn arn:aws:iam::670004242753:role/eksrole --resources-vpc-config subnetIds=subnet-07a0c6b47b139f442,subnet-08cb25c664433af01,subnet-0ab6898a0ecd79ef4,securityGroupIds=sg-0b6e4d47856156feb
```

Check the status of cluster using below command:

```bash
aws eks --region us-east-2 describe-cluster --name mediawiki1 --query "cluster.status"
```

## Update Kubeconfig for EKS cluster
Run below command to update kubeconfig at ~/.kube/config

```bash
aws eks update-kubeconfig --name mediawiki1
```
Check if able to connect to EKS cluster

```bash
kubectl get svc
```

## Create Nodegroup and worker nodes
Create auto scalable Nodegroup and worker nodes manually or using CloudFormation template from below location:

https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-08-12/amazon-eks-nodegroup.yaml

## Deploy aws-auth-cm config map
Create config map to connect nodegroup role and user(s) to eks cluster

Download aws-auth-cm.yaml manifest file and deploy it:

```bash
curl -o aws-auth-cm.yaml https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-08-12/aws-auth-cm.yaml
kubectl apply -f .\aws-auth-cm.yaml
kubectl get nodes --watch	
```

# MediaWiki

[MediaWiki](https://www.mediawiki.org) is an extremely powerful, scalable software and a feature-rich wiki implementation that uses PHP to process and display data stored in a database, such as MariaDB.

```bash
$ git clone https://github.com/vistasunil/mediawiki-demo.git
$ tar -xvf mediawiki-0.x.x.tgz
$ helm install my-mediawiki ./mediawiki		# Helm 3
$ helm install --name my-mediawiki ./mediawiki	# Helm 2
```

To update an exisiting _stable_ deployment with a chart hosted in the bitnami repository you can execute

```bash
$ git clone https://github.com/vistasunil/mediawiki-demo.git
$ tar -xvf mediawiki-0.x.x.tgz
$ helm upgrade my-mediawiki ./mediawiki  
```

## Prerequisites

- Kubernetes 1.15+
- Helm 2.11+ or Helm 3.0-beta3+
- PV provisioner support in the underlying infrastructure
- ReadWriteMany volumes for deployment scaling

## Installing the Chart

To install the chart with the release name `my-release`:

```console
$ helm install my-mediawiki ./mediawiki	
```

The command deploys MediaWiki on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
$ helm delete my-mediawiki
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

> **Note**:
>
> For Mediawiki to function correctly, you should specify the `mediawikiHost` parameter to specify the FQDN (recommended) or the public IP address of the Mediawiki service.

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
$ helm install my-release \
  --set mediawikiUser=admin,mediawikiPassword=password,mariadb.mariadbRootPassword=secretpassword \
    mediawiki
```

The above command sets the MediaWiki administrator account username and password to `admin` and `password` respectively. Additionally, it sets the MariaDB `root` user password to `secretpassword`.

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example,

```console
$ helm install my-release -f values.yaml mediawiki
```

> **Tip**: You can use the default [values.yaml](values.yaml)

