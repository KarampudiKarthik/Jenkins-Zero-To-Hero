## Deploy Docker image (CI / CD)

ECS- docker containing hosting platform

docker containing hosting platform
1. Docker engine (it doesn't have production level features like self healing, high availability)
2. kubernetes

### pre reques
1. create ECS cluster
2. create ECS service
3. install plugin `pipeline: aws steps


### 1. setup ECS cluster
1. go to ECS > clusters > create cluster

![image alt](https://github.com/KarampudiKarthik/Jenkins-Zero-To-Hero/blob/main/my/img/Capture2.PNG?raw=true)

2. create new task definition
> url from ECR

![image alt](https://github.com/KarampudiKarthik/Jenkins-Zero-To-Hero/blob/main/my/img/Capture3.PNG?raw=true)

![image alt](https://github.com/KarampudiKarthik/Jenkins-Zero-To-Hero/blob/main/my/img/Capture4.PNG?raw=true)


3. go to iAM > users> ecstaskexecutionrole and allow cloudwatchlogsfullaccess

![image alt](https://github.com/KarampudiKarthik/Jenkins-Zero-To-Hero/blob/main/my/img/Capture6.PNG?raw=true)

4. go to ECS > cluster > our project cluster(name) > create service
   
(i) scroll down to deployment configuration

1. family- vprofileapptask
2. revision - latest
3. service name - vprofileappsvc
4. uncheck deployment failure detection

(ii) go to networking

1. create new security group
> name - vproappecselb-sg
2. inbound rules

type-http

source- anywhere

add another rule

type- custom tcp

port - 8080

source- anywhere

(iii) go to load balancer

select application load balancer
> name - vproappelnecs

target group name -vproecstg

health check path - /login

```
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

    environment {
        registryCredential = 'ecr:us-east-2:awscreds'
        appRegistry = "951401132355.dkr.ecr.us-east-2.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://951401132355.dkr.ecr.us-east-2.amazonaws.com"
        cluster = "vprofile"
        service = "vprofileappsvc"
    }
  stages {
    stage('Fetch code'){
      steps {
        git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
      }
    }


    stage('Test'){
      steps {
        sh 'mvn test'
      }
    }

    stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('build && SonarQube analysis') {
            environment {
             scannerHome = tool 'sonar4.7'
          }
            steps {
                withSonarQubeEnv('sonar') {
                 sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
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

    stage('Build App Image') {
       steps {
       
         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
             }

     }
    
    }

    stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
     }
     
     stage('Deploy to ecs') {
          steps {
        withAWS(credentials: 'awscreds', region: 'us-east-2') {
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
      }
     }

  }
}
```
