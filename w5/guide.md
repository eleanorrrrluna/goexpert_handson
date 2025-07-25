# Week 5 Jumphost, ECS and CICD: 
## Level 0 (prerequisites)
**Task**: Create a Key Pair and an EC2 Instance

**Objective**: To create a key pair for SSH access and then create an EC2 instance using this key pair.

### Step by Step Guide:

**Step 1**: Create the key pair
- To create a key pair, use the following command:
```bash
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
```
- This command will create a key pair named `MyKeyPair` and save the private key in a file named `MyKeyPair.pem`.

**Step 2**: Create the EC2 instance
- To create an EC2 instance
   Select "Amazon Linux", and MyKeyPair

**Step 3**: Connect to the EC2 instance
- To connect to the EC2 instance from AWS console
- run `chmod 400 MyKeyPair.pem`, if you get error `Permissions 0644 for 'MyKeyPair.pem' are too open.`
- 点击进入刚创建的那个instance，在主页面找到【connect to instance】，点进来找到【SSH client】,然后复制“ssh -i "MyKeyPair.pem" ec2-user@你刚刚建的instance的Public DNS”到终端里。看到那只鸟的图案就证明成功在终端连上了刚建的ec2虚拟机。
- more to read for Linux file permission in https://www.linuxfoundation.org/blog/blog/classic-sysadmin-understanding-linux-file-permissions

**Step 4**: set up AWS configure
run aws configure, get credential from you administrator user, and set region to ap-southeast-2
另打开一个新的终端页面，回到本地用户而非刚刚链接到ec2 虚拟机。在新的终端页面输入“ls -al ~/.aws”，“cat ~/.aws/credentials”，然后返回ec2 终端界面，把得到的【aws_access_key_id and aws_secret_access_key 】信息粘贴上去。

```bash
aws configure
```
tips:
you may have set up AWS configure in your laptop, you can fetch aws_access_key_id and aws_secret_access_key via
```bash
ls -al ~/.aws
cat ~/.aws/credentials
```


**Step 5**: set up ecs-cli

```bash
sudo curl -Lo /usr/local/bin/ecs-cli https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-linux-amd64-latest
```

```bash
sudo chmod +x /usr/local/bin/ecs-cli
```

```bash
ecs-cli --version
```
**Step 6**: set up git and docker

```bash
sudo yum update -y
```

```bash
sudo yum install git -y
```

```bash
git --version
```

```bash
sudo yum install docker -y
```

```bash
docker --version
```
To read more for Yum in https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-yum#ch-yum

## Level 1 (Intern)
**Task**: Deploy in app in ECS with AWS CLI and AWS console

**Objective**: Create ECR, and push image; create ECS cluster, and run a task

### Step by Step Guide:

**Step 1**: Create ECR

```bash
aws ecr create-repository --repository-name web --region ap-southeast-2
```
check this ECR from AWS console, and "view push commands",

**Step 2**: Clone repo

```bash
git clone https://github.com/AutomationLover/goexpert_handson.git
```

```bash
cd goexpert_handson/w5/repo/
```

**Step 3**: Push image to ECR
follow the cli in "view push commands" in previous Step，create image under folder "goexpert_handson/w5/repo/", and then push this docker image to ECR 。 里面一共有4步：

第一步，Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI: “aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 400071841390.dkr.ecr.ap-southeast-2.amazonaws.com”

但是在第二步 “docker build -t web .”之前，先run “sudo usermod -aG docker ec2-user”，然后“sudo service docker restart”，退出虚拟机再重新ssh connect，应该就不会有docker cli不好使的问题。

记得重新ssh connect到虚拟机后，依然要“cd goexpert_handson/w5/repo/”去到这个目录下，然后再“ocker build -t web .” 。

第三步，After the build is completed, tag your image so you can push the image to this repository:
docker tag web:latest 400071841390.dkr.ecr.ap-southeast-2.amazonaws.com/web:latest

第四步：Run the following command to push this image to your newly created AWS repository:
docker push 400071841390.dkr.ecr.ap-southeast-2.amazonaws.com/web:latest


if some docker issue, please try restart

```bash
sudo service docker restart
```

If some permission issue for docker command, pls add sudo before docker.
If you do not want to add sudo, run 
```bash
sudo usermod -aG docker ec2-user
```
logout and then login

the sudo usermod -aG docker ec2-user command is used to add the user "ec2-user" to the "docker" group, allowing that user to run Docker commands without needing to use sudo or switch to the root user.
- `usermod`: This is the command used to modify a user account. It is typically followed by various options and arguments to specify the modifications to be made.
- `-aG docker`: These options are used to add the user to a specific group. In this case, the -a option indicates that the user should be added to the specified group, and the -G option is used to specify the group name, which is "docker" in this command.

Check ECR repo and you should find image there.

**Step 4**: Create ECS cluster AWS console
Create cluster
Cluster name: myfargate
Create



**Step 5**: Create task-definition
- Task definition family: web
- CPU: 0.25; Memory: 0.5GB

Container 1:
- name:  web
- Image: URL, copy from ECR, format like 839775502365.dkr.ecr.us-west-2.amazonaws.com/web:latest
- Container port: 8000

Click Add Container

Container 2:
- name:  redis
- Image: redislabs/redismod
- Add Port Mappings
- Container port: 6379



**Step 6**: Deploy task to ECS cluster
- Deploy -> Create service 
  - name: myWebCntService
  - Add Load Balancer (OPTIONAL)
   - Type: Application Load Balancer
   - Load balancer name: "myWebCountLB"


If error of IAM do below
- codebuild-my-project-service-role role add permission AmazonEC2ContainerRegistryPowerUser

Click link in create new Task, and click Configuration tab, you can find the Public IP. Then visit this IP with port 8000
http://{public_ip}:8000/hit

Notes:
Decommission process. To remove service/tasks 1st, and then delete Cluster

## Level 2 (skip in class)
**Task**: Create pipeline to build docker and push image to ECR
**Objective**: Learn pipeline to build docker and push image to ECR

**Step 1**: Create repo in CodeCommit


**Step 2**: Push code to CodeCommit
get the code from https://github.com/AutomationLover/goexpert_handson/tree/main/w5/repo

**Step 3**: Create pipeline
update the user id the https://github.com/AutomationLover/goexpert_handson/blob/main/w5/files/buildcommand.yaml

**Step 4**: Update app.py

Update app.py with version

**Step 5**: Run task and check

Run task in ECS cluster and check the version.

## Level 3 (skip in class)
**Task**: Create pipeline to deploy image in ECR to ECS
**Objective**: Learn pipeline to deploy image in ECR to ECS

Please refer to https://docs.aws.amazon.com/codepipeline/latest/userguide/ecs-cd-pipeline.html 
**Step 1**: Create repo in CodeCommit


**Step 2**: Push code to CodeCommit
get the code from https://github.com/AutomationLover/goexpert_handson/tree/main/w5/repo
Replace below, with your own AWS account ID and region, and push the code to CodeCommit repo 
https://github.com/AutomationLover/goexpert_handson/blob/main/w5/repo/buildspec.yml#L11

**Step 3**: Create pipeline
- step 1: select Build custom pipeline
- step 2: give Pipeline name "myCountWebPipeline"
- step 3:
   - select source provider "AWS CodeCommit", and
   - the repo you created before
   - branch "master"
- step 4:
   - Select "Other build providers"
   - Create project "myProject"
      - Select "Use a buildspec file"   
      - Remove the selection of "CloudWatch logs - optional"
      - Click "Continue to CodePipeline"
      - Others refer to "On the Step 3: Add build stage page, for Build provider choose AWS CodeBuild, and then choose Create project." in https://docs.aws.amazon.com/codepipeline/latest/userguide/ecs-cd-pipeline.html 

  
   - Double check the region is the same region of CodeCommit and ECR in previous step
     
- step 5:
   - Deploy provider: select "Amazon ECS"
   - Double check the region is the same region of CodeCommit and ECR in previous step
   - Select Cluster name, which you created in previous step
   - Select Service name, which you created in previous step
 
- step 6
   - Review and Confirm
   - If IAM error, which did not allow visit ECR, The CodePipeline wizard created an IAM role for the CodeBuild build project, called codebuild-build-project-name-service-role. For this tutorial, the name is codebuild-hello-world-service-role. Because the buildspec.yml file makes calls to Amazon ECR API operations, the role must have a policy that allows permissions to make these Amazon ECR calls.  check https://docs.aws.amazon.com/codepipeline/latest/userguide/ecs-cd-pipeline.html#code-build-perms
