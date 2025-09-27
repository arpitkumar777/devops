26th Sept Full



## step 1:

##### New Jenkins-server Setup Due To Update in Jenkins



useast 1a

Custom tcp Port 8080

12 gib gp2



Jenkins Terminal

\# hostnamectl set-hostname jenkins.example.com

\# bash

\# rpmquery Jenkins

\# yum update

\# yum repolist all



yum update -y

yum install java-21-amazon-corretto -y

 

  sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

 

rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

 

sudo yum install jenkins -y \*

sudo yum install jenkins

 

sudo systemctl start jenkins

sudo systemctl enable jenkins



networking -> copy public ip with :8080

open in a new tab

copy the command

cat /

paste it in terminal to get the password

install the plugins(Left Option)

http://3.238.79.123:8080/



built-in-node (offline)

\# df -h -> disk info

we have to increase the tempfs space to solve this issue



\# sudo mount -o remount,size=2G /tmp // temporary

\# df -h

\# vim /etc/fstab



down arrow

tmpfs /tmp tmpfs defaults,noatime,mode=1777,size=2G 0 0

:wq



\# systemctl daemon-reload

\# reboot (don't in prod env)

then use id and password



## step 2:

##### Create Git-clone-server

useast 1a

Same security group as jenkins-server

12gib t3



\# hostnamectl set-hostname gitClone-server.example.com

\# cd

\# ssh-keygen

\# cat .ssh/id\_rsa.pub

 -> Copy the key and paste it in Guthub settings ssh key section





\# mkdir /dev-data

\# cd /dev-data/

\# yum install git

\# git init

\# git clone https://github.com/sanjayguruji/devops-mumbai.git

\# ll

\# cd devops-mumbai

\# git remote remove origin

\# git -m main

\# git remote add origin git@github.com:dev-debaprasad/jenkis-practice-01.git

<!--SSH URL Of New Repo-->

\# git add .

\# git commit -m clone-done -a

\# git push origin main



## step 3:

##### Link GitHub Repo With Jenkins-Site



->git hub ->repo->settings->webhooks->payload url:

paste the public ip of Jenkins server along with :8080/github-webhook/



->content type - application/json



->Jenkins -> profile -> security -> Api token

create give name copy then go to webhook and paste in secret and then save and reload



-> Webhook should be ticked now



## step 4:

##### Pipeline Building



->Goto Jenkins-server Terminal

-># yum install maven -y



-> then go to Jenkins site and install plugin



 - GitHub integration



 - maven integratiom



 - deploy to container



-> go to Jenkins site settings tool

  -add Java and Mvn path (untick automatic installation)

  -> goto Jenkins-server Terminal

   # mvn -v



-> go to Jenkins site

 -> Create a new job



    - Maven Project

    - paste Github link

    - branch main



     - Trigger - Build whenever a SNAPSHOT dependency is  built :uncheck



             - GitHub hook trigger for GITScm polling :Check



-> Go to EC2 Page

 -> Stop ec2 instance

 -> Change the ec2 instance's Size from t3micro to t3small

 -> start the instance

 -> copy the public ip



-> Goto Jenkins-server Terminal



  # cd /var/lib/Jenkins

  # ll

  # vim jenkins.model.JenkinsLocationConfiguration.xml

  -> Replace the existing ip



  # systemctl restart Jenkins



-> goto Jenkins site

  and login then Pray to God and hit Build Now



## step 5:

##### Tomcat Server Configuration



Create new EC2 instance:

us east 1a

13gib t3

New security group 8080





\# hostnamectl set-hostname tomcat.example.com

\# bash

\# yum install java\\\* -y

\# yum install wget -y

\# yum install wget

\# wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.109/bin/apache-tomcat-9.0.109.tar.gz

\# tar -xvzf apache-tomcat-9.0.109.tar.gz

\# ll

\# cd apache-tomcat-9.0.109

\# cd bin/

\# ll

\# chmod +x startup.sh

\# chmod +x shutdown.sh

\# ./startup.sh

-> Copy Public Ip paste with :8080 to see Tomcat Default Page

\# ./shutdown.sh

\# cd ..

\# cd conf

\# vim tomcat-users.xml



goto -> https://medium.com/@raguyazhin/step-by-step-guide-to-install-apache-tomcat-on-amazon-linux-120748a151a9

 -> Copy



Update user information in tomcat-users.xml

\\#Add below lines between <tomcat-users> tag



<role rolename="manager-gui"/>

<role rolename="manager-script"/>

<role rolename="manager-jmx"/>

<role rolename="manager-status"/>   

<user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status"/>

<user username="deployer" password="deployer" roles="manager-script"/>

<user username="tomcat" password="s3cret" roles="manager-gui"/>



\# cd ..

\# find -name context.xml

 output:

 ./conf/context.xml

 ./webapps/docs/META-INF/context.xml

 ./webapps/examples/META-INF/context.xml

 ./webapps/host-manager/META-INF/context.xml

 ./webapps/manager/META-INF/context.xml



->Now We have to comment out valve statement of last 3 files



 # vim webapps/examples/META-INF/context.xml

 # vim webapps/host-manager/META-INF/context.xml

 # vim webapps/manager/META-INF/context.xml

 

\# cd ..

\# cd bin/

\# ./shutdown.sh

\# ./startup.sh



-> go to jenkins site

 ->settings(Manage Jenkins) ->Security ->credentials -> Global credentials -> Add credentials -> userid and pass - deployer

 ->Open the pipeline(Project) ->configuration->Build Setting->Post Build Actions ->select "deploy war file to container"

 -> inside war file : "\*\*/\*.war" -> inside context path : "/"

 -> Then in next field : crendentials - deployer/\*\*\*\*\* tomcaturl - copy paste

 -> Apply ->Save -> Build Now



-> copy Tomcat server ip and open in a new tab then

