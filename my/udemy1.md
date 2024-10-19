## ## install jenkins local
1. download jdk and extract
follow this video: https://www.youtube.com/watch?v=BzjAMcV-U2w&ab_channel=javafrm
2. download jenkins
follow this video: https://www.youtube.com/watch?v=Zdxko2bPAAw&ab_channel=ProgrammingKnowledge

login: http://localhost:8080/
>for password go to the path as shown in screen

>in manage jenkins> plugins> install docker pipeline
>>manage jenkins> tools> add jdk, Git along with path

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

## install tools and plugins
Install pugins : Nexus Artifact Uploader,SonarQube Scanner,Build Timestamp,Pipeline Maven Integration,Pipeline Utility Steps

1.open manage jenkins>tools>click on add jdk
>name:OpenJDK17

>path:/usr/lib/jvm/java-1.17.0-openjdk-amd64

Note: version of name and path will be same. for path and name and versioncheck in ec2 instance

2. add maven
   
   name:MAVEN3

after open jenkins and go to manage jenkins >plugins download docker pipeline and restart

environmen variables: https://wiki.jenkins.io/display/JENKINS/Building+a+software+project

## Jenkins,Nexus and Sonarqube setup

link: https://github.com/hkhcoder/vprofile-project/tree/ci-jenkins/userdata

we can also do it in local
>link: https://www.youtube.com/watch?v=TRI-GfkCNeE&ab_channel=DevOpsVijay

url for nexus: http://localhost:8081

url for sonaeqube: http://localhost:9000

## Pipeline as a code

Example: to build, fetch, save, test
```
pipeline {
	agent any
	tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

	stages {
	    stage('Fetch code') {
            steps {
               git branch: 'vp-rem', url: 'https://github.com/devopshydclub/vprofile-repo.git'
            }

	    }

	    stage('Build'){
	        steps{
	           sh 'mvn install -DskipTests'
	        }

	        post {
	           success {
	              echo 'Now Archiving it...'
	              archiveArtifacts artifacts: '**/target/*.war'
	           }
	        }
	    }

	    stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }
	}
}
```


Example: to build, fetch, save, test, checkstyle Analysis
```
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

    stages{
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

    }
}
```

Example: to build, fetch, save, test, checkstyle analysis, sonar analysis

```
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

    }
}
```

Example: to build, fetch, save, test, checkstyle analysis, sonar analysis,quality gate
```
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }



    }
}
```
