# Cloud Computing

29th Sept. 2025



\#What is cloud?



\#what is cloud computing?

-collection of resources present somewhere that can be accessed provided having Internet connection on a pay as you go basis.



\#why cloud computing is important?

-cost effective

-scalable

-capital expenditure not required



\#Benefits of cloud computing

* CAPX
* OPEX
* COST
* SCALE
* TIME
* UPKEEP



\##NOTE - 2 types of scaling : Vertical Scaling(On Premise)

&nbsp;                           : Horizontal Scaling-Auto(Cloud)



\#Types of Cloud Computing



* IAAS - ex: EC2, VPC
* PAAS - ex: AWS Databases, AWS EKS, ECS
* SAAS - ex: Social Media
* FAAS - Function



\##Refer to Notes Hereafter for thoery





### EC2 Instance





\#cat /etc/os-release

\#yum install httpd -y

\# rpm -qa httpd

\#cd /var/www/html/

\#echo "this is my webserver running in aws cloud > index.html

\#cat index.html

\#cd

\#systemctl start httpd

\#systemctl enable httpd



-> copy public ip \& paste along with :80





##### -> Run user data Script

->stopped the instance

->created another but added script in "user data" option present in Advance Details Menu





"

\#!/bin/bash

yum install httpd -y

echo "this is my webserver runnnig in aws cloud" >> /var/www/html/index.html

systemctl start httpd

systemctl enable http

useradd Dev -p devops -c HEHE

"



-> Copy public Ip and paste with :80

-> Then go to terminal to check everything



->Stopped the instance

\# cat /etc/passwd

\# systemctl status httpd



##### ->Now how to reserve Eip

\##refer to notes



-Goto EC2 -> Network \& Security -> Elastic IPs

&nbsp;-> Allocate EIP Address -> Select IP -> Actions -> Associate



Now Public Ip won't change after stopping the instance



##### ->Custom AMI Creation

-1st let's do some things in the instance 

\# useradd Tanjiro -c "Giyu San!"

\# passwd Tanjiro - sun

\# touch Tanjiro.txt{1..2}



->Stop the instance

->Action ->Image \& Template -> Create Image ->Img Name->Description->Add tag-> key:name value -devops-image-custom -> Create Image



##### Creating Instance From Image

->Images Menu ->AMIs -> Select Image -> Launch instance from AMI



then connect and copy ssh but replace 'root' with "ec2-user"



##### ->Copy AMI to another region

->AMIs->Select ->Action ->Copy AMI -> Destination Region -> select region ->Copy AMI





##### -> Share AMI to another account

->AMIs->Permissons->Edit AMI permissions -> Shared accounts -> Add account id -> Save Changes





##### 

## 

## After Lunch

#### ->Template Creation



->go to EC2 Instances page

->Launch Template

&nbsp;->Give name, description ,Tags

->Application \& OS Images -> Select Amazon(As per Requirements)

->Instance type : t3.micro(As per Requirements)

->key pair: select Existing

->subnet : US east 1a

->Security Group : Select Existing

-> Volume : Select as required

-> Advance Details : Add Script as required



->Launch Template

###### 

###### Then Create instance from template



select the Template -> Action -> select number of instanced -> Launch Instances







//New Task



->create an instance



\# df -h

\# lsblk

##### 

##### -> Adding another volume



EC2 Page -> Volumes ->Create volume ->gp2 -> 10gib



->select ->Action ->Attach



\#lsblk

\#blkid

\#mkfs.ext4 /dev/nvme1n1     | to assign file system

\#mkdir /data

\#mount /dev/nvme1n1 /data/     | Temporary Mounting

\#df -h

\#cd /data/

\#touch devops.txt{1..10}

&nbsp;



restart the instance from EC2 Page

goto terminal

start using ssh 



\#df -h       |

\#ll          | to Check

\#ll -d/data  |

\#ll



\#mount /dev/nvme1n1 /data/

\#cd /data/

\#ll



blkid -> copy UUID

UUID=06ab3daa-e3a6-4c1e-b447-31e0d65b5cc1       /data   ext4    defaults        0 0   

vim /etc/fstab -> Paste   | Permanent Mounting



reboot instance



again open terminal

\# cd /data/

\# ll



#### 

#### ///How to unmount

\# vim etc/fstab

comment out the UUID line

\#systemctl daemon-reload

\#df -h

\#umount /data

\#df -h

\#lsblk



->GO to EC2 page volumes select the volue -> action ->Detach







#### How to increase size of volume on the fly



->Increase Volume by Modifying Volume from EC2 Volume Page



\#resize2fs /dev/nvme1n1     |Only works for ext4

\#df -h

\#lsblk

\# cd /data

\#ll









#### Increase size of root volume on the fly



->Increase Volume by Modifying Volume from EC2 Volume Page

\#growpart /dev/nvme0n1p1

\# xfs\_growfs /

\#df -h

\#lsblk





