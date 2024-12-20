---
title: "AWS CLI queries"
date: 2022-02-05T10:31:29-05:00
---
```
Knowledge Base Article - CTAC (Communications Training Analysis Corporation)  
=================================================


Subject: AWS CLI Queries

Revision History:
    2015-07-13 13:53:00 - Version 1 created by Bao Nguyen
    2017-07-06 14:21:04 - Version more than 1 modified by Andrew O'Dell


Details: (include steps to reproduce, symptoms, corrective actions, contact information, links and a any other helpful information)


==============================
IAM Queries
==============================
aws iam list-users --query 'Users[*].[UserName]' --output text --profile hhs

#Reset your own password (substitute as necessary
aws-vault exec nocadmin -- aws iam update-login-profile --user-name aodell@ctacorp.com --password "makeupsomethingreallygood"

aws-vault exec nocadmin -- aws iam update-login-profile --user-name bnguyen@ctacorp.com --password "makeupsomethingreallygood"

#If you know your password
aws-vault exec aaccount -- aws iam change-password --cli-auto-prompt

==============================
AWS EC2
==============================

- AWS ec2 ands
aws ec2 describe-instances --query 'Reservations[*].[Instances[0].InstanceId, Instances[0].InstanceType]' --output table

aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,SubnetId,PrivateIpAddress,Tags[?Key==`Name`]| [0].Value]' --filters "Name=vpc-id,Values=vpc-e6d2f483" --output text

aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`]| [0].Value]' --filters "Name=vpc-id,Values=vpc-e6d2f483" --output text

aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,Tags[?Key==`Name`]| [0].Value]’ --filter “Name=tag:Name,Values=usa*” --output table

aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`]| [0].Value]’ --filter “Name=tag:Name,Values=*drupal*” --output text --output text

aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress,SecurityGroups[0].GroupId,Tags[?Key==`Name`] | [0].Value]' --filter "Name=tag:Name,Values=usa*" --output table

aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,Tags[?Key==`Name`]| [0].Value]' --filters "Name=vpc-id,Values=vpc-e6d2f483" --output text

- get instance id, hostname, private ip, public ip for only prod tier, for the VPC whose ID is vpc-e6d2f483
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,PrivateIpAddress,PublicIpAddress, Tags[?Key==`Name`]| [0].Value]' --filter "Name=vpc-id,Values=vpc-e6d2f483" --filter "Name=tag:Environment,Values=prod" --filter Name=instance-state-name,Values=running --output table

- I can’t remember why this query exists
aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress,Tags[?Key==`Name`] | [0].Value]' --profile GSA

- Return servers that are shut down (change to 16 for running)
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,Tags[?Key==`Name`]| [0].Value]' --filter "Name=tag:Type,Values=Production" --filter Name=instance-state-code,Values=80  --output=text

- same as above but with VPC filter
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,Tags[?Key==`Name`]| [0].Value]' --filters "Name=tag:Type,Values=Production", Name=instance-state-code,Values=16, Name=vpc-id,Values="vpc-1f8d4e7a" --output=text

aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,Tags[?Key==`Name`]| [0].Value]' --filters "Name=tag:Type,Values=stage”, Name=instance-state-code,Values=16, Name=vpc-id,Values="vpc-1f8d4e7a" --output=text

- list all percussion production instances
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,Tags[?Key==`Profile`]| [0].Value]' --filter "Name=tag:Name,Values=*percussion*prod*" --output table

- all instances for a given VPC that are shut down
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,Tags[?Key==`Name`]| [0].Value]' --filters "Name=vpc-id,Values=vpc-1f8d4e7a"  --filter Name=instance-state-code,Values=80 --output=text

- AWS ec2 describe-volumes
aws ec2 describe-volumes --query 'Volumes[*].[Size, VolumeId, Attachments[0].InstanceId]' --output table

aws ec2 describe-volumes --query 'Volumes[*].[Size, VolumeId, Attachments[0].InstanceId, Attachments[0].Device]' --output table

- Sum of all volumes in use
aws ec2 describe-volumes --query 'Volumes[*].[Size]' --output text |awk '{x+=$0}END{print x}'

paste <(for x in `aws ec2 describe-volumes --query 'Volumes[*].[Attachments[0].InstanceId]' --output text`; do aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`]| [0].Value]' --filters "Name=vpc-id,Values=vpc-e6d2f483" --instance-ids "$x" --output text; done) <(aws ec2 describe-volumes --query 'Volumes[*].[Attachments[0].InstanceId,VolumeId,State,Size]' --output text)

- ELB query to show OutofService instances
for x in `aws elb describe-load-balancers --query 'LoadBalancerDescriptions[].LoadBalancerName[]' --output text`; do echo $x; aws elb describe-instance-health --load-balancer-name "$x" --query 'InstanceStates[].[InstanceId,State]'  --output table; done

- ELB Query for security groups for a given ELB
aws elb describe-load-balancers --load-balancer-names elb-syndication-admin-stage --query "LoadBalancerDescriptions[].SecurityGroups" --output text | xargs aws ec2 describe-security-groups --query "SecurityGroups[].Description" --group-ids

- ELB Query for Canonical name and vpcID
aws elb describe-load-balancers --query "LoadBalancerDescriptions[].[CanonicalHostedZoneNameID,CanonicalHostedZoneName,VPCId]" --output table

- Show snapshots for HHS environment (owner-id = 249709554004)
aws ec2   describe-snapshots --query 'Snapshots[*].[OwnerId, Description, VolumeId, VolumeSize]' --owner-ids 249709554004 --output table

- describe-subnet queries
aws ec2 describe-subnets --query 'Subnets[].[CidrBlock]' --filters "Name=vpc-id,Values=vpc-e6d2f483,Name=subnet-id,Values=subnet-66b6c511" --output text
for x in `aws ec2 describe-instances --query 'Reservations[].Instances[].SubnetId' --filters "Name=vpc-id,Values=vpc-e6d2f483" --output text`; do aws ec2 describe-subnets --query 'Subnets[].[CidrBlock]' --filters "Name=subnet-id,Values=$x" --output text; done
for x in `aws ec2 describe-instances --query 'Reservations[].Instances[].SubnetId' --filters "Name=vpc-id,Values=vpc-e6d2f483" --output text`; do for y in `aws ec2 describe-subnets --query 'Subnets[].[CidrBlock]' --filters "Name=subnet-id,Values=$x" --output text`; do echo $x - $y; done ; done

- nslookup on all of your ELBs
for x in `aws elb describe-load-balancers --query "LoadBalancerDescriptions[].DNSName" --output text`; do nslookup $x; done

- Reserved instances
aws ec2   describe-reserved-instances --query 'ReservedInstances[*].[ReservedInstancesId,OfferingType,InstanceType,InstanceCount]'  --output text

- unhealthy host check
aws ec2 describe-instance-status --query 'InstanceStatuses[].[InstanceId,InstanceStatus.Status]' --filter "Name=instance-status.status,Values=impaired" --output text

- crazy ELB query for Mohammed at Aquilent, formatted for table names (use a json to csv converter)
- aws elb describe-load-balancers --query "LoadBalancerDescriptions[].{LBName:LoadBalancerName,DNSName:CanonicalHostedZoneName,SecurityGroups:SecurityGroups,Instances:Instances[*].InstanceId,Healthcheck:HealthCheck,LoadBalancerPortProtocol:ListenerDescriptions[].Listener.[LoadBalancerPort,Protocol]}”

- FDA Related queries for the monthly quotes
aws rds describe-db-instances --query 'DBInstances[*].[Endpoint.Address,DBInstanceClass, BackupRetentionPeriod, Iops, EngineVersion, AllocatedStorage, DBInstanceClass]' --output text | grep fda
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,InstanceType,PrivateIpAddress,Tags[?Key==`Name`]| [0].Value]' --filters "Name=vpc-id,Values=vpc-e6d2f483" --output text | grep -i fda

- query for just the currently used unique Security Group IDs
aws elb describe-load-balancers  --load-balancer-names hhs-sg-akamai-new-block1-http-https --query "LoadBalancerDescriptions[].SecurityGroups" | grep -v -e '\[' -e '\]' | sed 's/"//g' | sed 's/,//g' | sort  | uniq

- query to return unique security groups in use (need to install jq first)
aws elb describe-load-balancers  --query "LoadBalancerDescriptions[]" --output json | jq -r ".[].SecurityGroups[]" | sort -u

- Query to describe all running and/or windows instances:
aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress,Tags[?Key==`Name`]| [0].Value]' --filters "Name=vpc-id,Values=vpc-e6d2f483"  --filter Name=instance-state-name,Values=running  --filter "Name=platform,Values=windows"  --output text

# Get a count of running instances group by instance type
aws-vault exec cpscmain -- aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceType]' --output text --filter Name=instance-state-name,Values=running | sort | uniq -c


- Query to return what certs are in use by what load balancer:
for x in $(aws-vault exec hhsadmin -- aws elbv2 describe-load-balancers --query 'LoadBalancers[].[LoadBalancerArn]' --output text); do echo "$x"; aws-vault exec hhsadmin -- aws elbv2 describe-listeners --load-balancer-arn "$x" --query 'Listeners[].Certificates[].CertificateArn' --output text; done

- Query to list the IAM certificates by expiration date:
aws iam list-server-certificates --output text --query 'ServerCertificateMetadataList[*].[Expiration,ServerCertificateName]' | sort


- Get instance event status for a given instance id.
aws ec2 describe-instance-status --instance-id i-38f22deb --query 'InstanceStatuses[].Events[].Description' --output text


# Set MakeSnapshot tag to false for servers whose Environment tag=false
for x in `aws ec2 describe-volumes --profile hhs --filters Name=tag-key,Values="Environment" Name=tag-value,Values="dev" --query 'Volumes[].Attachments[].[VolumeId]' --output text`; do aws ec2 create-tags --profile hhs --resources $x --tags Key=MakeSnapshot,Value=false; done

# Obtaining private IP of load balancer
aws-vault exec hhsadmin -- aws ec2 describe-network-interfaces --query 'NetworkInterfaces[].[PrivateIpAddress]' --filters 'Name=description,Values=ELB app/alb-hhs-intranet-prod/477fb07bd611c477' --output text | head -1

#Obtain private IP of load balancer for intranet dev
aws-vault exec hhsadmin -- aws ec2 describe-network-interfaces --query 'NetworkInterfaces[].[PrivateIpAddress]' --filters 'Name=description,Values=ELB app/alb-hhs-intranet-dev/66508b9f1b312c45' --output text


#Query to get Auto Scaling health check type
aws-vault exec hhsadmin -- aws autoscaling  describe-auto-scaling-groups --query 'AutoScalingGroups[].[AutoScalingGroupName,HealthCheckType]' --output text


#Query for static hosts
aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=tag:host_type,Values=static" "Name=tag:Environment,Values=stg,dev,qa" --output text

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=tag:host_type,Values=static" "Name=tag:Environment,Values=prod,svcs" --output text


=============
AWS ALBv2
=============

for x in $(aws-vault exec hhsadmin -- aws elbv2 describe-load-balancers --query 'LoadBalancers[].[LoadBalancerArn]' --output text); do echo "$x,"; aws-vault exec hhsadmin -- aws elbv2 describe-listeners --load-balancer-arn "$x" --query 'Listeners[].Certificates[].CertificateArn' --output text; done

=============
AWS RDS
=============

- List RDS instances, their Multi-az setting, and copytagstosnapshot setting
aws rds describe-db-instances --query 'DBInstances[].[DBName,MultiAZ,CopyTagsToSnapshot]' --output table

- List RDS Endpoints
aws rds describe-db-instances --query 'DBInstances[*].Endpoint.[Address]' --output text

- RDS Retention Period
aws rds describe-db-instances --query 'DBInstances[].[Endpoint.Address,DBInstanceClass, BackupRetentionPeriod]' --output table

- Get the size of RDS usage
for x in $(aws-vault exec hhsadmin -- aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier]' --output text); do echo $x; echo "$(aws-vault exec hhsadmin -- aws cloudwatch get-metric-statistics --metric-name FreeStorageSpace --start-time 2020-09-24T00:00:00Z --end-time 2020-09-25T00:00:00Z --period 86400 --namespace AWS/RDS --statistics Maximum --dimensions="Name=DBInstanceIdentifier, Value=$x" --query 'Datapoints[].Maximum' --output text)/1024/1024" | bc; done
  done

- Another cloud watch RDS query
echo "$(aws-vault exec hhsadmin -- aws cloudwatch get-metric-statistics --metric-name FreeStorageSpace --start-time 2020-09-25T00:00:00Z --end-time 2020-09-25T00:00:00Z --period 86400 --namespace AWS/RDS --statistics Maximum --dimensions="Name=DBInstanceIdentifier, Value=hhs-opa-rds-drupal-prod-1" --query 'Datapoints[].Maximum' --output text)/1024/1024" | bc


RDS query for getting engine version
aws-vault exec hhsadmin -- aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,EngineVersion]' --output text

=============
AWS S3
=============

- S3 Bucket sizes (takes a long ass time to run)
for x in `aws s3api list-buckets --query Buckets[].Name --output text`; do echo $x; s3cmd du s3://$x; done

# listing s3 bucket and sorting count by file types
aws-vault exec nocadmin -- aws s3 ls s3://ctac-sysadmin-pub/ --recursive | sed '/\/$/d' | awk '{print $4}' | awk -F . '{print $NF}' | sed 's:.*/::' | sort | uniq -c | sort -nr

==========================
Misc
==========================
Passing Keys within your query (remember to rotate access keys after this or clear bash history)
AWS_ACCESS_KEY_ID=ABCD AWS_SECRET_ACCESS_KEY=EF1234 aws ec2 describe-instances
s3cmd put --access_key=AWS_ACCESS_KEY_ID --secret_key=AWS_SECRET_ACCESS_KEY



- Check for multipart uploads:
for x in $(aws-vault exec hhsadmin -- aws s3api list-buckets --query 'Buckets[].[Name]' --output text); do echo $x; aws-vault exec hhsadmin -- aws s3api list-multipart-uploads --bucket $x --output text | wc -l; done

- clear multi part uploads (untested):
for bucket in $(aws s3api list-buckets --query Buckets[*].Name --output text); do
  echo "$bucket"
  aws s3api list-multipart-uploads --bucket "$bucket" --query Uploads[*].[Key,UploadId,Initiated,Initiator.DisplayName] --output text | while read key id date user; do
    [[ "$key" == "None" ]] && continue
    echo "Press enter to abort s3://$bucket/$key (initiated $date by $user)"
    read < /dev/tty
    aws s3api abort-multipart-upload --bucket "$bucket" --key "$key" --upload-id "$id"
  done
done

==========================
Patch Party queries
==========================

#Non-prod
aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value, InstanceType, PrivateIpAddress, State.Name]' --filter "Name=vpc-id,Values=vpc-e6d2f483" "Name=instance-state-name,Values=running" "Name=tag:Environment,Values=stg,dev,qa" "Name=tag:host_type,Values=static" --output text | sort

#Prod

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value, InstanceType, PrivateIpAddress, State.Name]' --filter "Name=vpc-id,Values=vpc-e6d2f483" "Name=instance-state-name,Values=running" "Name=tag:Environment,Values=svcs,prod" "Name=tag:host_type,Values=static" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`]| [0].Value, InstanceType, PrivateIpAddress,State.Name]' --filter "Name=vpc-id,Values=vpc-e6d2f483" --output text  | sort




- query for production patch party
aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value], PrivateIpAddress, ' --filter "Name=vpc-id,Values=vpc-e6d2f483" --filter Name=instance-state-name,Values=running --output text | grep prod | sort

- similar to the above but with more tags
aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress,PublicIpAddress, Tags[?Key==`Name`]| [0].Value,Tags[?Key==`Tier`]| [0].Value, Tags[?Key==`Profile`]| [0].Value, Tags[?Key==`Customer`]| [0].Value]' --filter "Name=vpc-id,Values=vpc-e6d2f483" --filter "Name=tag:Environment,Values=dev,stg" --output text



==========================
SSP Queries (Can probably combine these into 1).
==========================

First:
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,Tags[?Key==`Name`]| [0].Value,InstanceType,PrivateIpAddress,PublicIpAddress,SubnetId]' --filters "Name=vpc-id,Values=vpc-e6d2f483" > hhs-instances.out

Second:
for x in `aws ec2 describe-instances --query 'Reservations[].Instances[].[SubnetId]' --filters "Name=vpc-id,Values=vpc-e6d2f483" | head -5`
do 
   aws ec2 describe-subnets --query 'Subnets[].[Tags[?Key==Name]| [0].Value,CidrBlock]' --filters "Name=subnet-id,Values='$x'"
 done

==========================
Some queries for the Baseline document:
==========================
	
aws --profile hhs ec2 describe-addresses --output text --query 'Addresses[].[PrivateIpAddress]' --output text >> private-ips.out
for x in $(cat private-ips.out); do aws --profile hhs ec2 describe-instances --filters "Name=network-interface.addresses.private-ip-address,Values=$x" --query 'Reservations[].Instances[].[Tags[?Key==`Name`]| [0].Value]' --output text; done

aws --profile hhs ec2 describe-key-pairs --output text --query 'KeyPairs[].[KeyFingerprint,KeyName]'
tr
Appendix 4:
aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,SecurityGroups[].GroupId]' --filter Name=instance-state-name,Values=running --output text
aws-vault exec hhsadmin -- aws ec2 dwescribe-instances --query 'Reservations[].Instances[].SecurityGroups[].[GroupId]' --filter Name=instance-state-name,Values=running --output text

==========================
Maintenance worksheet queries:
==========================

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceType, Tags[?Key==`Name`] | [0].Value, PrivateIpAddress, State.Name]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=stg,dev,qa" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceType, Tags[?Key==`Name`] | [0].Value, PrivateIpAddress, State.Name]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=prod,svcs" --output text | sort

#For raj's spreadsheet:
aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value, InstanceType,PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=prod,svcs" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value, InstanceType,PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" --filter Name=instance-state-name,Values=running "Name=tag:Environment,Values=dev,stg,qa" --output text | sort


========================
Nessus
========================

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=qa" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=stg" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=dev" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=svcs" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=prod" --output text | sort


aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=qa, dev, stg" --output text | sort

aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PrivateIpAddress]' --filter "Name=vpc-id,Values=vpc-e6d2f483" Name=instance-state-name,Values=running "Name=tag:Environment,Values=prod,svcs" --output text | sort


#Quarterly Reporting

#Get an inventory of all EC2 instance types and their counts
aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceType]' --output text | sort | uniq -c


#Table 7
aws-vault exec hhsadmin -- aws ec2 describe-instances --query 'Reservations[].Instances[].[PublicIpAddress, Tags[?Key==`Name`]| [0].Value]' --output text | grep -v -e None


#Table 9
aws-vault exec hhsadmin -- aws elbv2 describe-load-balancers --query 'LoadBalancers[].[LoadBalancerName]' --output text

#Table 10
aws-vault exec hhsadmin aws s3 ls | awk '{print $3}'

#Table 11:
aws-vault exec hhsadmin -- aws efs describe-file-systems --query 'FileSystems[].[Name]' --output text | sort

# Table 12:
aws-vault exec hhsadmin -- aws rds describe-db-instances --query 'DBInstances[].[DBInstanceClass]' --output text | uniq -c

=================================================
This document is proprietary and confidential. No part of this document may be 
disclosed in any manner to a third party without the prior written consent of 
CTAC.```
