# End-to-End DevSecOps Kubernetes Three-Tier Project using AWS EKS, ArgoCD, Prometheus, Grafana, and Jenkins

# **Project Introduction:**

Welcome to the End-to-End DevSecOps Kubernetes Project guide! In this comprehensive project, we will walk through the process of setting up a robust Three-Tier architecture on AWS using Kubernetes, DevOps best practices, and security measures. This project aims to provide hands-on experience in deploying, securing, and monitoring a scalable application environment.

# **Project Overview:**

In this project, we will cover the following key aspects:

1. **IAM User Setup**: Create an IAM user on AWS with the necessary permissions to facilitate deployment and management activities.
2. I**nfrastructure as Code (IaC):** Use Terraform and AWS CLI to set up the Jenkins server (EC2 instance) on AWS.
3. **Jenkins Server Configuration**: Install and configure essential tools on the Jenkins server, including Jenkins itself, Docker, Sonarqube, Terraform, Kubectl, AWS CLI, and Trivy.
4. **EKS Cluster Deployment**: Utilize eksctl commands to create an Amazon EKS cluster, a managed Kubernetes service on AWS.
5. **Load Balancer Configuration**: Configure AWS Application Load Balancer (ALB) for the EKS cluster.
6. **Amazon ECR Repositories**: Create private repositories for both frontend and backend Docker images on Amazon Elastic Container Registry (ECR).
7. **ArgoCD Installation**: Install and set up ArgoCD for continuous delivery and GitOps.
8. **Sonarqube Integration**: Integrate Sonarqube for code quality analysis in the DevSecOps pipeline.
9. **Jenkins Pipelines**: Create Jenkins pipelines for deploying backend and frontend code to the EKS cluster.
10. **Monitoring Setup**: Implement monitoring for the EKS cluster using Helm, Prometheus, and Grafana.
11. **ArgoCD Application Deployment**: Use ArgoCD to deploy the Three-Tier application, including database, backend, frontend, and ingress components.
12. **DNS Configuration**: Configure DNS settings to make the application accessible via custom subdomains.
13. **Data Persistence**: Implement persistent volume and persistent volume claims for database pods to ensure data persistence.
14. **Conclusion and Monitoring:** Conclude the project by summarizing key achievements and monitoring the EKS cluster’s performance using Grafana.

# **Prerequisites:**

Before starting the project, ensure you have the following prerequisites:

- An AWS account with the necessary permissions to create resources.
- Terraform and AWS CLI installed on your local machine.
- Basic familiarity with Kubernetes, Docker, Jenkins, and DevOps principles.

# **Step 1: We need to create an IAM user and generate the AWS Access key**

Create a new IAM User on AWS and give it to the AdministratorAccess for testing purposes (not recommended for your Organization's Projects)

Go to the AWS IAM Service and click on **Users.**

[https://miro.medium.com/v2/resize:fit:1400/0*V9SD-s0nd8UGjjol](https://miro.medium.com/v2/resize:fit:1400/0*V9SD-s0nd8UGjjol)

Click on **Create user**

[https://miro.medium.com/v2/resize:fit:1400/0*YkmJMmPJLp_C3cPZ](https://miro.medium.com/v2/resize:fit:1400/0*YkmJMmPJLp_C3cPZ)

Provide the name to your user and click on **Next.**

[https://miro.medium.com/v2/resize:fit:1400/0*JHAaZKv7GxGK_nLk](https://miro.medium.com/v2/resize:fit:1400/0*JHAaZKv7GxGK_nLk)

Select the **Attach policies directly** option and search for **AdministratorAccess** then select it**.**

Click on the **Next.**

[https://miro.medium.com/v2/resize:fit:1400/0*WkJnqN_wwmSwaaaC](https://miro.medium.com/v2/resize:fit:1400/0*WkJnqN_wwmSwaaaC)

Click on **Create user**

[https://miro.medium.com/v2/resize:fit:1400/0*rqG8tMLvYrebO2FE](https://miro.medium.com/v2/resize:fit:1400/0*rqG8tMLvYrebO2FE)

Now, Select your created user then click on **Security credentials** and generate access key by clicking on **Create access key.**

[https://miro.medium.com/v2/resize:fit:1400/0*2fc2AxuLIV7jmbOR](https://miro.medium.com/v2/resize:fit:1400/0*2fc2AxuLIV7jmbOR)

Select the **Command Line Interface (CLI)** then select the checkmark for the confirmation and click on **Next.**

[https://miro.medium.com/v2/resize:fit:1400/0*aZMrjWaoxKsyQ7Xm](https://miro.medium.com/v2/resize:fit:1400/0*aZMrjWaoxKsyQ7Xm)

Provide the **Description** and click on the **Create access key.**

[https://miro.medium.com/v2/resize:fit:1400/0*sHZC9cDdCEyiHSLS](https://miro.medium.com/v2/resize:fit:1400/0*sHZC9cDdCEyiHSLS)

Here, you will see that you got the credentials and also you can download the CSV file for the future.

[https://miro.medium.com/v2/resize:fit:1400/0*fOjOS5briseP-tVx](https://miro.medium.com/v2/resize:fit:1400/0*fOjOS5briseP-tVx)

# **Step 2: We will install Terraform & AWS CLI to deploy our Jenkins Server(EC2) on AWS.**

Install & Configure Terraform and AWS CLI on your local machine to create Jenkins Server on AWS Cloud

**Terraform Installation Script**

```
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg - dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform -y
```

**AWSCLI Installation Script**

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```

Now, Configure both the tools

**Configure Terraform**

Edit the file /etc/environment using the below command add the highlighted lines and add your keys in the blur space.

```
sudo vim /etc/environment
```

[https://miro.medium.com/v2/resize:fit:1400/0*7lZLYeMwB5bz-Ppk](https://miro.medium.com/v2/resize:fit:1400/0*7lZLYeMwB5bz-Ppk)

After doing the changes, restart your machine to reflect the changes of your environment variables.

**Configure AWS CLI**

Run the below command, and add your keys

```
aws configure
```

[https://miro.medium.com/v2/resize:fit:1400/0*4dv8rDmjOpsrPTmM](https://miro.medium.com/v2/resize:fit:1400/0*4dv8rDmjOpsrPTmM)

# **Step 3: Deploy the Jenkins Server(EC2) using Terraform**

Clone the Git repository- [https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project](https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project)

Navigate to the **Jenkins-Server-TF**

Do some modifications to the backend.tf ****file such as changing the **bucket**name and **dynamodb** table(make sure you have created both manually on AWS Cloud).

[https://miro.medium.com/v2/resize:fit:1400/0*XS5aofaHWZSUPg8t](https://miro.medium.com/v2/resize:fit:1400/0*XS5aofaHWZSUPg8t)

Now, you have to replace the Pem File name as you have some other name for your Pem file. To provide the Pem file name that is already created on AWS

[https://miro.medium.com/v2/resize:fit:1400/0*b5iwwOZAwq25eGTr](https://miro.medium.com/v2/resize:fit:1400/0*b5iwwOZAwq25eGTr)

Initialize the backend by running the below command

```
terraform init
```

[https://miro.medium.com/v2/resize:fit:1400/0*1ujtQ2hefpfNP797](https://miro.medium.com/v2/resize:fit:1400/0*1ujtQ2hefpfNP797)

Run the below command to check the syntax error

```
terraform validate
```

[https://miro.medium.com/v2/resize:fit:1400/0*61UIT8Uwhm3ihzyh](https://miro.medium.com/v2/resize:fit:1400/0*61UIT8Uwhm3ihzyh)

Run the below command to get the blueprint of what kind of AWS services will be created.

```
terraform plan -var-file=variables.tfvars
```

[https://miro.medium.com/v2/resize:fit:1400/0*kCbNFg7KctoMk6tT](https://miro.medium.com/v2/resize:fit:1400/0*kCbNFg7KctoMk6tT)

Now, run the below command to create the infrastructure on AWS Cloud which will take 3 to 4 minutes maximum

```
terraform apply -var-file=variables.tfvars --auto-approve
```

[https://miro.medium.com/v2/resize:fit:1400/0*3as2JQMy9qDWwMzP](https://miro.medium.com/v2/resize:fit:1400/0*3as2JQMy9qDWwMzP)

Now, connect to your Jenkins-Server by clicking on Connect.

[https://miro.medium.com/v2/resize:fit:1400/0*ezFgBNWFgdB7RLKZ](https://miro.medium.com/v2/resize:fit:1400/0*ezFgBNWFgdB7RLKZ)

Copy the **ssh** command and paste it on your local machine.

[https://miro.medium.com/v2/resize:fit:1400/0*zmfCKcSKbshUWVxw](https://miro.medium.com/v2/resize:fit:1400/0*zmfCKcSKbshUWVxw)

# **Step 4: Configure the Jenkins**

Now, we logged into our **Jenkins server.**

[https://miro.medium.com/v2/resize:fit:1400/0*9M5OfMYa7lHX8Djo](https://miro.medium.com/v2/resize:fit:1400/0*9M5OfMYa7lHX8Djo)

We have installed some services such as Jenkins, Docker, Sonarqube, Terraform, Kubectl, AWS CLI, and Trivy.

Let’s validate whether all our installed or not.

```
jenkins --version
docker --version
docker ps
terraform --version
kubectl version
aws --version
trivy --version
eksctl --version
```

[https://miro.medium.com/v2/resize:fit:1400/0*wFais-upPs-M8Ojy](https://miro.medium.com/v2/resize:fit:1400/0*wFais-upPs-M8Ojy)

[https://miro.medium.com/v2/resize:fit:1400/0*gmVfH4Yn94mpQ9gL](https://miro.medium.com/v2/resize:fit:1400/0*gmVfH4Yn94mpQ9gL)

Now, we have to configure Jenkins. So, copy the public IP of your Jenkins Server and paste it on your favorite browser with an 8080 port.

[https://miro.medium.com/v2/resize:fit:1400/0*5TbuZn5IlK0B4DQi](https://miro.medium.com/v2/resize:fit:1400/0*5TbuZn5IlK0B4DQi)

Click on **Install suggested plugins**

[https://miro.medium.com/v2/resize:fit:1400/0*jf6h7sIpMTvoCIoh](https://miro.medium.com/v2/resize:fit:1400/0*jf6h7sIpMTvoCIoh)

The plugins will be installed

[https://miro.medium.com/v2/resize:fit:1400/0*8mHj10xce-8XBcGs](https://miro.medium.com/v2/resize:fit:1400/0*8mHj10xce-8XBcGs)

After installing the plugins, continue as admin

[https://miro.medium.com/v2/resize:fit:1400/0*xnzj3AqswIMou3Y1](https://miro.medium.com/v2/resize:fit:1400/0*xnzj3AqswIMou3Y1)

Click on **Save and Finish**

[https://miro.medium.com/v2/resize:fit:1400/0*S7lmx0AJUQWeoqRU](https://miro.medium.com/v2/resize:fit:1400/0*S7lmx0AJUQWeoqRU)

Click on **Start using Jenkins**

[https://miro.medium.com/v2/resize:fit:1400/0*JqsKBq71DGO-lKEV](https://miro.medium.com/v2/resize:fit:1400/0*JqsKBq71DGO-lKEV)

The Jenkins Dashboard will look like the below snippet

[https://miro.medium.com/v2/resize:fit:1400/0*Cmu3Xv8wYgwPmo1O](https://miro.medium.com/v2/resize:fit:1400/0*Cmu3Xv8wYgwPmo1O)

# **Step 5: We will deploy the EKS Cluster using eksctl commands**

Now, go back to your Jenkins Server **terminal** and configure the AWS.

[https://miro.medium.com/v2/resize:fit:1400/0*Fz7D7ZED7AS-f83m](https://miro.medium.com/v2/resize:fit:1400/0*Fz7D7ZED7AS-f83m)

Go to **Manage Jenkins**

Click on **Plugins**

[https://miro.medium.com/v2/resize:fit:1400/0*00Y0vchDDh-QOYoe](https://miro.medium.com/v2/resize:fit:1400/0*00Y0vchDDh-QOYoe)

Select the **Available plugins** install the following plugins and click on **Install**

*AWS Credentials*

*Pipeline: AWS Steps*

[https://miro.medium.com/v2/resize:fit:1400/0*QWxwPmI2YdIeNu-s](https://miro.medium.com/v2/resize:fit:1400/0*QWxwPmI2YdIeNu-s)

Once, both the plugins are installed, restart your Jenkins service by checking the **Restart Jenkins** option.

[https://miro.medium.com/v2/resize:fit:1400/0*fRBQ5VF78r04M8y-](https://miro.medium.com/v2/resize:fit:1400/0*fRBQ5VF78r04M8y-)

Login to your Jenkins Server Again

[https://miro.medium.com/v2/resize:fit:1400/0*tvpY1Iv0DbCu8xsh](https://miro.medium.com/v2/resize:fit:1400/0*tvpY1Iv0DbCu8xsh)

Now, we have to set our AWS credentials on Jenkins

Go to **Manage Plugins** and click on **Credentials**

[https://miro.medium.com/v2/resize:fit:1400/0*xSjY4Gp3hEv3xxIU](https://miro.medium.com/v2/resize:fit:1400/0*xSjY4Gp3hEv3xxIU)

Click on **global.**

[https://miro.medium.com/v2/resize:fit:1400/0*ykCdz9NneQsOFD3i](https://miro.medium.com/v2/resize:fit:1400/0*ykCdz9NneQsOFD3i)

Select **AWS Credentials** as **Kind** and add **the ID** same as shown in the below snippet except for your AWS Access Key & Secret Access key and click on **Create.**

[https://miro.medium.com/v2/resize:fit:1400/0*xmqbCBS8Jl0CY0Bx](https://miro.medium.com/v2/resize:fit:1400/0*xmqbCBS8Jl0CY0Bx)

The Credentials will look like the below snippet.

[https://miro.medium.com/v2/resize:fit:1400/0*VFmP6yGuiY0MvJuc](https://miro.medium.com/v2/resize:fit:1400/0*VFmP6yGuiY0MvJuc)

Now, We need to add GitHub credentials as well because currently, my repository is Private.

This thing, I am performing this because in Industry Projects your repository will be private.

So, add the username and personal access token of your GitHub account.

[https://miro.medium.com/v2/resize:fit:1400/0*EYXSJ5dvm8CjFPgx](https://miro.medium.com/v2/resize:fit:1400/0*EYXSJ5dvm8CjFPgx)

Both credentials will look like this.

[https://miro.medium.com/v2/resize:fit:1400/0*3wVRVgOUjVf6TxEX](https://miro.medium.com/v2/resize:fit:1400/0*3wVRVgOUjVf6TxEX)

Create an eks cluster using the below commands.

```
eksctl create cluster --name Three-Tier-K8s-EKS-Cluster --region us-east-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-east-1 --name Three-Tier-K8s-EKS-Cluster
```

[https://miro.medium.com/v2/resize:fit:1400/0*7ZM0msLdmJmEAigk](https://miro.medium.com/v2/resize:fit:1400/0*7ZM0msLdmJmEAigk)

Once your cluster is created, you can validate whether your nodes are ready or not by the below command

```
kubectl get nodes
```

[https://miro.medium.com/v2/resize:fit:1400/0*cwKpmSrZ6OWoTzX5](https://miro.medium.com/v2/resize:fit:1400/0*cwKpmSrZ6OWoTzX5)

# **Step 6: Now, we will configure the Load Balancer on our EKS because our application will have an ingress controller.**

Download the policy for the LoadBalancer prerequisite.

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

[https://miro.medium.com/v2/resize:fit:1400/0*8EdT_JP1gxIaJ36t](https://miro.medium.com/v2/resize:fit:1400/0*8EdT_JP1gxIaJ36t)

Create the IAM policy using the below command

```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```

[https://miro.medium.com/v2/resize:fit:1400/0*0qQ0iYpn3Sc3xXUU](https://miro.medium.com/v2/resize:fit:1400/0*0qQ0iYpn3Sc3xXUU)

Create OIDC Provider

```
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=Three-Tier-K8s-EKS-Cluster --approve
```

[https://miro.medium.com/v2/resize:fit:1400/0*jy6plZOd-Ctiw9Ej](https://miro.medium.com/v2/resize:fit:1400/0*jy6plZOd-Ctiw9Ej)

Create a Service Account by using below command and replace your account ID with your one

```
eksctl create iamserviceaccount --cluster=Three-Tier-K8s-EKS-Cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your_account_id>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1
```

[https://miro.medium.com/v2/resize:fit:1400/0*KM3uBMEXALirrzvP](https://miro.medium.com/v2/resize:fit:1400/0*KM3uBMEXALirrzvP)

Run the below command to deploy the AWS Load Balancer Controller

```
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```

After 2 minutes, run the command below to check whether your pods are running or not.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

[https://miro.medium.com/v2/resize:fit:1400/0*em5hdSHZAUEVj6dc](https://miro.medium.com/v2/resize:fit:1400/0*em5hdSHZAUEVj6dc)

# **Step 7: We need to create Amazon ECR Private Repositories for both Tiers (Frontend & Backend)**

Click on Create repository

[https://miro.medium.com/v2/resize:fit:1400/0*i2Y0E-wwltsEDbs_](https://miro.medium.com/v2/resize:fit:1400/0*i2Y0E-wwltsEDbs_)

Select the Private option to provide the repository and click on **Save**.

[https://miro.medium.com/v2/resize:fit:1400/0*3FoUALXSU9qjbsYC](https://miro.medium.com/v2/resize:fit:1400/0*3FoUALXSU9qjbsYC)

Do the same for the backend repository and click on **Save**

[https://miro.medium.com/v2/resize:fit:1400/0*LysvSFnlJrbiwfJR](https://miro.medium.com/v2/resize:fit:1400/0*LysvSFnlJrbiwfJR)

Now, we have set up our ECR Private Repository and

[https://miro.medium.com/v2/resize:fit:1400/0*kMUuFZTZ5eZPWi6M](https://miro.medium.com/v2/resize:fit:1400/0*kMUuFZTZ5eZPWi6M)

Now, we need to configure ECR locally because we have to upload our images to Amazon ECR.

Copy the **1st** command for login

[https://miro.medium.com/v2/resize:fit:1400/0*jZatwfI7hXZA8BoH](https://miro.medium.com/v2/resize:fit:1400/0*jZatwfI7hXZA8BoH)

Now, run the copied command on your **Jenkins Server.**

[https://miro.medium.com/v2/resize:fit:1400/0*qX8Drv-3ELdZBO-7](https://miro.medium.com/v2/resize:fit:1400/0*qX8Drv-3ELdZBO-7)

# **Step 8: Install & Configure ArgoCD**

We will be deploying our application on a three-tier namespace. To do that, we will create a three-tier namespace on EKS

```
kubectl create namespace three-tier
```

[https://miro.medium.com/v2/resize:fit:1400/0*qoLwA0HhXCQdcBQS](https://miro.medium.com/v2/resize:fit:1400/0*qoLwA0HhXCQdcBQS)

As you know, Our two ECR repositories are private. So, when we try to push images to the ECR Repos it will give us the error **Imagepullerror.**

To get rid of this error, we will create a secret for our ECR Repo by the below command and then, we will add this secret to the deployment file.

**Note**: The Secrets are coming from the .docker/config.json file which is created while login the ECR in the earlier steps

```
kubectl create secret generic ecr-registry-secret \
  --from-file=.dockerconfigjson=${HOME}/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson --namespace three-tier
kubectl get secrets -n three-tier
```

[https://miro.medium.com/v2/resize:fit:1400/0*eSPMcpCejpppnfML](https://miro.medium.com/v2/resize:fit:1400/0*eSPMcpCejpppnfML)

Now, we will install argoCD.

To do that, create a separate namespace for it and apply the argocd configuration for installation.

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
```

[https://miro.medium.com/v2/resize:fit:1400/0*TdWY04ZXZIek5OuW](https://miro.medium.com/v2/resize:fit:1400/0*TdWY04ZXZIek5OuW)

All pods must be running, to validate run the below command

```
kubectl get pods -n argocd
```

[https://miro.medium.com/v2/resize:fit:1400/0*p64YtND22T0fs5du](https://miro.medium.com/v2/resize:fit:1400/0*p64YtND22T0fs5du)

Now, expose the argoCD server as LoadBalancer using the below command

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

[https://miro.medium.com/v2/resize:fit:1400/0*J2rpVY56Z_nPM3mk](https://miro.medium.com/v2/resize:fit:1400/0*J2rpVY56Z_nPM3mk)

You can validate whether the Load Balancer is created or not by going to the AWS Console

[https://miro.medium.com/v2/resize:fit:1400/0*0i1RWu1jP9iNQEY5](https://miro.medium.com/v2/resize:fit:1400/0*0i1RWu1jP9iNQEY5)

To access the argoCD, copy the LoadBalancer DNS and hit on your favorite browser.

You will get a warning like the below snippet.

Click on **Advanced.**

[https://miro.medium.com/v2/resize:fit:1400/0*XFz4syg6yPbraayd](https://miro.medium.com/v2/resize:fit:1400/0*XFz4syg6yPbraayd)

Click on the below link which is appearing under **Hide advanced**

[https://miro.medium.com/v2/resize:fit:1400/0*QH2XTU2oCp81Ir7o](https://miro.medium.com/v2/resize:fit:1400/0*QH2XTU2oCp81Ir7o)

Now, we need to get the password for our argoCD server to perform the deployment.

To do that, we have a pre-requisite which is **jq.** Install it by the command below.

```
sudo apt install jq -y
```

[https://miro.medium.com/v2/resize:fit:1400/0*RV9UVtziS2BKdzu9](https://miro.medium.com/v2/resize:fit:1400/0*RV9UVtziS2BKdzu9)

```
export ARGOCD_SERVER='kubectl get svc argocd-server -n argocd -o json | jq - raw-output '.status.loadBalancer.ingress[0].hostname''
export ARGO_PWD='kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d'
echo $ARGO_PWD
```

[https://miro.medium.com/v2/resize:fit:1400/0*tbPrTdjVhQunGlsr](https://miro.medium.com/v2/resize:fit:1400/0*tbPrTdjVhQunGlsr)

Enter the username and password in argoCD and click on **SIGN IN**.

[https://miro.medium.com/v2/resize:fit:1400/0*Cih_eJ5o_4qpyUNj](https://miro.medium.com/v2/resize:fit:1400/0*Cih_eJ5o_4qpyUNj)

Here is our ArgoCD **Dashboard**.

[https://miro.medium.com/v2/resize:fit:1400/0*dAa4bJlR_pWCCtmP](https://miro.medium.com/v2/resize:fit:1400/0*dAa4bJlR_pWCCtmP)

# **Step 9: Now, we have to configure Sonarqube for our DevSecOps Pipeline**

To do that, copy your Jenkins Server public IP and paste it on your favorite browser with a 9000 port

The username and password will be **admin**

Click on **Log In.**

[https://miro.medium.com/v2/resize:fit:1400/0*VLQi4ARw2u2sAyMp](https://miro.medium.com/v2/resize:fit:1400/0*VLQi4ARw2u2sAyMp)

Update the password

[https://miro.medium.com/v2/resize:fit:1400/0*Y_Sk-S4usHdFuwCJ](https://miro.medium.com/v2/resize:fit:1400/0*Y_Sk-S4usHdFuwCJ)

Click on **Administration** then **Security**, and select **Users**

[https://miro.medium.com/v2/resize:fit:1400/0*BJBUJhn7kdWLfINO](https://miro.medium.com/v2/resize:fit:1400/0*BJBUJhn7kdWLfINO)

Click on **Update tokens**

[https://miro.medium.com/v2/resize:fit:1400/0*rNJI6nC-Bl4D_ZT6](https://miro.medium.com/v2/resize:fit:1400/0*rNJI6nC-Bl4D_ZT6)

Click on **Generate**

[https://miro.medium.com/v2/resize:fit:1400/0*qT2RYD0tJgI5MU9X](https://miro.medium.com/v2/resize:fit:1400/0*qT2RYD0tJgI5MU9X)

Copy the **token** keep it somewhere safe and click on **Done**.

[https://miro.medium.com/v2/resize:fit:1400/0*bt8XAF4MROuWki_u](https://miro.medium.com/v2/resize:fit:1400/0*bt8XAF4MROuWki_u)

Now, we have to configure **webhooks** for quality checks.

Click on **Administration** then, Configuration and select **Webhooks**

[https://miro.medium.com/v2/resize:fit:1400/0*bQ9cOkumcl4ZDh5L](https://miro.medium.com/v2/resize:fit:1400/0*bQ9cOkumcl4ZDh5L)

Click on **Create**

[https://miro.medium.com/v2/resize:fit:1400/0*qNxFFvdZLeKqGws6](https://miro.medium.com/v2/resize:fit:1400/0*qNxFFvdZLeKqGws6)

Provide the name of your project and in the URL, provide the Jenkins server public IP with port 8080 add sonarqube-webhook in the suffix, and click on Create.

http://<jenkins-server-public-ip>:8080/sonarqube-webhook/

[https://miro.medium.com/v2/resize:fit:1400/0*mwNvzfs2bgMS-eRG](https://miro.medium.com/v2/resize:fit:1400/0*mwNvzfs2bgMS-eRG)

Here, you can see the **webhook**.

[https://miro.medium.com/v2/resize:fit:1400/0*-KfhYHTFcevfWsFx](https://miro.medium.com/v2/resize:fit:1400/0*-KfhYHTFcevfWsFx)

Now, we have to create a Project for frontend code**.**

Click on **Manually.**

[https://miro.medium.com/v2/resize:fit:1400/0*cYQHP_vVN9KuSmPq](https://miro.medium.com/v2/resize:fit:1400/0*cYQHP_vVN9KuSmPq)

Provide the display name to your **Project** and click on **Setup**

[https://miro.medium.com/v2/resize:fit:1400/0*6TmbHaWHpD5kKUN2](https://miro.medium.com/v2/resize:fit:1400/0*6TmbHaWHpD5kKUN2)

Click on **Locally.**

[https://miro.medium.com/v2/resize:fit:1400/0*u4i-WFHilywDXusS](https://miro.medium.com/v2/resize:fit:1400/0*u4i-WFHilywDXusS)

Select the **Use existing token** and click on **Continue.**

[https://miro.medium.com/v2/resize:fit:1400/0*07c-DyJf_aBZMrfA](https://miro.medium.com/v2/resize:fit:1400/0*07c-DyJf_aBZMrfA)

Select **Other** and **Linux** as OS.

After performing the above steps, you will get the command which you can see in the below snippet.

Now, use the command in the Jenkins Frontend Pipeline where Code Quality Analysis will be performed.

[https://miro.medium.com/v2/resize:fit:1400/0*J4cL6zQNpLpO4xpy](https://miro.medium.com/v2/resize:fit:1400/0*J4cL6zQNpLpO4xpy)

Now, we have to create a Project for backend code**.**

Click on **Create Project.**

[https://miro.medium.com/v2/resize:fit:1400/0*90X-HvJMVGufXUws](https://miro.medium.com/v2/resize:fit:1400/0*90X-HvJMVGufXUws)

Provide the name of your project name and click on **Set up.**

[https://miro.medium.com/v2/resize:fit:1400/0*678-zmRHMUwibqGS](https://miro.medium.com/v2/resize:fit:1400/0*678-zmRHMUwibqGS)

Click on **Locally.**

[https://miro.medium.com/v2/resize:fit:1400/0*2Ym3JEFOAiiOrjBT](https://miro.medium.com/v2/resize:fit:1400/0*2Ym3JEFOAiiOrjBT)

Select the **Use existing token** and click on **Continue.**

[https://miro.medium.com/v2/resize:fit:1400/0*dtXMJxBjpmWRE_J1](https://miro.medium.com/v2/resize:fit:1400/0*dtXMJxBjpmWRE_J1)

Select **Other** and **Linux** as OS.

After performing the above steps, you will get the command which you can see in the below snippet.

Now, use the command in the Jenkins Backend Pipeline where Code Quality Analysis will be performed.

[https://miro.medium.com/v2/resize:fit:1400/0*7ncu2YvpCOrSsmZS](https://miro.medium.com/v2/resize:fit:1400/0*7ncu2YvpCOrSsmZS)

Now, we have to store the sonar credentials.

Go to **Dashboard -> Manage Jenkins -> Credentials**

Select the kind as **Secret text** paste your token in **Secret** and keep other things as it is.

Click on **Create**

[https://miro.medium.com/v2/resize:fit:1400/0*16qKLACxpe53NHhX](https://miro.medium.com/v2/resize:fit:1400/0*16qKLACxpe53NHhX)

Now, we have to store the GitHub Personal access token to push the deployment file which will be modified in the pipeline itself for the ECR image.

**Add GitHub credentials**

Select the kind as **Secret text** and paste your GitHub Personal access token(not password) in Secret and keep other things as it is.

Click on **Create**

**Note**: If you haven’t generated your token then, you have it generated first then paste it into the Jenkins

[https://miro.medium.com/v2/resize:fit:1400/0*PmFud39CH5O2peL0](https://miro.medium.com/v2/resize:fit:1400/0*PmFud39CH5O2peL0)

Now, according to our Pipeline, we need to add an Account ID in the Jenkins credentials because of the ECR repo URI.

Select the kind as **Secret text** paste your AWS Account ID in Secret and keep other things as it is.

Click on **Create**

[https://miro.medium.com/v2/resize:fit:1400/0*TI-h4azZp1BxYDPT](https://miro.medium.com/v2/resize:fit:1400/0*TI-h4azZp1BxYDPT)

Now, we need to provide our ECR image name for frontend which is **frontend** only.

Select the kind as **Secret text** paste your frontend repo name in Secret and keep other things as it is.

Click on **Create**

[https://miro.medium.com/v2/resize:fit:1400/0*xQ3RN8HFk6LRQ3xP](https://miro.medium.com/v2/resize:fit:1400/0*xQ3RN8HFk6LRQ3xP)

Now, we need to provide our ECR image name for the backend which is **backend** only.

Select the kind as **Secret text,** paste your backend repo name in Secret, and keep other things as it is.

Click on **Create**

[https://miro.medium.com/v2/resize:fit:1400/0*Od9oUAzatYi7pf2I](https://miro.medium.com/v2/resize:fit:1400/0*Od9oUAzatYi7pf2I)

Final Snippet of all Credentials that we needed to implement this project.

[https://miro.medium.com/v2/resize:fit:1400/0*NknLh2XMt1yoVSQW](https://miro.medium.com/v2/resize:fit:1400/0*NknLh2XMt1yoVSQW)

# **Step 10: Install the required plugins and configure the plugins to deploy our Three-Tier Application**

Install the following plugins by going to **Dashboard -> Manage Jenkins -> Plugins -> Available Plugins**

```
Docker
Docker Commons
Docker Pipeline
Docker API
docker-build-step
Eclipse Temurin installer
NodeJS
OWASP Dependency-Check
SonarQube Scanner
```

[https://miro.medium.com/v2/resize:fit:1400/0*9Sv389_NNDnMjT7E](https://miro.medium.com/v2/resize:fit:1400/0*9Sv389_NNDnMjT7E)

Now, we have to configure the installed plugins.

Go to **Dashboard -> Manage Jenkins -> Tools**

We are configuring jdk

Search for **jdk** and provide the configuration like the below snippet.

[https://miro.medium.com/v2/resize:fit:1400/0*7RsMr58tNsVzJG9o](https://miro.medium.com/v2/resize:fit:1400/0*7RsMr58tNsVzJG9o)

Now, we will configure the sonarqube-scanner

Search for the sonarqube scanner and provide the configuration like the below snippet.

[https://miro.medium.com/v2/resize:fit:1400/0*_mU6-Vd06Lz83g_b](https://miro.medium.com/v2/resize:fit:1400/0*_mU6-Vd06Lz83g_b)

Now, we will configure **nodejs**

Search for **node** and provide the configuration like the below snippet.

[https://miro.medium.com/v2/resize:fit:1400/0*b2z_rnTavmEfQul5](https://miro.medium.com/v2/resize:fit:1400/0*b2z_rnTavmEfQul5)

Now, we will configure the OWASP Dependency check

Search for **Dependency-Check** and provide the configuration like the below snippet.

[https://miro.medium.com/v2/resize:fit:1400/0*MZoquLi-h9BopM-f](https://miro.medium.com/v2/resize:fit:1400/0*MZoquLi-h9BopM-f)

Now, we will configure the docker

Search for **docker** and provide the configuration like the below snippet.

[https://miro.medium.com/v2/resize:fit:1400/0*-UKXqzEodSOl0KfC](https://miro.medium.com/v2/resize:fit:1400/0*-UKXqzEodSOl0KfC)

Now, we have to set the path for **Sonarqube** in **Jenkins**

Go to **Dashboard -> Manage Jenkins -> System**

Search for **SonarQube** **installations**

Provide the name as it is, then in the Server URL copy the sonarqube public IP (same as Jenkins) with port 9000 select the sonar token that we have added recently, and click on Apply & Save.

[https://miro.medium.com/v2/resize:fit:1400/0*xVxL6x-iO_2E6PJY](https://miro.medium.com/v2/resize:fit:1400/0*xVxL6x-iO_2E6PJY)

Now, we are ready to create our Jenkins Pipeline to deploy our Backend Code.

Go to Jenkins **Dashboard**

Click on **New** **Item**

[https://miro.medium.com/v2/resize:fit:1400/0*ZBFtbNPtlyCanrmr](https://miro.medium.com/v2/resize:fit:1400/0*ZBFtbNPtlyCanrmr)

Provide the name of your **Pipeline** and click on **OK**.

[https://miro.medium.com/v2/resize:fit:1400/0*livXCTbX_MR1yIbq](https://miro.medium.com/v2/resize:fit:1400/0*livXCTbX_MR1yIbq)

This is the Jenkins file to deploy the Backend Code on **EKS**.

Copy and paste it into the **Jenkins**

[https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-Backend](https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-Backend)

Click **Apply** & **Save**.

[https://miro.medium.com/v2/resize:fit:1400/0*NuBmt1dUWJsObdZj](https://miro.medium.com/v2/resize:fit:1400/0*NuBmt1dUWJsObdZj)

Now, click on the **build**.

Our **pipeline** was **successful** after a few common mistakes.

**Note**: Do the changes in the Pipeline according to your project.

[https://miro.medium.com/v2/resize:fit:1400/0*asHSELq69aMhiS2c](https://miro.medium.com/v2/resize:fit:1400/0*asHSELq69aMhiS2c)

Now, we are ready to create our Jenkins Pipeline to deploy our Frontend Code.

Go to Jenkins **Dashboard**

Click on **New** **Item**

Provide the name of your **Pipeline** and click on **OK**.

[https://miro.medium.com/v2/resize:fit:1400/0*-e4NX_4iBeF-qyYf](https://miro.medium.com/v2/resize:fit:1400/0*-e4NX_4iBeF-qyYf)

This is the Jenkins file to deploy the Frontend Code on **EKS**.

Copy and paste it into the **Jenkins**

[https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-Frontend](https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-Frontend)

Click **Apply** & **Save**.

[https://miro.medium.com/v2/resize:fit:1400/0*-wGcfCCl4ZGH8Ghn](https://miro.medium.com/v2/resize:fit:1400/0*-wGcfCCl4ZGH8Ghn)

Now, click on the **build**.

Our **pipeline** was **successful** after a few common mistakes.

**Note**: Do the changes in the Pipeline according to your project.

[https://miro.medium.com/v2/resize:fit:1400/0*5zV2Mm-0Dy-3tv01](https://miro.medium.com/v2/resize:fit:1400/0*5zV2Mm-0Dy-3tv01)

**Setup 10: We will set up the Monitoring for our EKS Cluster. We can monitor the Cluster Specifications and other necessary things.**

We will achieve the monitoring using Helm

Add the prometheus repo by using the below command

```
helm repo add stable https://charts.helm.sh/stable
```

[https://miro.medium.com/v2/resize:fit:1400/0*Wi2E9w62_h7LtRhc](https://miro.medium.com/v2/resize:fit:1400/0*Wi2E9w62_h7LtRhc)

Install the prometheus

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
```

[https://miro.medium.com/v2/resize:fit:1400/0*nIWvFquxA6YfMJW7](https://miro.medium.com/v2/resize:fit:1400/0*nIWvFquxA6YfMJW7)

Now, check the service by the below command

```
kubectl get svc
```

[https://miro.medium.com/v2/resize:fit:1400/0*oqhY3keWKOLDwwwM](https://miro.medium.com/v2/resize:fit:1400/0*oqhY3keWKOLDwwwM)

Now, we need to access our Prometheus and Grafana consoles from outside of the cluster.

For that, we need to change the Service type from ClusterType to **LoadBalancer**

Edit the **stable-kube-prometheus-sta-prometheus** service

```
kubectl edit svc stable-kube-prometheus-sta-prometheus
```

[https://miro.medium.com/v2/resize:fit:1400/0*NJmMiX7f8AhdU2Eo](https://miro.medium.com/v2/resize:fit:1400/0*NJmMiX7f8AhdU2Eo)

Modification in the 48th line from ClusterType to **LoadBalancer**

[https://miro.medium.com/v2/resize:fit:1400/0*n6WrahD4FpcKqUnS](https://miro.medium.com/v2/resize:fit:1400/0*n6WrahD4FpcKqUnS)

Edit the **stable-grafana** service

```
kubectl edit svc stable-grafana
```

[https://miro.medium.com/v2/resize:fit:1400/0*AlPEotjwnO22rAI7](https://miro.medium.com/v2/resize:fit:1400/0*AlPEotjwnO22rAI7)

Modification in the 39th line from ClusterType to **LoadBalancer**

[https://miro.medium.com/v2/resize:fit:1400/0*DJHsWa0GAKt17iyM](https://miro.medium.com/v2/resize:fit:1400/0*DJHsWa0GAKt17iyM)

Now, if you list again the service then, you will see the LoadBalancers DNS names

```
kubectl get svc
```

[https://miro.medium.com/v2/resize:fit:1400/0*8FQJWV5UGE-SfOWs](https://miro.medium.com/v2/resize:fit:1400/0*8FQJWV5UGE-SfOWs)

You can also validate from your console.

[https://miro.medium.com/v2/resize:fit:1400/0*n__Cg9-txhe3vo1b](https://miro.medium.com/v2/resize:fit:1400/0*n__Cg9-txhe3vo1b)

Now, access your Prometheus Dashboard

Paste the <Prometheus-LB-DNS>:9090 in your favorite browser and you will see like this

[https://miro.medium.com/v2/resize:fit:1400/0*65gDRAqoz2VsgvhB](https://miro.medium.com/v2/resize:fit:1400/0*65gDRAqoz2VsgvhB)

Click on **Status** and select **Target.**

You will see a lot of Targets

[https://miro.medium.com/v2/resize:fit:1400/0*wNgdePSYlFcbsU1J](https://miro.medium.com/v2/resize:fit:1400/0*wNgdePSYlFcbsU1J)

Now, access your **Grafana** **Dashboard**

Copy the ALB DNS of Grafana and paste it into your favorite browser.

The username will be **admin** and the password will be **prom-operator** for your Grafana LogIn.

[https://miro.medium.com/v2/resize:fit:1400/0*v-aKBEgp1HjwZbpK](https://miro.medium.com/v2/resize:fit:1400/0*v-aKBEgp1HjwZbpK)

Now, click on **Data Source**

[https://miro.medium.com/v2/resize:fit:1400/0*w5sjCh7_X8WNm5hd](https://miro.medium.com/v2/resize:fit:1400/0*w5sjCh7_X8WNm5hd)

Select **Prometheus**

[https://miro.medium.com/v2/resize:fit:1400/0*R7E0N_Fbf5y2We20](https://miro.medium.com/v2/resize:fit:1400/0*R7E0N_Fbf5y2We20)

In the **Connection,** paste your <Prometheus-LB-DNS>:9090.

[https://miro.medium.com/v2/resize:fit:1400/0*Z2MqEMYQyIlIRHpj](https://miro.medium.com/v2/resize:fit:1400/0*Z2MqEMYQyIlIRHpj)

If the URL is correct, then you will see a green notification/

Click on **Save** & **test**.

[https://miro.medium.com/v2/resize:fit:1400/0*zd4mMa4NkIQPGW0f](https://miro.medium.com/v2/resize:fit:1400/0*zd4mMa4NkIQPGW0f)

Now, we will create a dashboard to visualize our Kubernetes Cluster Logs.

Click on **Dashboard.**

[https://miro.medium.com/v2/resize:fit:1400/0*Dd-95JdO7f7xrg3i](https://miro.medium.com/v2/resize:fit:1400/0*Dd-95JdO7f7xrg3i)

Once you click on **Dashboard.** You will see a lot of Kubernetes components monitoring.

[https://miro.medium.com/v2/resize:fit:1400/0*xjfOA2P0_v8tJps1](https://miro.medium.com/v2/resize:fit:1400/0*xjfOA2P0_v8tJps1)

Let’s try to import a type of Kubernetes Dashboard.

Click on **New** and select **Import**

[https://miro.medium.com/v2/resize:fit:1400/0*fT_tSwtXLohNWxuG](https://miro.medium.com/v2/resize:fit:1400/0*fT_tSwtXLohNWxuG)

Provide **6417** ID ****and click on **Load**

**Note:** 6417 is a unique ID from Grafana which is used to Monitor and visualize Kubernetes Data

[https://miro.medium.com/v2/resize:fit:1400/0*0jCNbY5ZiIOxkoAb](https://miro.medium.com/v2/resize:fit:1400/0*0jCNbY5ZiIOxkoAb)

Select the **data source** that you have created earlier and click on **Import.**

[https://miro.medium.com/v2/resize:fit:1400/0*3ZLxhJR-IzihlRG5](https://miro.medium.com/v2/resize:fit:1400/0*3ZLxhJR-IzihlRG5)

Here, you go.

You can view your Kubernetes Cluster Data.

Feel free to explore the other details of the Kubernetes Cluster.

[https://miro.medium.com/v2/resize:fit:1400/0*QtmraIKAuiOOKysG](https://miro.medium.com/v2/resize:fit:1400/0*QtmraIKAuiOOKysG)

# **Step 11: We will deploy our Three-Tier Application using ArgoCD.**

As our repository is private. So, we need to configure the Private Repository in ArgoCD.

Click on **Settings** and select **Repositories**

[https://miro.medium.com/v2/resize:fit:1400/0*iiANkZQawTciUtjd](https://miro.medium.com/v2/resize:fit:1400/0*iiANkZQawTciUtjd)

Click on **CONNECT REPO USING HTTPS**

[https://miro.medium.com/v2/resize:fit:1400/0*ZZIbX0Qa28KmApre](https://miro.medium.com/v2/resize:fit:1400/0*ZZIbX0Qa28KmApre)

Now, provide the repository name where your Manifests files are present.

Provide the username and GitHub Personal Access token and click on **CONNECT.**

[https://miro.medium.com/v2/resize:fit:1400/0*7gWpaGl4_7EEUNk8](https://miro.medium.com/v2/resize:fit:1400/0*7gWpaGl4_7EEUNk8)

If your **Connection Status** is **Successful** it means repository connected successfully.

[https://miro.medium.com/v2/resize:fit:1400/0*L_Q9D3UfiOadfRLF](https://miro.medium.com/v2/resize:fit:1400/0*L_Q9D3UfiOadfRLF)

Now, we will create our first application which will be a database.

Click on **CREATE APPLICATION.**

[https://miro.medium.com/v2/resize:fit:1400/0*NRGUVQ8_9Xvg7kCd](https://miro.medium.com/v2/resize:fit:1400/0*NRGUVQ8_9Xvg7kCd)

Provide the details as it is provided in the below snippet and scroll down.

[https://miro.medium.com/v2/resize:fit:1400/0*7WABYpxZc6zQv9mz](https://miro.medium.com/v2/resize:fit:1400/0*7WABYpxZc6zQv9mz)

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where your Manifest files are presented and provide other things as shown in the below screenshot.

Click on **CREATE.**

[https://miro.medium.com/v2/resize:fit:1400/0*XFiSCJ6CleHG08sW](https://miro.medium.com/v2/resize:fit:1400/0*XFiSCJ6CleHG08sW)

While your database Application is starting to deploy, We will create an application for the backend.

Provide the details as it is provided in the below snippet and scroll down.

[https://miro.medium.com/v2/resize:fit:1400/0*hx6_bVPFk8nO5GNr](https://miro.medium.com/v2/resize:fit:1400/0*hx6_bVPFk8nO5GNr)

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where your Manifest files are presented and provide other things as shown in the below screenshot.

Click on **CREATE.**

[https://miro.medium.com/v2/resize:fit:1400/0*IwPPAUYu4RHgMVj5](https://miro.medium.com/v2/resize:fit:1400/0*IwPPAUYu4RHgMVj5)

While your backend Application is starting to deploy, We will create an application for the frontend.

Provide the details as it is provided in the below snippet and scroll down.

[https://miro.medium.com/v2/resize:fit:1400/0*nDCubL8R75Tm0X-5](https://miro.medium.com/v2/resize:fit:1400/0*nDCubL8R75Tm0X-5)

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where your Manifest files are presented and provide other things as shown in the below screenshot.

Click on **CREATE.**

[https://miro.medium.com/v2/resize:fit:1400/0*K_dEWYIWTTH16s-z](https://miro.medium.com/v2/resize:fit:1400/0*K_dEWYIWTTH16s-z)

While your frontend Application is starting to deploy, We will create an application for the ingress.

Provide the details as it is provided in the below snippet and scroll down.

[https://miro.medium.com/v2/resize:fit:1400/0*K_5W2KRVn8G1x1-w](https://miro.medium.com/v2/resize:fit:1400/0*K_5W2KRVn8G1x1-w)

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where your Manifest files are presented and provide other things as shown in the below screenshot.

Click on **CREATE.**

[https://miro.medium.com/v2/resize:fit:1400/0*ypj4qy9_vXG5EPPy](https://miro.medium.com/v2/resize:fit:1400/0*ypj4qy9_vXG5EPPy)

Once your Ingress application is deployed. It will create an **Application Load Balancer**

You can check out the load balancer named with k8s-three.

[https://miro.medium.com/v2/resize:fit:1400/0*3AmniUgnLLoFwbx_](https://miro.medium.com/v2/resize:fit:1400/0*3AmniUgnLLoFwbx_)

Now, Copy the ALB-DNS and go to your Domain Provider in my case porkbun is the domain provider.

Go to **DNS** and add a **CNAME** type with hostname **backend** then add your **ALB** in the Answer and click on **Save**

**Note:** I have created a subdomain backend.amanpathakdevops.study

[https://miro.medium.com/v2/resize:fit:1400/0*nEYG4ebWYgKt3ZG1](https://miro.medium.com/v2/resize:fit:1400/0*nEYG4ebWYgKt3ZG1)

You can see all 4 application deployments in the below snippet.

[https://miro.medium.com/v2/resize:fit:1400/0*u7B1vbstPC6fSj-k](https://miro.medium.com/v2/resize:fit:1400/0*u7B1vbstPC6fSj-k)

Now, hit your subdomain after 2 to 3 minutes in your browser to see the magic.

[https://miro.medium.com/v2/resize:fit:1400/0*irM1WFSzXrw3cdOn](https://miro.medium.com/v2/resize:fit:1400/0*irM1WFSzXrw3cdOn)

You can play with the application by adding the records.

[https://miro.medium.com/v2/resize:fit:1400/0*67DLFdI0JeJHdbaj](https://miro.medium.com/v2/resize:fit:1400/0*67DLFdI0JeJHdbaj)

You can play with the application by deleting the records.

[https://miro.medium.com/v2/resize:fit:1400/0*yGSKeAappDnVERgs](https://miro.medium.com/v2/resize:fit:1400/0*yGSKeAappDnVERgs)

Now, you can see your Grafana Dashboard to view the EKS data such as pods, namespace, deployments, etc.

[https://miro.medium.com/v2/resize:fit:1400/0*AVM2g9q70eVWsFtC](https://miro.medium.com/v2/resize:fit:1400/0*AVM2g9q70eVWsFtC)

If you want to monitor the three-tier namespace.

In the namespace, replace three-tier with another namespace.

You will see the deployments that are done by ArgoCD

[https://miro.medium.com/v2/resize:fit:1400/0*nbeuEQeqsuySpvf-](https://miro.medium.com/v2/resize:fit:1400/0*nbeuEQeqsuySpvf-)

This is the **Ingress** Application Deployment in ArgoCD

[https://miro.medium.com/v2/resize:fit:1400/0*lG4qo0WD3O4AO9EM](https://miro.medium.com/v2/resize:fit:1400/0*lG4qo0WD3O4AO9EM)

This is the **Frontend** Application Deployment in ArgoCD

[https://miro.medium.com/v2/resize:fit:1400/0*iGaEmdBYefGerqBM](https://miro.medium.com/v2/resize:fit:1400/0*iGaEmdBYefGerqBM)

This is the **Backend** Application Deployment in ArgoCD

[https://miro.medium.com/v2/resize:fit:1400/0*pu6IPEsSw9B04cOQ](https://miro.medium.com/v2/resize:fit:1400/0*pu6IPEsSw9B04cOQ)

This is the **Database** Application Deployment in ArgoCD

[https://miro.medium.com/v2/resize:fit:1400/0*i58DUi_TH8cZ8143](https://miro.medium.com/v2/resize:fit:1400/0*i58DUi_TH8cZ8143)

If you observe, we have configured the Persistent Volume & Persistent Volume Claim. So, if the pods get deleted then, the data won’t be lost. The Data will be stored on the host machine.

To validate it, delete both Database pods.

[https://miro.medium.com/v2/resize:fit:1400/0*d-0PnShonxFIAkX3](https://miro.medium.com/v2/resize:fit:1400/0*d-0PnShonxFIAkX3)

Now, the new pods will be started.

[https://miro.medium.com/v2/resize:fit:1400/0*8HtOkQqZHYptwp4C](https://miro.medium.com/v2/resize:fit:1400/0*8HtOkQqZHYptwp4C)

And Your Application won’t lose a single piece of data.

[https://miro.medium.com/v2/resize:fit:1400/0*SlqBu_yX0o-z08kZ](https://miro.medium.com/v2/resize:fit:1400/0*SlqBu_yX0o-z08kZ)

# **Conclusion:**

In this comprehensive DevSecOps Kubernetes project, we successfully:

- Established IAM user and Terraform for AWS setup.
- Deployed Jenkins on AWS, configured tools, and integrated it with Sonarqube.
- Set up an EKS cluster, configured a Load Balancer, and established private ECR repositories.
- Implemented monitoring with Helm, Prometheus, and Grafana.
- Installed and configured ArgoCD for GitOps practices.
- Created Jenkins pipelines for CI/CD, deploying a Three-Tier application.
- Ensured data persistence with persistent volumes and claims.

**Support my work-**

https://www.buymeacoffee.com/aman.pathak

Stay connected on **LinkedIn**: [LinkedIn Profile](https://www.linkedin.com/in/aman-devops/)

Stay up-to-date with **GitHub**: [GitHub Profile](https://github.com/AmanPathak-DevOps)

Want to discuss trending technologies in DevOps & Cloud

Join the **Discord Server**- [https://discord.gg/jdzF8kTtw2](https://discord.gg/jdzF8kTtw2)

Feel free to reach out to me, if you have any other queries.

Happy Learning!