---
title: Tag and Push Docker Image to AWS ECR 
date: 2023-03-06 15:29:00 +0800  
categories: [English, AWS Learning Journey]  
tags: [python, docker]  
---
This article introduces how create ECR repository using terraform then to push a local docker image to ECR on AWS.

## References
+ [Copy a container image from one repository to another repository](https://docs.aws.amazon.com/eks/latest/userguide/copy-image-to-repository.html)

## Prerequisites
+ An image is ready in local docker repository. See [Create a Web Application Using Flask in Python 3](/posts/Create-a-Web-Application-Using-Flask-in-Python-3/)
+ [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
+ [Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
+ [Collapsible Code Blocks in GitHub Pages](https://www.endtoend.ai/tutorial/collapsible-code-blocks/)

## Tag
Ensure the docker image has been built locally
```
PS C:\flask> docker image ls
REPOSITORY                                                TAG                     IMAGE ID       CREATED             SIZE
flask-web                                                 latest                  f98f208ab295   3 minutes ago       124MB
```
Tag the docker image with `docker tag {YOUR REPOSITORY}:{TAG} {AWS ACCOUNT ID}.dkr.ecr.{REGION}.amazonaws.com/{TARGET ECR REPOSITORY}:{Image tag}`
```
PS C:\flask> docker tag flask-web:latest 123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/flask-web:v20230303
PS C:\flask> docker image ls
REPOSITORY                                                TAG                     IMAGE ID       CREATED          SIZE
123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/flask-web    v20230303               f98f208ab295   54 minutes ago   124MB
flask-web                                                 latest                  f98f208ab295   54 minutes ago   124MB
```

## Create terraform code and run
See terrafrom code from [01-ecr](https://github.com/hivsuper/demos/tree/master/aws-demo/terraform/app/01-ecr). 

```
resource "aws_ecr_repository" "flask" {
  name = local.flask-web
  tags = module.flask-tags.tags

  image_scanning_configuration {
    scan_on_push = true
  }
}

module "common-ecs-ecr-policy-flask" {
  source = "../../modules/ecs/resource/ecr/policy"
  name   = aws_ecr_repository.flask.name
}

variable "flask" {
  type = string
}

locals {
  flask-web = format("%s-web", var.flask)
}
```
{: file='aws-demo/terraform/app/01-ecr/flask.tf'}

Run `aws ecr describe-repositories` to ensure the target repository doesn't exist.
```
root@777777777777:/data/demos/aws-demo/terraform/scripts# aws ecr describe-repositories --profile test-aws
{
    "repositories": []
}
```
Deploy the terraform module to create ECR repository on AWS
<details><summary markdown="span">Deploy 01-ecr console</summary>

```
root@777777777777:/data/demos/aws-demo/terraform/scripts# ./terraform.sh deploy app 01-ecr
---------- start to deploy 01-ecr ----------
Initializing modules...

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v4.57.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_ecr_repository.flask will be created
  + resource "aws_ecr_repository" "flask" {
      + arn                  = (known after apply)
      + id                   = (known after apply)
      + image_tag_mutability = "MUTABLE"
      + name                 = "flask-web"
      + registry_id          = (known after apply)
      + repository_url       = (known after apply)
      + tags                 = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all             = {
          + "SERVICE_ID" = "flask"
        }

      + image_scanning_configuration {
          + scan_on_push = true
        }
    }

  # module.common-ecs-ecr-policy-flask.aws_ecr_lifecycle_policy.policy will be created
  + resource "aws_ecr_lifecycle_policy" "policy" {
      + id          = (known after apply)
      + policy      = jsonencode(
            {
              + rules = [
                  + {
                      + action       = {
                          + type = "expire"
                        }
                      + description  = "Keep last 10 images"
                      + rulePriority = 1
                      + selection    = {
                          + countNumber   = 10
                          + countType     = "imageCountMoreThan"
                          + tagPrefixList = [
                              + "v",
                            ]
                          + tagStatus     = "tagged"
                        }
                    },
                  + {
                      + action       = {
                          + type = "expire"
                        }
                      + description  = "Only keep one untagged image"
                      + rulePriority = 2
                      + selection    = {
                          + countNumber = 1
                          + countType   = "imageCountMoreThan"
                          + tagStatus   = "untagged"
                        }
                    },
                  + {
                      + action       = {
                          + type = "expire"
                        }
                      + description  = "Keep at most 10 tagged images"
                      + rulePriority = 3
                      + selection    = {
                          + countNumber = 11
                          + countType   = "imageCountMoreThan"
                          + tagStatus   = "any"
                        }
                    },
                ]
            }
        )
      + registry_id = (known after apply)
      + repository  = "flask-web"
    }

Plan: 2 to add, 0 to change, 0 to destroy.
╷
│ Warning: Value for undeclared variable
│
│ The root module does not declare a variable named "vpc_id" but a value was found in file "../../scripts/terraform.app.tfvars". If you meant to use this value, add a "variable" block to the configuration.
│
│ To silence these warnings, use TF_VAR_... environment variables to provide certain "global" settings to all configurations in your organization. To reduce the verbosity of these warnings, use the -compact-warnings option.
╵
╷
│ Warning: Value for undeclared variable
│
│ The root module does not declare a variable named "aws_availability_zones" but a value was found in file "../../scripts/terraform.app.tfvars". If you meant to use this value, add a "variable" block to the configuration.
│
│ To silence these warnings, use TF_VAR_... environment variables to provide certain "global" settings to all configurations in your organization. To reduce the verbosity of these warnings, use the -compact-warnings option.
╵

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_ecr_repository.flask: Creating...
aws_ecr_repository.flask: Creation complete after 1s [id=flask-web]
module.common-ecs-ecr-policy-flask.aws_ecr_lifecycle_policy.policy: Creating...
module.common-ecs-ecr-policy-flask.aws_ecr_lifecycle_policy.policy: Creation complete after 1s [id=flask-web]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
---------- end to deploy 01-ecr ----------
root@777777777777:/data/demos/aws-demo/terraform/scripts#
```
</details>

Deployment succeeded
```
root@777777777777:/data/demos/aws-demo/terraform/scripts# aws ecr describe-repositories --profile test-aws
{
    "repositories": [
        {
            "repositoryArn": "arn:aws:ecr:ap-southeast-1:123456789012:repository/flask-web",
            "registryId": "123456789012",
            "repositoryName": "flask-web",
            "repositoryUri": "123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/flask-web",
            "createdAt": "2023-03-06T15:01:29+08:00",
            "imageTagMutability": "MUTABLE",
            "imageScanningConfiguration": {
                "scanOnPush": true
            },
            "encryptionConfiguration": {
                "encryptionType": "AES256"
            }
        }
    ]
}
root@777777777777:/data/demos/aws-demo/terraform/scripts#
```

## Push
Login AWS with `aws ecr get-login-password --region {REGION} --profile {PROFILE}| docker login --username AWS --password-stdin {AWS ACCOUNT ID}.dkr.ecr.{REGION}.amazonaws.com`
```
PS C:\flask> aws ecr get-login-password --region ap-southeast-1 --profile test-aws| docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-southeast-1.amazonaws.com
Login Succeeded
```
Push with `docker push {AWS ACCOUNT ID}.dkr.ecr.{REGION}.amazonaws.com/{TARGET ECR REPOSITORY}:{Image tag}`
```
PS C:\flask> docker push 123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/flask-web:v20230303
The push refers to repository [123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/flask-web]
7af31f90d4cf: Pushed
ace0817c9ed6: Pushed
7f6d0cadef3c: Pushed
55d265de0062: Pushed
cb0f81202195: Pushed
b6d1c6840e97: Pushed
aa1acb65f1a2: Pushed
f7cd9720fc7e: Pushed
7cd52847ad77: Pushed
v20230303: digest: sha256:7bee037e4a59183dbbe55e47b188b773ff00fec8df9e38c54e936a04bc8f23db size: 2201
PS C:\flask>
```
The image is pushed successfully
```
root@777777777777:/data/demos/aws-demo/terraform/scripts# aws ecr list-images --repository-name flask-web
{
    "imageIds": [
        {
            "imageDigest": "sha256:7bee037e4a59183dbbe55e47b188b773ff00fec8df9e38c54e936a04bc8f23db",
            "imageTag": "v20230303"
        }
    ]
}
root@777777777777:/data/demos/aws-demo/terraform/scripts#
```
