# ROSA integration with Amazon FSx for NetApp ONTAP- Scalable persistent storage solution for stateful container applications.

## About the Setup
The infrastructure setup consists of a Multi-AZ ROSA cluster with  EC2 worker nodes and a FSx ONTAP file system that spans across multiple availability zones. After infrastructure deployment, we will walk through how to leverage NetApp’s Trident Container Storage Interface (CSI) (https://netapp-trident.readthedocs.io/en/stable-v19.01/kubernetes/trident-csi.html)to create storage volume powered by FSxONTAP for a MySql database that runs on ROSA cluster. NetApp's Trident Container Storage Interface (CSI) driver provides a CSI interface that allows ROSA clusters to manage the lifecycle of Amazon FSx for NetApp ONTAP file systems.We will also demonstrate how to scale stateful kubernetes pods across multiple AZ to provide HA to your application. The test environment is created using AWS CLI, ROSA CLI and OC CLI tools along with AWS CloudFormation scripts. We will dive deep into how to deploy Trident CSI Operator into the ROSA cluster via Helm (https://helm.sh/), create the storage class, persistent volume claims so as to let the stateful application pod mount on the volume provided by FSxONTAP.

## Architecture Diagram

![Diagram](/Architecture.png)

## Project Structure

```
.
├── fsx                                     # Holds CloudFormation templates for creating the network environment and FSxONTAP file system
│   ├── FSxONTAP.yaml                       # CloudFormation template for creating FSxONTAP File System
│   └── backend-ontap-nas.yaml              # YAML file for configuring backend settings of Trident CSI
│   └── storage-class-csi-nas.yaml          # YAML file for creating storage class 
│   └── svm_secret.yaml                     # YAML file for creating a k8s secret that stores credentials of FSxONTAP
│   └── pvc-trident.yaml                    # A sample YAML file for creating PVC for trident CSI
│  
│── mysql                               # Holds MySQL artifacts for K8S
|   ├── mysql-configmap.yaml
|   └── mysql-service.yaml
|   └── mysql-statefulset.yaml
|   └── mysql-secrets.yaml
|   └── mysql-client.yaml
└── ...
```

## Prerequisites

You will need the following resources:

* AWS account (https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup)
* A Red Hat account (https://console.redhat.com/)
* IAM user with appropriate permission to create and access ROSA cluster
* Install AWS CLI based on your OS (https://aws.amazon.com/cli/)
* Install ROSA CLI based on your OS (https://console.redhat.com/openshift/downloads)
* Install OpenShift command-line interface (oc) (https://console.redhat.com/openshift/downloads)
* Helm 3 - https://docs.aws.amazon.com/eks/latest/userguide/helm.html
* A ROSA cluster created (Refer here (https://www.rosaworkshop.io/rosa/1-account_setup/) if you want to create a new ROSA cluster)
* Access to Red Hat OpenShift web console
* Note down the router ID of all the subnets of ROSA cluster from AWS console.



# Getting started

## Clone Git repo 
As you clone the referenced GitHub repo, change directory as below:
```
cd rosa-fsx-netapp-ontap/fsx
```



## Create an Amazon FSx for NetApp ONTAP file system
Launch the Cloudformation stack as below to spin up an Amazon FSx for NetApp ONTAP file system. And you need to fill in the parameters and configuration accordingly. Make sure you choose those two private subnets of VPC as that of ROSA cluster.

```
aws cloudformation create-stack \
  --stack-name FSXONTAP \
  --template-body file://./FSxONTAP.yaml \
  --region <region-name> \
  --parameters \
  ParameterKey=Subnet1ID,ParameterValue=[your_preferred_subnet1] \
  ParameterKey=Subnet2ID,ParameterValue=[your_preferred_subnet2] \
  ParameterKey=myVpc,ParameterValue=[your_VPC] \
  ParameterKey=FSxONTAPRouteTable,ParameterValue=[your_routetable] \
  ParameterKey=FileSystemName,ParameterValue=myFSxONTAP \
  ParameterKey=ThroughputCapacity,ParameterValue=512 \
  ParameterKey=FSxAllowedCIDR,ParameterValue=[your_allowed_CIDR] \
  ParameterKey=FsxAdminPassword,ParameterValue=[Define password] \
  ParameterKey=SvmAdminPassword,ParameterValue=[Define password] \
  --capabilities CAPABILITY_NAMED_IAM    
```

## Create an ROSA cluster
Verify the ROSA pre-requisities are met. Open terminal on your computer and execute below commands 

```
rosa create cluster --cluster-name my-rosa-cluster --sts --mode auto --yes --multi-az --region us-west-2 --version 4.9.23 --compute-nodes 3 --compute-machine-type m5.2xlarge --machine-cidr 10.0.0.0/16 --service-cidr 172.30.0.0/16 --pod-cidr 10.128.0.0/14 --host-prefix 23
```

## Clean up
```
rosa delete cluster --cluster <cluster-name>
```

```
aws cloudformation delete-stack --stack-name FSxONTAP --region <region-name>
```



# Security 
See [CONTRIBUTING](https://github.com/aws-samples/mltiaz-fsxontap-eks/blob/main/CONTRIBUTING.md) for more information.

# License
This library is licensed under the MIT-0 License. See the LICENSE file.
