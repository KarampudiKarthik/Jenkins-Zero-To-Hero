## install jenkins 
docu: https://www.jenkins.io/doc/book/installing/linux/

look at readme file in jenkins folder

create ec2
```
sudo apt update
sudo apt install openjdk-17-jre

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins
```
password location: /var/lib/jenkins/secrets/initialAdminPassword
location: /var/lib/jenkins/

##install tools and plugins
1.open manage jenkins>tools>click on add jdk
>name:OpenJDK17
>>path:/usr/lib/jvm/java-1.17.0-openjdk-amd64
Note: version of name and path will be same. for path and name and versioncheck in ec2 instance

2. add maven
   >name:MAVEN3

after open jenkins and go to manage jenkins >plugins download docker pipeline and restart

environmen variables: https://wiki.jenkins.io/display/JENKINS/Building+a+software+project







