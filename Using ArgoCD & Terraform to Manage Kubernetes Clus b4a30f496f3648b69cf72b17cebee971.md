# Using ArgoCD & Terraform to Manage Kubernetes Cluster

Seamless deployment and management of your infrastructure and application is key for a successful organization. This is where ArgoCD and Terraform can help.

Terraform is an infrastructure-as-code (IaC) tool that allows you to provision infrastructure and works with many cloud providers, databases, VCS systems, and Kubernetes. ArgoCD is a GitOps delivery tool for Kubernetes that uses git as its only source of truth for deploying and managing your Kubernetes workloads.

# **Why use ArgoCD with Terraform?**

Using [ArgoCD](https://spacelift.io/blog/argocd) with [Terraform](https://spacelift.io/blog/what-is-terraform) combines infrastructure deployment with application deployment. By using them together, you can ensure a seamless workflow process.

Here are some key advantages of combining them:

- **Declarative IaC and application management —** Both tools have a declarative approach when it comes to managing the infrastructure and application, ensuring the current state matches the desired state.
- **GitOps workflow —** You can achieve a GitOps workflow for both of them — out of the box for ArgoCD and with a specialized product for Terraform. In GitOps, git is used as the only source of truth, and you can easily handle your workflow and ensure everything is working smoothly.
- **Consistency** — By leveraging ArgoCD and Terraform, you can ensure the same configuration is deployed on different environments, reducing the chances for errors and discrepancies.

# **Tutorial: How to manage Kubernetes cluster with ArgoCD and Terraform**

Let’s build an automation that showcases how to manage a K8s cluster with ArgoCD and Terraform.

## **Prerequisites**

For this automation, you need to have an AWS account, everything else will be built and shared during the tutorial. You can get the repository code [here](https://github.com/saturnhead/eks-argo-terraform).

## **Step 1 — Prepare Terraform code for EKS**

To ensure everything works smoothly, we will create all the components necessary to have an EKS cluster running:

- Network (VPC, Subnets, Route table, Internet Gateway)
- IAM (Role and Policies)
- Node Group
- EKS cluster

Network resources:

```
data "aws_availability_zones" "available" {}

resource "aws_vpc" "main" {
 cidr_block = "10.0.0.0/16"

 tags = {
   Name = "main-vpc"
 }
}

resource "aws_subnet" "public_subnet" {
 count                   = 2
 vpc_id                  = aws_vpc.main.id
 cidr_block              = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
 availability_zone       = data.aws_availability_zones.available.names[count.index]
 map_public_ip_on_launch = true

 tags = {
   Name = "public-subnet-${count.index}"
 }
}

resource "aws_internet_gateway" "main" {
 vpc_id = aws_vpc.main.id

 tags = {
   Name = "main-igw"
 }
}

resource "aws_route_table" "public" {
 vpc_id = aws_vpc.main.id

 route {
   cidr_block = "0.0.0.0/0"
   gateway_id = aws_internet_gateway.main.id
 }

 tags = {
   Name = "main-route-table"
 }
}

resource "aws_route_table_association" "a" {
 count          = 2
 subnet_id      = aws_subnet.public_subnet.*.id[count.index]
 route_table_id = aws_route_table.public.id
}
```

We are creating one VPC, two subnets for the EKS nodes, an internet gateway, and a route table with a rule to the internet gateway. We associate the route table with the two subnets.

```
locals {
 policies = ["arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy", "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy", "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"]
}

resource "aws_iam_role" "eks_cluster_role" {
 name = "eks-role"

 assume_role_policy = <<EOF
{
 "Version": "2012-10-17",
 "Statement": [
   {
     "Action": "sts:AssumeRole",
     "Principal": {
       "Service": "eks.amazonaws.com"
     },
     "Effect": "Allow",
     "Sid": ""
   }
 ]
}
EOF

 tags = {
   Name = "eks-role"
 }
}

resource "aws_iam_role_policy_attachment" "eks_cluster_role_attachment" {
 role       = aws_iam_role.eks_cluster_role.name
 policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}

resource "aws_iam_role" "eks_node_role" {
 name = "eks-node-role"

 assume_role_policy = <<EOF
{
 "Version": "2012-10-17",
 "Statement": [
   {
     "Action": "sts:AssumeRole",
     "Principal": {
       "Service": "ec2.amazonaws.com"
     },
     "Effect": "Allow",
     "Sid": ""
   }
 ]
}
EOF

 tags = {
   Name = "eks-node-role"
 }
}

resource "aws_iam_role_policy_attachment" "eks_role_attachment" {
 for_each   = toset(local.policies)
 role       = aws_iam_role.eks_node_role.name
 policy_arn = each.value
}
```

## **Step 2 — Prepare the Terraform code that deploys ArgoCD**

For this deployment, we will use the helm provider to deploy the ArgoCD Helm chart in our Kubernetes cluster. It will also deploy a load balancer to access it.

```
data "aws_eks_cluster_auth" "main" {
 name = aws_eks_cluster.main.name
}

resource "helm_release" "argocd" {
 depends_on = [aws_eks_node_group.main]
 name       = "argocd"
 repository = "https://argoproj.github.io/argo-helm"
 chart      = "argo-cd"
 version    = "4.5.2"

 namespace = "argocd"

 create_namespace = true

 set {
   name  = "server.service.type"
   value = "LoadBalancer"
 }

 set {
   name  = "server.service.annotations.service\\.beta\\.kubernetes\\.io/aws-load-balancer-type"
   value = "nlb"
 }
}

data "kubernetes_service" "argocd_server" {
 metadata {
   name      = "argocd-server"
   namespace = helm_release.argocd.namespace
 }
}
```

## **Step 3 — Configure remote state**

It is essential to keep your state in a remote location. You can learn more in our guide: [How to manage Terraform remote state](https://spacelift.io/blog/terraform-remote-state#benefits-of-using-terraform-remote-state).

For our example, we will use an S3 backend:

```
terraform {
 required_version = "1.5.7"
 backend "s3" {
   bucket = "your-bucket-name"
   key    = "your-bucket-key"
   region = "eu-west-1"
 }
}
```

You will need to specify a bucket name, the region where the bucket can be found, and a name for your state file.

## **Step 4 — Run the Terraform code**

First, navigate to the directory that contains your Terraform code and run [*terraform init*](https://spacelift.io/blog/terraform-init).

```
terraform init

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Finding latest version of hashicorp/kubernetes...
- Finding latest version of hashicorp/helm...
- Installing hashicorp/aws v5.50.0...
- Installed hashicorp/aws v5.50.0 (signed by HashiCorp)
- Installing hashicorp/kubernetes v2.23.0...
- Installed hashicorp/kubernetes v2.23.0 (unauthenticated)
- Installing hashicorp/helm v2.13.2...
- Installed hashicorp/helm v2.13.2 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Let’s apply the code:

```
terraform apply -auto-approve

...

Plan: 16 to add, 0 to change, 0 to destroy.

Changes to Outputs:
 + argocd_initial_admin_secret = "kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath={.data.password} | base64 -d"
 + argocd_server_load_balancer = (known after apply)
 + eks_connect                 = "aws eks --region eu-west-1 update-kubeconfig --name main-eks-cluster"
```

It takes about ten minutes for all the resources to be created.

```
Apply complete! Resources: 16 added, 0 changed, 0 destroyed.

Outputs:
argocd_initial_admin_secret = "kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath=\"{.data.password}\" | base64 -d"
argocd_server_load_balancer = "a53a6bf4abebe46f79da6179425ca5f4-7d359c1121a50305.elb.eu-west-1.amazonaws.com"
eks_connect = "aws eks --region eu-west-1 update-kubeconfig --name main-eks-cluster"
```

## **Step 5 — Prepare the Kubernetes manifests to deploy a sample nginx application**

In this step, we want to prepare the configuration that will be deployed with ArgoCD. We will create a simple nginx application that will output “Hello from Argo”. For that, we will need a configmap, a deployment, and a service:

```
# confimap_yaml
apiVersion: v1
kind: ConfigMap
metadata:
 name: nginx-config
 namespace: default
data:
 index.html: |
   <html>
   <head><title>Hello from Argo</title></head>
   <body>
   <h1>Hello from Argo</h1>
   </body>
   </html>
```

In the [Kubernetes configmap](https://spacelift.io/blog/kubernetes-configmap), we save the index.html configuration file that will be used by our nginx deployment.

```
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: nginx-deployment
 namespace: default
spec:
 replicas: 1
 selector:
   matchLabels:
     app: nginx
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:latest
       ports:
       - containerPort: 80
       volumeMounts:
       - name: nginx-config-volume
         mountPath: /usr/share/nginx/html
     volumes:
     - name: nginx-config-volume
       configMap:
         name: nginx-config
```

In the deployment file, we specify which containers we want to use (nginx in our case), and we mount our configmap to get the index.html file.

```
# service.yaml
apiVersion: v1
kind: Service
metadata:
 name: nginx-service
 namespace: default
spec:
 selector:
   app: nginx
 ports:
   - protocol: TCP
     port: 80
     targetPort: 80
 type: LoadBalancer
```

We will expose our application through a load balancer service.

## **Step 6 — Create the ArgoCD application manifest**

Before creating the manifest, we need to ensure that we push this code to a VCS repository. If you are using the repository I have provided, you can leave this file as it is. Otherwise, you should make it point to the correct repository.

The ArgoCD application manifest will look like this:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
 name: nginx-app
 namespace: argocd
spec:
 project: default
 source:
   repoURL: 'https://github.com/saturnhead/eks-argo-terraform'
   targetRevision: HEAD
   path: 'argocd/app'
 destination:
   server: 'https://kubernetes.default.svc'
   namespace: default
 syncPolicy:
   automated:
     prune: true
     selfHeal: true
   syncOptions:
   - CreateNamespace=true
```

Now, we need to connect to our Kubernetes cluster. This can be done by running the following command:

```
aws eks --region eu-west-1 update-kubeconfig --name main-eks-cluster
```

Next, we can run our application manifest:

```
kubectl apply -f argocd-app.yaml
application.argoproj.io/nginx-app created
```

# **Step 7 — Log in to Argo and see the application**

To log in to Argo, we can use the outputs exposed by our Terraform code:

```
argocd_initial_admin_secret = "kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath={.data.password} | base64 -d"
argocd_server_load_balancer = "a53a6bf4abebe46f79da6179425ca5f4-7d359c1121a50305.elb.eu-west-1.amazonaws.com"
```

We need to get the initial ArgoCD admin secret, so we can run the first command for that:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath=\"{.data.password}\" | base64 -d"
```

Now, open a browser and go to the argocd_server_load_balancer address:

[https://miro.medium.com/v2/resize:fit:1400/0*qmmiRTUNtRZ0ibKf](https://miro.medium.com/v2/resize:fit:1400/0*qmmiRTUNtRZ0ibKf)

Use admin as the username and provide the password you got when you ran the first command.

[https://miro.medium.com/v2/resize:fit:1400/0*9fFCe0peDp4B9sRH](https://miro.medium.com/v2/resize:fit:1400/0*9fFCe0peDp4B9sRH)

You should see your application healthy and synced. If you click on it, you will see all the resources that have been deployed with Argo:

[https://miro.medium.com/v2/resize:fit:1400/0*0epg5SR0anFWHt-4](https://miro.medium.com/v2/resize:fit:1400/0*0epg5SR0anFWHt-4)

If you want to access the application itself, select the nginx-service, go to the last line of the manifest, select the hostname, and paste it into your browser:

[https://miro.medium.com/v2/resize:fit:1400/0*0wZLodCKL3g5iL1e](https://miro.medium.com/v2/resize:fit:1400/0*0wZLodCKL3g5iL1e)

# **Managing Kubernetes cluster with ArgoCD, Terraform, and GitHub Actions**

Let’s take the above automation to the next step and deploy it through a GitHub Actions pipeline to enable collaboration and ensure that you won’t need to do anything manually.

We will build three different workflows:

1. Terraform deployment
2. ArgoCD application deployment
3. Tearing down the infrastructure and application

## **Step 1 — Prerequisites**

For all of these deployments, we need to define some GitHub Actions secrets for the AWS credentials. We need to use:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_SESSION_TOKEN
```

To add a secret to your repository, navigate to the repository’s settings, select **Secrets and Variables**, and click on **Actions**.

[https://miro.medium.com/v2/resize:fit:1400/0*PLpxrBAhtJcK827v](https://miro.medium.com/v2/resize:fit:1400/0*PLpxrBAhtJcK827v)

## **Step 2 — Prepare the Terraform creation workflow**

You can find the workflow file [here](https://github.com/saturnhead/eks-argo-terraform/blob/main/.github/workflows/terraform_deployment.yaml).

This workflow will run on pull requests, and main branch merges whenever there are changes to the Terraform directory in the repository. For pull requests, it will also show the plan as a PR comment.

The workflow does the following:

1. Checks out the code
2. Sets up Terraform
3. Sets up the AWS credentials
4. Runs `terraform init`
5. Runs `terraform validate` to ensure the configuration is valid
6. Runs `terraform fmt` to check if the code is formatted correctly
7. Runs a `terraform plan` and comments on the plan on a PR only on pull requests
8. Runs a `terraform apply` on a master branch push

## **Step 3 — Prepare the Argo creation workflow**

The workflow file can be found [here](https://github.com/saturnhead/eks-argo-terraform/blob/main/.github/workflows/argo.yaml).

This workflow will run whenever the ArgoCD folder changes or when the Terraform workflow finishes successfully from the main branch.

It goes through the following steps:

1. Checks out the code
2. Verifies if the workflow was triggered from the main branch
3. Configures AWS credentials
4. Installs kubectl
5. Updates the kubeconfig
6. Deploys the application

## **Step 4 — Prepare the destruction workflow [Optional]**

The workflow file can be found [here](https://github.com/saturnhead/eks-argo-terraform/blob/main/.github/workflows/destroy.yaml).

It only runs manually and goes through the following steps:

1. Checks out the code
2. Sets up Terraform
3. Sets up the AWS credentials
4. Installs kubectl
5. Deletes the argocd application
6. Runs `terraform init`
7. Runs `terraform destroy`

## **Step 5 — Trigger a run**

Make a dummy change to your Terraform code and see the pipelines get triggered (I just added a new line).

[https://miro.medium.com/v2/resize:fit:1400/0*w2UiyAOfMM6J2IsW](https://miro.medium.com/v2/resize:fit:1400/0*w2UiyAOfMM6J2IsW)

[https://miro.medium.com/v2/resize:fit:1400/0*xi7xS_mAt13R93vf](https://miro.medium.com/v2/resize:fit:1400/0*xi7xS_mAt13R93vf)

# **Managing Terraform & Kubernetes with Spacelift**

To take your deployment to the ultimate level, you can easily leverage Spacelift.

Spacelift supports both [Terraform](https://docs.spacelift.io/vendors/terraform/) and [Kubernetes](https://docs.spacelift.io/vendors/kubernetes/) and enables users to create [stacks](https://docs.spacelift.io/concepts/stack/) based on them. Leveraging Spacelift, you can build CI/CD pipelines to combine them and get the best of each tool. This way, you will use a single tool to manage your Terraform and Kubernetes resources lifecycle, allow your teams to collaborate easily, and add some necessary security controls to your workflows.

## **Step 1 — Build the stack automation**

Let’s build the same integration we built for GitHub actions. To automate as much as possible, I will create the deployment stacks using [OpenTofu](https://spacelift.io/blog/what-is-opentofu):

```
provider "spacelift" {}

terraform {
 required_providers {
   spacelift = {
     source = "spacelift-io/spacelift"
   }
 }
}

resource "spacelift_stack" "K8s-cluster" {
 branch            = "main"
 description       = "Provisions a Kubernetes cluster"
 name              = "Terraform Kubernetes Cluster"
 project_root      = "terraform"
 repository        = "eks-argo-terraform"
 terraform_version = "1.5.7"

 labels = ["terraform-argocd"]
}

resource "spacelift_stack" "argocd" {
 kubernetes {
   namespace = "argocd"
 }
 branch       = "main"
 description  = "Deploys an ArgoCD application"
 name         = "ArgoCD application"
 project_root = "argocd/config"
 repository   = "eks-argo-terraform"
 labels       = ["terraform-argocd"]
 before_init  = ["$AWS_LOGIN"]
}

resource "spacelift_aws_integration_attachment" "K8s-cluster" {
 integration_id = var.integration_id
 stack_id       = spacelift_stack.K8s-cluster.id
 read           = true
 write          = true
}

resource "spacelift_aws_integration_attachment" "argocd" {
 integration_id = var.integration_id
 stack_id       = spacelift_stack.argocd.id
 read           = true
 write          = true
}

resource "spacelift_stack_dependency" "cluster-argo" {
 stack_id            = spacelift_stack.argocd.id
 depends_on_stack_id = spacelift_stack.K8s-cluster.id
}

resource "spacelift_stack_dependency_reference" "output" {
 stack_dependency_id = spacelift_stack_dependency.cluster-argo.id
 output_name         = "eks_connect"
 input_name          = "AWS_LOGIN"
}
```

We are creating two stacks that will leverage an existing cloud integration for AWS (this will generate dynamic credentials), and we will build a stack dependency between them. The Terraform stack will deploy the Kubernetes cluster and ArgoCD as before, and the K8s stack will depend on it and receive the kubeconfig login command as an output. The stacks will leverage the same code as before.

You can check out how to configure your own cloud integration [here](https://docs.spacelift.io/integrations/cloud-providers/aws).

## **Step 2 — Create the stack inside Spacelift**

Let’s create this stack inside Spacelift. First, go to **Stacks** and select **Create Stack**:

[https://miro.medium.com/v2/resize:fit:1400/0*jbiYDkUS0o8jC1KY](https://miro.medium.com/v2/resize:fit:1400/0*jbiYDkUS0o8jC1KY)

Then, select the repository and the path to the Spacelift configuration:

[https://miro.medium.com/v2/resize:fit:1400/0*HIv58AhLu8p-XuBz](https://miro.medium.com/v2/resize:fit:1400/0*HIv58AhLu8p-XuBz)

Next, select the tool you want to use to provision the resources. We will use OpenTofu, but you can use Terraform instead:

[https://miro.medium.com/v2/resize:fit:1400/0*XO6G1Lj8TnxVT3NU](https://miro.medium.com/v2/resize:fit:1400/0*XO6G1Lj8TnxVT3NU)

Next, in the **Define Behavior** tab check the **Administrative** option. This will let you provision Spacelift resources inside your account:

[https://miro.medium.com/v2/resize:fit:1400/0*XTiKT82RFnadiL-V](https://miro.medium.com/v2/resize:fit:1400/0*XTiKT82RFnadiL-V)

Skip to **Summary** and finish the stack creation wizard. Before running the code, we need to add the environment variable for the cloud integration ID that will be passed to both configurations (Terraform and K8s). This can be done from the **Environment** tab. Ensure the variable name is prefixed with “TF_VAR”. The integration_id is the id of the integration that you’ve previously built by following the documentation.

[https://miro.medium.com/v2/resize:fit:1400/0*q1XQnbkBzbT4EX2A](https://miro.medium.com/v2/resize:fit:1400/0*q1XQnbkBzbT4EX2A)

## **Step 3 — Create the Terraform and K8s stacks using the admin stack**

Now, we are ready to run the code. Return to the **Tracked runs** tab and trigger a run.

[https://miro.medium.com/v2/resize:fit:1400/0*xSG_9E-4aoEqXLc7](https://miro.medium.com/v2/resize:fit:1400/0*xSG_9E-4aoEqXLc7)

Confirm the run and wait for the resources to be created. The apply should take less than ten seconds:

[https://miro.medium.com/v2/resize:fit:1400/0*u2s7mjsqxN-fg4TU](https://miro.medium.com/v2/resize:fit:1400/0*u2s7mjsqxN-fg4TU)

Next, if you go to your stacks, you should see two new stacks created:

[https://miro.medium.com/v2/resize:fit:1400/0*QOVPBKjP28llo8pT](https://miro.medium.com/v2/resize:fit:1400/0*QOVPBKjP28llo8pT)

## **Step 4 — Run the Terraform Kubernetes cluster stack and wait for its dependencies to be triggered**

Now, you can either trigger a run directly on the Terraform Kubernetes cluster stack, or change the Terraform code. For simplicity, let’s just trigger a run as we did before:

[https://miro.medium.com/v2/resize:fit:1400/0*Y7lA7uqYX7hTXlQ9](https://miro.medium.com/v2/resize:fit:1400/0*Y7lA7uqYX7hTXlQ9)

We can see all the resources that will be created in our infrastructure. Let’s confirm the run and let’s check the K8s stack until the apply finishes and see what is happening with it:

[https://miro.medium.com/v2/resize:fit:1400/0*R88VhRCZCAYhRqe5](https://miro.medium.com/v2/resize:fit:1400/0*R88VhRCZCAYhRqe5)

As you can see, this one has a run queued and is waiting for the Terraform stack to finish before applying.

The Terraform stack finished running, and the run on the K8s one was triggered:

[https://miro.medium.com/v2/resize:fit:1400/0*KGqdn03AwYCcqHLy](https://miro.medium.com/v2/resize:fit:1400/0*KGqdn03AwYCcqHLy)

[https://miro.medium.com/v2/resize:fit:1400/0*z8J2SgbYv1Jrv8BX](https://miro.medium.com/v2/resize:fit:1400/0*z8J2SgbYv1Jrv8BX)

A few seconds after confirming the run, the resource is created:

[https://miro.medium.com/v2/resize:fit:1400/0*ndVQSZrDNQgedaWn](https://miro.medium.com/v2/resize:fit:1400/0*ndVQSZrDNQgedaWn)

To see the outputs from the Terraform stack, we can navigate back to the Terraform stack and select the **Outputs** tab:

[https://miro.medium.com/v2/resize:fit:1400/0*4eIynae1PQLphRcq](https://miro.medium.com/v2/resize:fit:1400/0*4eIynae1PQLphRcq)

We can log in to the Argo instance by repeating the process we’ve done before, logging in to the K8s cluster, getting the initial admin secret, and navigating to the argocd server load balancer:

[https://miro.medium.com/v2/resize:fit:1400/0*kBKVC8MFvlADXZYw](https://miro.medium.com/v2/resize:fit:1400/0*kBKVC8MFvlADXZYw)

# **Key points**

In this post, we’ve covered how to use Terraform with ArgoCD and shown how the deployment can be configured.

Deploying from a local environment will not work if you are collaborating with multiple engineers, and using GitHub Actions can prove complicated — especially when configuring the pipeline itself. Using Spacelift, you benefit from out-of-the-box workflows for your favorite infrastructure tools, and you can easily build dependable workflows.

If you want to learn more about Spacelift, [create a free account](https://spacelift.io/free-trial) today, or [book a demo](https://spacelift.io/schedule-demo) with one of our engineers.

*Originally published at [https://spacelift.io](https://spacelift.io/blog/argocd-terraform).*

[Terraform](https://medium.com/tag/terraform?source=post_page-----a70e9d852d89---------------terraform-----------------)

[Argo](https://medium.com/tag/argo?source=post_page-----a70e9d852d89---------------argo-----------------)

[Cloud](https://medium.com/tag/cloud?source=post_page-----a70e9d852d89---------------cloud-----------------)

[DevOps](https://medium.com/tag/devops?source=post_page-----a70e9d852d89---------------devops-----------------)