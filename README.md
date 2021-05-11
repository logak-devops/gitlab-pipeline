# gitlab-pipeline


Gitlab server, GitLab runner setup and maven CI pipeline to push the docker image to ECR 
Below are the high-level steps:
1.	Install GitLab server
2.	Install GitLab-runner
3.	Register GitLab-runner using token
4.	Create ECR repository
5.	Import the project and run pipeline
6.	Run the container for checking the the docker image 


# 1.Install GitLab server: 
I am using the Amazon Linux Instance, but depending on your OS flavor you can choose the repo from below website:
https://packages.gitlab.com/gitlab/gitlab-ce

Install the required dependencies :

yum install curl policycoreutils-python openssh-server  -y

Download git repo and install it :

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
EXTERNAL_URL="`curl http://169.254.169.254/latest/meta-data/public-ipv4`" yum install -y gitlab-ce

Note : 
Once you installed the GitLab package, you can execute the required configuration utility. This file provides automatic configurations, and you can modify it according to your need. Run the following edit of the GitLab configuration file.

$ sudo vim /etc/gitlab/gitlab.rb

Now, edit the configuration file to change hostname using external_url variable so that, you can access them from other remote machine using the specified hostname and other parameters:
Run the following command to reconfigure the services of GitLab:

$ sudo gitlab-ctl reconfigure

Open the port number 80 as gitlab is running on port 80.

Check the GitLab UI from browser:

http://public_ip_of_ec2_instance or  http://18.132.245.177

To access the GitLab web portal that will ask you to set the username and password of root. Enter the new root password. After verifying then, click the ‘Change your password’ option.

Now, login with the username as root and then provide the password. You will see the following gitlab dashboard screen on your system.


![image](https://user-images.githubusercontent.com/84037413/117859012-654f7500-b286-11eb-9fc2-0e8a35bc6ad5.png)

![image](https://user-images.githubusercontent.com/84037413/117859768-54533380-b287-11eb-88ad-6058dc0ad47e.png)

 
# 2.Install GitLab-runner

https://docs.gitlab.com/runner/install/linux-repository.html#installing-the-runner

-	Add GitLab official repository:

curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash

-	Install the latest version of GitLab Runner, or skip to the next step to install a specific version:
-	
export GITLAB_RUNNER_DISABLE_SKEL=true; sudo -E yum install gitlab-runner

-	To install a specific version of GitLab Runner:
-	

yum list gitlab-runner --showduplicates | sort -r
export GITLAB_RUNNER_DISABLE_SKEL=true;sudo -E yum install gitlab-runner-10.0.0-1
 

# 3. Register runner

For registering the runner, we need to get token from GitLab server first.
To get the token go to projects -> select project -> settings -> CI/CD Runners 

![image](https://user-images.githubusercontent.com/84037413/117860410-2b7f6e00-b288-11eb-9db8-eb41609e56d5.png)


Copy token and url

![image](https://user-images.githubusercontent.com/84037413/117860641-6f727300-b288-11eb-8e29-1c440a02c73e.png)


#There are multiple types of runner executer like shell, docker, k8s. For our use case we will use docker executor. 

First install Docker on the server where we are running GitLab Runner.
# yum install docker -y
# service docker start
# chmod 666 /var/run/docker.sock
```
sudo gitlab-runner register -n \
    	--url http://18.132.245.177/ \
    	--registration-token wy79Ym3d37mLD6ckApZY \
   	--executor docker \
    	--description "My Docker Runner1" \
        --docker-image "docker:stable" \
	--docker-privileged
```
#You can get the list of registered runners:

# gitlab-runner list


![image](https://user-images.githubusercontent.com/84037413/117861190-0fc89780-b289-11eb-99b8-697a1d819042.png)

![image](https://user-images.githubusercontent.com/84037413/117861314-34247400-b289-11eb-8b92-d3d4b773536e.png) 

# 4.Create ECR repository
#Goto ECR repository page and create repository with the name : springdemo

![image](https://user-images.githubusercontent.com/84037413/117861914-e52b0e80-b289-11eb-9429-955a07681717.png)


# 5. Import the project and commit it

#Goto Projects in GitLab and do import project (from github)and specify the below git url to import repo.

![image](https://user-images.githubusercontent.com/84037413/117861672-967d7480-b289-11eb-9b69-e3778df5f29a.png)

![image](https://user-images.githubusercontent.com/84037413/117862268-62568380-b28a-11eb-83f9-61da7e634efd.png)

![image](https://user-images.githubusercontent.com/84037413/117862384-80bc7f00-b28a-11eb-97c9-bc650296d65c.png)

#Github Personal Token

![image](https://user-images.githubusercontent.com/84037413/117863325-8cf50c00-b28b-11eb-93fb-bc04bef7acc8.png)

	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

#Provide that token in Gitlab:

![image](https://user-images.githubusercontent.com/84037413/117863595-d9404c00-b28b-11eb-9617-0655620f7eb4.png)

![image](https://user-images.githubusercontent.com/84037413/117863677-f248fd00-b28b-11eb-97a6-94da0b533f1e.png)

# Link for git repo
[Springoot](https://github.com/logambigaik/springboo-without-db.git)
 
 

 
#Now goto CI/CD  Variables  and add the following AWS variables to access the password of ECR and login to it.

![image](https://user-images.githubusercontent.com/84037413/117864841-2670ed80-b28d-11eb-8f08-cbd82d4be1ff.png)

```

 Variable			Allowed Values
AWS_ACCESS_KEY_ID		Any(access_key_id)
AWS_DEFAULT_REGION		Any(eu-west-2)
AWS_SECRET_ACCESS_KEY		Any(access_key)
```
 
![image](https://user-images.githubusercontent.com/84037413/117865170-8e273880-b28d-11eb-92ac-5f352e94ebcc.png)


#Once repo is imported, edit the file .gitlab-ci.yml update the DOCKER_REGISTRY: with your ECR repo url and commit it. Once you commit the repo it will auto trigger the pipeline.
					Or 
#If you want you can goto projects  CI/CD  pipeline  run project. 


![image](https://user-images.githubusercontent.com/84037413/117867310-1a3a5f80-b290-11eb-83a2-453f097dd75e.png)

#You can see the stages and status as below :
 
![image](https://user-images.githubusercontent.com/84037413/117865555-fc6bfb00-b28d-11eb-927d-f650e5e662a8.png)


#Once pipeline is finished you can verify if image is pushed to your ECR repo or not.

![image](https://user-images.githubusercontent.com/84037413/117880886-845b0080-b2a0-11eb-9570-95aa229d4dfc.png)


 
#To verify if image is created correctly or not, we can pull the image run the docker container using below command:

# 6  aws configure
```
AWS Access Key ID [None]: AKIAT6DSXXXXXXXXXXX
AWS Secret Access Key [None]: 5raxCB4Ymy8MXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: us-east-1
Default output format [None]: json
```

[root@ip-172-31-67-134 opt]# aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin 136962450893.dkr.ecr.eu-west-2.amazonaws.com/springdemo

![image](https://user-images.githubusercontent.com/84037413/117877709-caae6080-b29c-11eb-85e7-7bc29b28d713.png)

![image](https://user-images.githubusercontent.com/84037413/117879796-45787b00-b29f-11eb-8005-e557be8318ae.png)



# 7: Pulling from springdemo

	docker pull 136962450893.dkr.ecr.eu-west-2.amazonaws.com/springdemo:10

![image](https://user-images.githubusercontent.com/84037413/117879904-650fa380-b29f-11eb-9e2c-52948c54a4a2.png)

```
[root@ip-172-31-9-241 opt]# docker pull 136962450893.dkr.ecr.eu-west-2.amazonaws.com/springdemo:10
10: Pulling from springdemo
050382585609: Pull complete
a8c71082b2bb: Pull complete
4180c4b68a4e: Pull complete
Digest: sha256:2d41b5160094d46d97b39ba8c8432f176970440702dc0369b75a7d73444cdc7c
Status: Downloaded newer image for 136962450893.dkr.ecr.eu-west-2.amazonaws.com/springdemo:10
136962450893.dkr.ecr.eu-west-2.amazonaws.com/springdemo:10
```


# docker ps
```
[root@ip-172-31-9-241 opt]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@ip-172-31-9-241 opt]# docker images
REPOSITORY                                                TAG               IMAGE ID       CREATED         SIZE
136962450893.dkr.ecr.eu-west-2.amazonaws.com/springdemo   10                e86b8fe24a03   8 minutes ago   354MB
gitlab/gitlab-runner-helper                               x86_64-7f7a4bb0   4763454e021f   2 hours ago     70.2MB
docker                                                    19-dind           c0272ea5b8a2   10 days ago     236MB
docker                                                    dind              dc8c389414c8   10 days ago     263MB
maven                                                     3-jdk-8           87963037f00b   2 weeks ago     525MB
docker                                                    latest            d2979b152a7d   3 weeks ago     246MB
```

# docker run -it -p 8081:33333 -d 136962450893.dkr.ecr.eu-west-2.amazonaws.com/springdemo:10


# docker ps

![image](https://user-images.githubusercontent.com/84037413/117880356-ec5d1700-b29f-11eb-90c0-8a4d2dac0fef.png)


#Open port 8081 and access the http://18.132.245.177:8081/listallcustomers url to check if image working properly or not.

![image](https://user-images.githubusercontent.com/84037413/117880496-17e00180-b2a0-11eb-8eb1-b6d6777f032c.png)
 
 
