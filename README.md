The repo is intended for following the steps to run SaaS on top of AWS IaaS, including k8s+helm for development.

### Below are the IaC tools we will use:

#### Jenkins for CI/CD 
SAAS infrastructure orchestration with Jenkins and CI/CD pipelines

#### AWS 
AWS IaaS platform where we would build the images and provisioning SAAS infrastructure using AWS API. 

#### Packer (and ansible) 
Packer for creating custom AMI images on AWS cloud

#### Terraform 
for Infrastructure/Cloud Orchestration: create, change, and improve SAAS infrastructure. Terraform will be used because we will have to support multiple clouds long term: Azure, Oracle, Google, Hybrid, etc. TF can be used to provision and manage any cloud, infrastructure, or service)

<img src="https://github.com/adavarski/SaaS-AWS-POC/blob/main/diagrams/IaC_SaaS_DEV.png?raw=true" width="650">



###  SaaS

AWS infrastructure EC2 (VMs) for dev  

HashiCorp Valut VM + Monitoring/Debugging and Log management VMs: ElasticSearch cluster (3-nodes/3 VMs), Logstash/Fluentd, Kibana, Grafana, InfuxDB, Sensu, Zabbix + Kafka VMs: Kafka 3-nodes cluster (3-nodes/3 VMs), Kafka Replicator, Kafka Connect, Schema Registry + Consul cluster VMs (3-nodes/3 VMs) + Zookeper cluster (3-nodes/3 VMs) + Postgres cluster (3-nodes/3 VMs) + etc. . 

We have to deploy the whole AWS SaaS infrastructure : VPCs, IAM, EC2, AMIs, EBS, ELB, S3, Route53, RDS, ECS, CloudWatch, Security Groups & ACLs, CloudTrail, Networks/VPNs ( SaaS VPC <---> Customer VPC: VPC peering/Site-to-Site VPN/etc.) , etc. with Jenkins/TF/Ansible/etc. (IaC based —> we have to have all source @github).


##### awscli checks: AWS Management Console URL: https://console 

```
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_DEFAULT_REGION=

aws iam get-group --group-name Administrators | jq -r '.Users[].UserName'
aws ec2 describe-key-pairs | jq -r '.KeyPairs[].KeyName'
aws ec2 describe-security-groups | jq '.SecurityGroups[] | select (.GroupName == "default") | .GroupId'
aws ec2 describe-security-groups | jq -r '.SecurityGroups[]'
aws ec2 describe-subnets | jq '.Subnets[]'
aws ec2 describe-vpcs | jq '.Vpcs[] | { VpcId:.VpcId,CidrBlock:.CidrBlock }'
aws ec2 describe-images --owners aws-marketplace --filters Name=product-code,Values=aw0evgkw8e5c1q413zgy5pjce --query 'Images[*].[CreationDate,Name,ImageId]' --filters "Name=name,Values=CentOS Linux 7*" --region us-west-2 --output table | sort -r
aws ec2 describe-images --owners 941105238039 --query 'Images[*].[CreationDate,Name,ImageId]' --filters "Name=name,Values=redshift*2020*" --region us-west-2 --output table | sort -r
aws ec2 describe-instances   --filter 'Name=instance-state-name,Values=running' |   jq -r '.Reservations[].Instances[] | [.InstanceId, .PrivateIpAddress, .Tags[].Value] | @json' 
... 
```
##### Note1: 

- ansible: playbooks & roles for SaaS services 
- packer: packer templates for SaaS services (AWS AMIs) to build Zoo/Consul/Postgres/etc. clusters (AWS AMIs based on ansible roles and custom scripts)
- builder: HOWTOs docs for different saaS services build (jenkins+ansible+packer+terraform) 
- jenkins: J.pipelines to deploy/upgrade/update/destroy infrastructure services 
- terraform: DEV infrastructure provisioning  (examples for VPC & EC2 & etc. DEV NOC/MC infrastructure provisioning)
- diagrams: diagrams & pictures

##### Note2:

AWS VPCs for SaaS dev/staging/prod:
VPC for AWS SAAS Dev env: 10.0.0.1/16
VPC for AWS SAAS Staging env : 10.1.0.1/16
VPC for AWS SAAS Production env: 10.2.0.1/16


##### Note3: 
TF create infrastructure FULL DEV (VPC, EC2, ELB, etc.): We have to deploy the whole AWS infrastructure (VPCs, IAM, EC2, AMI, EBS, ELB, S3, Route53, RDS, ECS, CloudWatch, Security Groups & ACLs, CloudTrail, Networks/VPNs, etc.) with TF/Ansible/packer/AMIs.

For SaaS DEV env: 

	1.	VPC
	2.	Public Subnet - 1
	3.	Private Subnet - 2 & 3
	4.	Internet Gateway
	5.	Route Tables
	6.	ELB
	7.	Security Groups
	8.	EC2 Instances
	9.	Install HAProxy LB for internal & external access
	10.	Attach Instances to AWS ELBs 
	11.	Setup CloudWatch SNS email alerts (?), etc.
  

#####  Note4. 
terraform:main.tf for the full SaaS DEV infra creation VPC+EC2+etc. or. we can create only VPC, networks, sec groups with TF terraform:main-vpc.tf.example and after that hardcode VPC ID, networks IDs, sec groups, etc. for terraform:main-ec2-route53-elb.tf.example for all SaaS VMs services creation (use CM:ansible after VMs provisioning with TF for all services where service setup with TF+AMIs(packer+ansible+custom scripts) is not possible (for some of services TF+AMIs, for some use ansible for setup/configuration).


