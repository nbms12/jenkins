AIM :  Design and implment complete CI/CD setup for Hotstar app (Entertainment application ) into tomcat server


Tools installations and Configurations 

1 . ##jenkins install 

Jenkins Setup - Amazon Linux 2 AMI, t2.micro

- Integrate Git & GitHub
 
- Integrate Maven
 
Tomcat Setup - Amazon Linux 2 AMI, t2.micro

SonarQube  Setup - Amazon Linux 2 AMI, t2.medium (Billable)

Nexus Setup - Ubuntu 24.04 AMI, t4.medium

#STEP-1: Installing Git and Maven

yum install git maven -y

#STEP-2: Repo Information (jenkins.io --> download -- > redhat)

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

#STEP-3: Download Java 17 and Jenkins

sudo yum install java-17-amazon-corretto -y

yum install jenkins -y

#STEP-4: Start and check the JENKINS Status

systemctl start jenkins.service

systemctl status jenkins.service

#STEP-5: Auto-Start Jenkins

chkconfig jenkins on




2. ##SonarQube install

cd /opt/

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip

unzip sonarqube-8.9.6.50800.zip

amazon-linux-extras install java-openjdk11 -y

useradd sonar

chown sonar:sonar sonarqube-8.9.6.50800 -R

chmod 777 sonarqube-8.9.6.50800 -R
su - sonar


Accesss: 

Open Port no. 9000 for the server and setup SonarQube 
Username: admin

Password: admin

3. ##Tomcat install and configure

sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.73/bin/apachetomcat-9.0.73.tar.gz

tar -xvf apachetomcat-9.0.73.tar.gz

start tomacat under bin folder /bin/startup.sh

under webapps/manager/META-INF/context.xml > vi context.xml > valve" tag edit ".*"

Configuring the Users in Tomcat

<role rolename="manager-gui" />

<user username="tomcat" password="tomcat" roles="manager-gui" />

<role rolename="admin-gui" />

<role rolename="manager-script" />

<user username="admin" password="admin" roles="manager-gui,admin-gui,manager-script"/>

cd bin
./shutdown.sh
./startup.sh


change port no for tomcat server inside server.xml file under connector Tag 9090 
access tomcat under tis port. 

4. ##install and configure nexus

before tis need to install java and jdk 

##Download the SonaType Nexus on Ubuntu using wget

cd /opt 

https://download.sonatype.com/nexus/3/nexus-3.80.0-06-linux-x86_64.tar.gz   


##Extract the Nexus repository setup in /opt directory

tar -zxvf latest-unix.tar.gz     

##Rename the extracted Nexus setup folder to nexus

sudo mv /opt/nexus-3.77.0-08/ /opt/nexus

##As security practice, not to run nexus service using root user, so lets create new user named

nexus to run nexus service

sudo adduser nexus  ----> give password i.e root1234

Press ENTER for 5 times
Press Y
You will see 'Adding user 'nexus' to group 'users'



##To set no password for nexus user open the visudo file in ubuntu

sudo visudo

##Add below line after  "Defaults" 		use_pty

nexus ALL=(ALL) NOPASSWD: ALL

Save and exit (Control + X, Press Y, Click Enter)


##Give permission to nexus files and nexus directory to nexus user

sudo chown -R nexus:nexus /opt/nexus

sudo chown -R nexus:nexus /opt/sonatype-work        



##To run nexus as service at boot time, open /opt/nexus/bin/nexus.rc file, uncomment it and add 

nexus user as shown below

sudo nano /opt/nexus/bin/nexus.rc

Uncomment the line which you see after opening the file

In the double quotes, keep "nexus". The finale line should be as shown below

run_as_user="nexus"   

Save and exit (Control + X, Press Y, Click Enter)



##To Increase the nexus JVM heap size, you can modify the size as shown below

vi /opt/nexus/bin/nexus.vmoptions

Remove the first 15 lines i.e upto the space just before the "#additional vmoptions needed for

java9+". Remember, dont delete "#additional vmoptions needed for java9+" line

Paste the below content above "#additional vmoptions needed for java9+" line

-Xms1024m

-Xmx1024m

-XX:MaxDirectMemorySize=1024m

-XX:LogFile=./sonatype-work/nexus3/log/jvm.log

-XX:-OmitStackTraceInFastThrow

-Djava.net.preferIPv4Stack=true

-Dkaraf.home=.

-Dkaraf.base=.

-Dkaraf.etc=etc/karaf

-Djava.util.logging.config.file=/etc/karaf/java.util.logging.properties

-Dkaraf.data=./sonatype-work/nexus3

-Dkaraf.log=./sonatype-work/nexus3/log

-Djava.io.tmpdir=./sonatype-work/nexus3/tmp 

Save and exit



##To run nexus as service using Systemd

sudo vi /etc/systemd/system/nexus.service ----> Paste the below content ---->


[Unit]

Description=nexus service

After=network.target

[Service]

Type=forking

LimitNOFILE=65536

ExecStart=/opt/nexus/bin/nexus start

ExecStop=/opt/nexus/bin/nexus stop

User=nexus

Restart=on-abort

[Install]
WantedBy=multi-user.target 

Save and exit



##To start the nexus Service

sudo systemctl start nexus

sudo systemctl enable nexus 

sudo systemctl status nexus  

Open port no. 8081 and access nexus




5. ##Install below plugins in Jenkins

Pipeline stageview,

Deploy to container,

S3 publisher,

nexus artifact uploader,

SonarQube Scanner,

Sonar Quality Gate, 

Maven Integration



6. #Setup Tomcat Creds in Jenkins

Username with Password, Scope: Global, username: admin, Password: admin,


7.#Open Pipeline Syntax in new tab 

----> Sample step: deploy: Deploy war/ear to a container, WAR/EAR Files: **/*.war, Context Path: hotstarapp, Add container: Select 'Tomcat 9.x Remote', Credentials: Select the tomcat creds from dropdown, Tomcat URL: Paste upto 8080/ ----> Generate pipeline script ----> Copy and paste in 'Deploy to Tomcat' stage

  8. #Working with SonarQube
     
Goto SonarQube ----> Projects ----> Add project ----> Manually ----> Project key: hotstar, Display name: hotstar ----> Setup ----> Generate a token: hotstar ----> Generate ----> Copy the token ----> Lets configure the SonarQube credentials in Jenkins

sonarqube creds create : 

Jenkins ----> Manage Jenkins ----> Security ----> Credentials ----> Click on 'global' ----> Add creds ----> Kind: Secret text, Scope: Global, Secret: Paste the secret copied from SonarQube, ID: sonar, Description: sonar ----> Create

add sonarqube inside jenkins server: 

Jenkins ----> Manage Jenkins ----> System Configuration ----> System ----> Scroll down to 'SonarQube servers' ----> 'Check' environment variables, Add SonarQube ----> Name: SonarQube, Server URL: <SonarQube URL> [only upto 9000] ----> Server authentication token: Select 'sonar' from dropdown ----> Apply ----> Save

Jenkins ----> Manage Jenkins ----> System Configuration ----> Tools ----> Scroll down to 'sonarqube Scanner Installations' ----> Add SonarQube scanner ----> Name: sonarscanner, 'Check' Install automatically, Version: 7.1.0.4889 ----> Apply ----> Save

Lets integrate SonarQube in the pipeline ----> Open the job ----> Paste the below script (In the script, i have added sonarqube-scanning' stage before the artifacts stage)

insert a new stage

```
 pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git '<Repo URL>'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        } 
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Scanning') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('Artifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                // <Paste The Script>
            }
        }
        stage('Nexus') {
            steps {
                // <Paste The Script>
            }
        }
        stage('Artifact to S3') {
            steps {
                // <Paste The Script>
            }
        }
    }
}

```
8. #Nexus Configuration

Goto Nexus tab in browser ----> Signin ----> Username: admin, To get the password, copy the path seen in the nexus browser ----> Goto the nexus connected tab in Moba ----> cat <Paste The Path> ----> Copy the password. dont copy "root@nexus" ----> Paste the password in nexus browser ----> Signin ----> Set the new password ----> Enable anonymous access ----> Signin

Click on settings icon ----> Click on 'repositories' ----> Create repo ----> Scroll down and Click on 'maven2 hosted' ----> Name: hotstar, Version policy: Snapshot, Layout policy: Strict, Content disposition: Inline, Blob store: default, Deployment policy: Allow redeploy ----> Create repository ----> You will see 'hotstar' repo ----> Lets configure the Nexus credentials in Jenkins

Jenkins ----> Manage Jenkins ----> Security ----> Credentials ----> Click on 'global' ----> Add creds ----> Kind: Username with Password, Scope: Global, Username: admin, Password: <Enter The Password of SonarQube>,ID: nexus, Description: nexus ----> Create


Apply and Build the job


Output : 

sonarqube 

![image](https://github.com/user-attachments/assets/6786c2af-19aa-4a8c-925b-0c48f4f0211a)

Build artifacts are stored into nexus artifact repo 

![image](https://github.com/user-attachments/assets/a47eb814-83f8-4688-b5d6-8ffabe10b9fc)


app deployed in tomcat server successfully . 

![image](https://github.com/user-attachments/assets/5fac86d8-c749-4e13-ab7e-aab4705c105c)


-----END OF PROJECT----
