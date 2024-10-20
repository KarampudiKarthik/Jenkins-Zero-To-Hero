## Job Triggers

it is used for automation (eg: build now button in jenkins)

popular triggers
1. git webhook
2. poll SCM
3. scheduled jobs
4. remote triggers
5. build after other project are built

pre reques
1. create git repo
2. ssh auth
3. create a jenkin file in git repo and commit
4. create a jenkin job to access jenkins file from git repo
5. test triggers

### 1. create git repo
1. create git repo
2. open git bash command prompt in windows
```
ssh-keygen.exe
```
it will generate public and private key

3. to check the keys
```
$ ls ~/.ssh/
```

4. copy the public key using this command
```
$ cat ~/.ssh/id_ed25519.pub
```

5. go to git account settings > SSH and GPG keys > new SSH key > paste the public key
6. go to the newly created git repo anc click on SSH and copy the link beside it
7. go to git bash on windows
8. create directory
```
$ mkdir -p /f/gitrepos
```

```
cd /f/gitrepos/
```
we need to paste the url which we copied in step 6
```
git clone git@github.com:KarampudiKarthik/jenkinsTriggers.git
```

```
cd jenkinsTriggers/
```

9. save a simple jenkinfile in gitrepo path in windows
```
pipeline {
	agent any
	stages {
		stage('Build') {
			steps{
				sh 'echo "Build completed."'
			}
		}
	}
}
```
save the file as all types. if the file is shows like Jenkinsfile.txt. then we need to change to ajenkinsfile
```
mv Jenkinsfile.txt Jenkinsfile
```

```
git add .
```

```
git commit -m "first commit"
```

```
git push origin main
```

it will add the Jenkinsfile in git

### to avoid host key verification failed in jenkins
go to jenkins > manage jenkins > security > change  Git Host Key Verification Configuration - Apply first connect


