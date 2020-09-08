# aws-cli-to-check-ec2-intance-type-and-name-and-public-and-private-address
aws cli to check ec2 intance type and name and public and private address

```
aws ec2 describe-instances  --filter "Name=instance-state-name,Values=running" --query 'Reservations[*].Instances[*].[InstanceId, InstanceType, PrivateIpAddress, PublicIpAddress,  InstanceLifecycle, [Tags[?Key==`Name`].Value] [0][0],[Tags[?Key==`billing`].Value] [0][0] ]  ' --output text

aws ec2 describe-instances --output text --query 'Reservations[*].Instances[*].[InstanceId, InstanceType, ImageId, State.Name, LaunchTime, Placement.AvailabilityZone, PrivateIpAddress, PublicIpAddress, SubnetId, InstanceLifecycle, [Tags[?Key==`Name`].Value] [0][0],[Tags[?Key==`Reserve`].Value] [0][0], [Tags[?Key==`Users`].Value] [0][0] , [Tags[?Key==`Owner-Tech`].Value] [0][0] , [Tags[?Key==`Cost-Center`].Value] [0][0] ,  [Tags[?Key==`BU-Head`].Value] [0][0] , [Tags[?Key==`Details`].Value] [0][0] , [Tags[?Key==`Created by`].Value] [0][0] , [Tags[?Key==`Approved by`].Value] [0][0] , [Tags[?Key==`Requested by`].Value] [0][0] , [Tags[?Key==`Services`].Value] [0][0] , [Tags[?Key==`Console-Access`].Value] [0][0] , [Tags[?Key==`Application`].Value] [0][0] , [Tags[?Key==`Env`].Value] [0][0] , [Tags[?Key==`Purpose`].Value] [0][0] , [Tags[?Key==`Domain`].Value] [0][0] , [Tags[?Key==`Tech-Team`].Value] [0][0] , [Tags[?Key==`SSL`].Value] [0][0] , [Tags[?Key==`Linked-Servers`].Value] [0][0] , [Tags[?Key==`Backup-interval-daily`].Value] [0][0] , [Tags[?Key==`Backup-interval-weekly`].Value] [0][0] , [Tags[?Key==`Backup-interval-monthly`].Value] [0][0] , [Tags[?Key==`Backup-interval-yearly`].Value] [0][0] , [Tags[?Key==`db`].Value] [0][0] ]  ' > raw.tsv
```
# to check the access key id of all the aws account with user name

```
for user in $(aws iam list-users --output text | awk '{print $NF}'); do
    aws iam  list-access-keys --user $user --output text 
    #aws iam get-access-key-last-used --access-key-id $user |awk '{print $2}' >>date
done

#for i in $(cat date)
#do
#	echo $i; --output text
#	aws iam get-access-key-last-used --access-key-id $i | egrep -i "UserName|LastUsedDate"
#	
#done

```
# to check volume size attach to ec2 instance
```
aws ec2 describe-volumes --region ap-south-1 --query "Volumes[*].[Attachments[0].VolumeId,Attachments[0].InstanceId,Attachments[0].State,Size]" --output text | awk '{print $2,$4}' >sizeid.txt

--id.txt will contain all the instance ID from the above query of ec2 details---------------------
for i in $(cat id.txt)
do
	cat sizeid.txt | grep $i
	echo " "
done	
```

# to check the s3 bucket size of all aws account

```
#!/bin/bash
aws_profile=('default');

#loop AWS profiles
for i in "${aws_profile[@]}"; do
  echo "${i}"
  buckets=($(aws --profile "${i}" --region ap-south-1 s3 ls s3:// --recursive | awk '{print $3}'))

  #loop S3 buckets
  for j in "${buckets[@]}"; do
  echo "${j}"
  aws --profile "${i}" --region ap-south-1 s3 ls s3://"${j}" --recursive --human-readable --summarize | awk END'{print}'
  done

done

```

# Average CPU and Max CPU of ec2 instance 
```
for i in $( cat instanceid.txt )
do
       	aws cloudwatch get-metric-statistics --region ap-south-1 --metric-name CPUUtilization --start-time 2020-03-11T14:21:00 --end-time 2020-03-25T14:21:00 --period 3600 --namespace AWS/EC2 --statistics Maximum --dimensions Name=InstanceId,Value=$i --output text | awk '{print $2}' | sort -nrk1,1  | head -1 
done


# replace Maximum with Average 
# instanceid.txt is the file which contain all the instanceId of ec2 instance, use below command to find instance ID
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[State.Name, InstanceId]' --output text 

#to find cpu of single instance. replcae Value with your instance ID and replace average with Maximum
aws cloudwatch get-metric-statistics --region ap-south-1 --metric-name CPUUtilization --start-time 2020-03-11T14:21:00 --end-time 2020-03-25T14:21:00 --period 3600 --namespace AWS/EC2 --statistics Average --dimensions Name=InstanceId,Value=i-06d953ad09590f2a2 --output text | awk '{print $2}' | sort -nrk1,1  | head -1

```

# RDS details of all the DB
```
aws rds describe-db-instances --query 'DBInstances[*].[DBName,DBInstanceIdentifier, DBInstanceClass, StorageType,DbInstancePort,AvailabilityZone,DBInstanceStatus,MasterUsername] ' --output text


---------------
# CPU average and maximum

#!/bin/bash
export AWS_PROFILE=girnar
#export AWS_PROFILE=gaadi
#export AWS_PROFILE=GIBPL
start_time=2020-05-01T11:00:00
end_time=2020-06-03T11:00:00
period=3600


Average()
{
for i in $( cat id.txt )	
do
       	aws cloudwatch get-metric-statistics --region ap-south-1 --metric-name CPUUtilization --start-time $start_time --end-time $end_time --period $period --namespace AWS/RDS --statistics Average --dimensions Name=DBInstanceIdentifier,Value=$i --output text | awk '{print $2}' | sort -nrk1,1  | head -1
done

echo "   " >> file.txt
}


Maximum()
{
for i in $( cat id.txt )
do
        aws cloudwatch get-metric-statistics --region ap-south-1 --metric-name CPUUtilization --start-time $start_time --end-time $end_time --period $period --namespace AWS/RDS --statistics Maximum --dimensions Name=DBInstanceIdentifier,Value=$i --output text | awk '{print $2}' | sort -nrk1,1  | head -1
done
}

truncate -s 0 file.txt
echo "Average CPU"
Average

echo " "
echo "Maximum CPU"
Maximum
--------------------------------------
# Memory Average of RDS, Replace Average with Maximum in case of Maximum

#!/bin/bash
#export AWS_PROFILE=girnar
export AWS_PROFILE=gaadi
#export AWS_PROFILE=GIBPL
start_time=2020-05-01T11:00:00
end_time=2020-06-03T11:00:00
period=3600
region=ap-south-1

for i in $( cat id.txt )	
do
       	aws cloudwatch get-metric-statistics --region $region --metric-name FreeableMemory --start-time $start_time --end-time $end_time --period $period --namespace AWS/RDS --statistics Average --dimensions Name=DBInstanceIdentifier,Value=$i --output text | awk '{print $2}' | sort -nrk1,1  | head -1 | awk '{ byte =$1 /1024/1024/1024; print byte " GB" }' | awk '{print $1}' 
done




```

# Route53 details of all domain
```
sudo apt-get install jq

aws route53 list-hosted-zones|jq '.[] | .[] | .Id' | sed 's!/hostedzone/!!' | sed 's/"//g'> zones
for z in `cat zones`; do    echo $z;    aws route53 list-resource-record-sets --hosted-zone-id $z| jq -r '.ResourceRecordSets[] | [.Name, .Type, (.ResourceRecords[]? | .Value), .AliasTarget.DNSName?] | @csv ' >> records1.csv; done
```
