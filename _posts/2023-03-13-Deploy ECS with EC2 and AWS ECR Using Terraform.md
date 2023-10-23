---
title: Deploy ECS with EC2 and AWS ECR Using Terraform  
date: 2023-03-13 11:33:00 +0800  
categories: [Technology, AWS Learning Journey]  
tags: [python, docker]  
---
This article introduces how to create ECS service running with EC2 and an ECR image on AWS using terraform.

## Prerequisites
+ An image has been pushed to AWS ECR. See [Tag and Push Docker Image to AWS ECR](/posts/Tag-and-Push-Docker-Image-to-AWS-ECR/)

## Create terraform code and run
See terrafrom code from [02-ecs](https://github.com/hivsuper/learning-journey/tree/master/demos/tree/master/aws-demo/terraform/app/02-ecs). 
<details><summary markdown="span">Deploy 02-ecs console</summary>

```
root@777777777777:/data/demos/aws-demo/terraform/scripts# ./terraform.sh deploy app 02-ecs
---------- start to deploy 02-ecs ----------
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
var.ecr_tag_version
  Enter a value: v20230303

data.aws_region.current: Reading...
module.common-vpc.data.aws_vpc.vpc-aws-default: Reading...
module.common-ecs-ecr.data.aws_ecr_repository.repository: Reading...
data.aws_region.current: Read complete after 0s [id=ap-southeast-1]
module.common-ami.data.aws_ami.image: Reading...
module.common-ami.data.aws_ami.image: Read complete after 1s [id=ami-0fd1ee6c8b656f020]
module.common-ecs-ecr.data.aws_ecr_repository.repository: Read complete after 1s [id=flask-web]
module.common-vpc.data.aws_vpc.vpc-aws-default: Read complete after 1s [id=vpc-b68844d0]
module.common-vpc.data.aws_subnets.subnets-aws-default: Reading...
module.common-vpc.data.aws_subnets.subnets-aws-default: Read complete after 0s [id=ap-southeast-1]
module.common-vpc.data.aws_subnet.subnet-aws-default["subnet-114d2148"]: Reading...
module.common-vpc.data.aws_subnet.subnet-aws-default["subnet-1c60d17a"]: Reading...
module.common-vpc.data.aws_subnet.subnet-aws-default["subnet-ccb61784"]: Reading...
module.common-vpc.data.aws_subnet.subnet-aws-default["subnet-1c60d17a"]: Read complete after 0s [id=subnet-1c60d17a]
module.common-vpc.data.aws_subnet.subnet-aws-default["subnet-114d2148"]: Read complete after 0s [id=subnet-114d2148]
module.common-vpc.data.aws_subnet.subnet-aws-default["subnet-ccb61784"]: Read complete after 0s [id=subnet-ccb61784]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_autoscaling_group.asg-flask will be created
  + resource "aws_autoscaling_group" "asg-flask" {
      + arn                       = (known after apply)
      + availability_zones        = [
          + "ap-southeast-1a",
          + "ap-southeast-1b",
          + "ap-southeast-1c",
        ]
      + default_cooldown          = (known after apply)
      + desired_capacity          = 1
      + force_delete              = false
      + force_delete_warm_pool    = false
      + health_check_grace_period = 300
      + health_check_type         = (known after apply)
      + id                        = (known after apply)
      + max_size                  = 1
      + metrics_granularity       = "1Minute"
      + min_size                  = 1
      + name                      = "flask-service"
      + name_prefix               = (known after apply)
      + protect_from_scale_in     = false
      + service_linked_role_arn   = (known after apply)
      + vpc_zone_identifier       = (known after apply)
      + wait_for_capacity_timeout = "10m"

      + instance_refresh {
          + strategy = "Rolling"
          + triggers = [
              + "tag",
              + "tags",
            ]

          + preferences {
              + instance_warmup        = "300"
              + min_healthy_percentage = 90
              + skip_matching          = false
            }
        }

      + launch_template {
          + id      = (known after apply)
          + name    = (known after apply)
          + version = (known after apply)
        }

      + tag {
          + key                 = "SERVICE_ID"
          + propagate_at_launch = true
          + value               = "flask"
        }
    }

  # aws_cloudwatch_log_group.cloudwatch-log-group-flask will be created
  + resource "aws_cloudwatch_log_group" "cloudwatch-log-group-flask" {
      + arn               = (known after apply)
      + id                = (known after apply)
      + name              = "flask-cluster"
      + name_prefix       = (known after apply)
      + retention_in_days = 7
      + skip_destroy      = false
      + tags              = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all          = {
          + "SERVICE_ID" = "flask"
        }
    }

  # aws_ecs_cluster.common-ecs-cluster-flask will be created
  + resource "aws_ecs_cluster" "common-ecs-cluster-flask" {
      + arn                = (known after apply)
      + capacity_providers = (known after apply)
      + id                 = (known after apply)
      + name               = "flask-cluster"
      + tags               = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all           = {
          + "SERVICE_ID" = "flask"
        }

      + default_capacity_provider_strategy {
          + base              = (known after apply)
          + capacity_provider = (known after apply)
          + weight            = (known after apply)
        }

      + setting {
          + name  = (known after apply)
          + value = (known after apply)
        }
    }

  # aws_ecs_service.service-ec2-no-scaling will be created
  + resource "aws_ecs_service" "service-ec2-no-scaling" {
      + cluster                            = "flask-cluster"
      + deployment_minimum_healthy_percent = 0
      + enable_ecs_managed_tags            = false
      + enable_execute_command             = false
      + iam_role                           = (known after apply)
      + id                                 = (known after apply)
      + launch_type                        = (known after apply)
      + name                               = "flask-service"
      + platform_version                   = (known after apply)
      + propagate_tags                     = "SERVICE"
      + scheduling_strategy                = "DAEMON"
      + tags                               = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all                           = {
          + "SERVICE_ID" = "flask"
        }
      + task_definition                    = (known after apply)
      + triggers                           = (known after apply)
      + wait_for_steady_state              = false

      + load_balancer {
          + container_name   = "flask-service"
          + container_port   = 8000
          + target_group_arn = (known after apply)
        }

      + placement_constraints {
          + expression = "attribute:service == flask-service"
          + type       = "memberOf"
        }
    }

  # aws_ecs_task_definition.common-ecs-task-definition-flask will be created
  + resource "aws_ecs_task_definition" "common-ecs-task-definition-flask" {
      + arn                   = (known after apply)
      + container_definitions = jsonencode(
            [
              + {
                  + cpu              = 1024
                  + essential        = true
                  + image            = "123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/flask-web:v20230303"
                  + logConfiguration = {
                      + logDriver = "awslogs"
                      + options   = {
                          + awslogs-group         = "flask-cluster"
                          + awslogs-region        = "ap-southeast-1"
                          + awslogs-stream-prefix = "ecs"
                        }
                    }
                  + memory           = 768
                  + name             = "flask-service"
                  + portMappings     = [
                      + {
                          + containerPort = 8000
                          + hostPort      = 80
                        },
                    ]
                },
            ]
        )
      + cpu                   = "1024"
      + family                = "flask-service"
      + id                    = (known after apply)
      + memory                = "768"
      + network_mode          = (known after apply)
      + revision              = (known after apply)
      + skip_destroy          = false
      + tags                  = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all              = {
          + "SERVICE_ID" = "flask"
        }
      + task_role_arn         = (known after apply)
    }

  # aws_iam_instance_profile.iam-instance-profile-assume-role-flask will be created
  + resource "aws_iam_instance_profile" "iam-instance-profile-assume-role-flask" {
      + arn         = (known after apply)
      + create_date = (known after apply)
      + id          = (known after apply)
      + name        = "flask-service-role"
      + path        = "/service-role/"
      + role        = (known after apply)
      + tags        = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all    = {
          + "SERVICE_ID" = "flask"
        }
      + unique_id   = (known after apply)
    }

  # aws_iam_role.iam-role-flask will be created
  + resource "aws_iam_role" "iam-role-flask" {
      + arn                   = (known after apply)
      + assume_role_policy    = jsonencode(
            {
              + Statement = [
                  + {
                      + Action    = [
                          + "sts:AssumeRole",
                        ]
                      + Effect    = "Allow"
                      + Principal = {
                          + Service = [
                              + "ec2.amazonaws.com",
                              + "ecs-tasks.amazonaws.com",
                            ]
                        }
                    },
                ]
              + Version   = "2012-10-17"
            }
        )
      + create_date           = (known after apply)
      + force_detach_policies = false
      + id                    = (known after apply)
      + managed_policy_arns   = (known after apply)
      + max_session_duration  = 3600
      + name                  = (known after apply)
      + name_prefix           = "flask-service-role"
      + path                  = "/service-role/"
      + tags                  = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all              = {
          + "SERVICE_ID" = "flask"
        }
      + unique_id             = (known after apply)

      + inline_policy {
          + name   = (known after apply)
          + policy = (known after apply)
        }
    }

  # aws_iam_role_policy.flask-permissions will be created
  + resource "aws_iam_role_policy" "flask-permissions" {
      + id     = (known after apply)
      + name   = "flask-service-role"
      + policy = jsonencode(
            {
              + Statement = [
                  + {
                      + Action   = [
                          + "logs:*",
                        ]
                      + Effect   = "Allow"
                      + Resource = "*"
                    },
                  + {
                      + Action   = [
                          + "s3:*",
                        ]
                      + Effect   = "Allow"
                      + Resource = "*"
                    },
                ]
              + Version   = "2012-10-17"
            }
        )
      + role   = (known after apply)
    }

  # aws_iam_role_policy_attachment.flask-permissions["arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"] will be created
  + resource "aws_iam_role_policy_attachment" "flask-permissions" {
      + id         = (known after apply)
      + policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"
      + role       = (known after apply)
    }

  # aws_launch_template.aws-launch-template-flask will be created
  + resource "aws_launch_template" "aws-launch-template-flask" {
      + arn                     = (known after apply)
      + default_version         = (known after apply)
      + disable_api_termination = false
      + id                      = (known after apply)
      + image_id                = "ami-0fd1ee6c8b656f020"
      + instance_type           = "t2.micro"
      + latest_version          = (known after apply)
      + name                    = (known after apply)
      + name_prefix             = "flask-service"
      + security_group_names    = (known after apply)
      + tags                    = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all                = {
          + "SERVICE_ID" = "flask"
        }
      + user_data               = "IyEvdXNyL2Jpbi9lbnYgYmFzaAojIEluc3RhbGwgZG9ja2VyIGluIEVDMgojIGh0dHBzOi8vd3d3LmNuYmxvZ3MuY29tL0Nhbi1kYXlkYXl1cC9wLzE2NDcyMzc1Lmh0bWwKc3VkbyAtcwphcHQtZ2V0IGluc3RhbGwgY2EtY2VydGlmaWNhdGVzIGN1cmwgZ251cGcgbHNiLXJlbGVhc2UKbWtkaXIgLXAgL2V0Yy9hcHQva2V5cmluZ3MKY3VybCAtZnNTTCBodHRwczovL2Rvd25sb2FkLmRvY2tlci5jb20vbGludXgvdWJ1bnR1L2dwZyB8IHN1ZG8gZ3BnIC0tZGVhcm1vciAtbyAvZXRjL2FwdC9rZXlyaW5ncy9kb2NrZXIuZ3BnCmVjaG8gImRlYiBbYXJjaD0kKGRwa2cgLS1wcmludC1hcmNoaXRlY3R1cmUpIHNpZ25lZC1ieT0vZXRjL2FwdC9rZXlyaW5ncy9kb2NrZXIuZ3BnXSBodHRwczovL2Rvd25sb2FkLmRvY2tlci5jb20vbGludXgvdWJ1bnR1ICQobHNiX3JlbGVhc2UgLWNzKSBzdGFibGUiIHwgc3VkbyB0ZWUgL2V0Yy9hcHQvc291cmNlcy5saXN0LmQvZG9ja2VyLmxpc3QgPiAvZGV2L251bGwKYXB0LWdldCB1cGRhdGUKYXB0LWdldCAtLXllcyBpbnN0YWxsIGRvY2tlci1jZSBkb2NrZXItY2UtY2xpIGNvbnRhaW5lcmQuaW8gZG9ja2VyLWNvbXBvc2UtcGx1Z2luCiMgSW5zdGFsbCBlY3MgYWdlbnQgaW4gRUMyCiMgaHR0cHM6Ly9kb2NzLmF3cy5hbWF6b24uY29tL0FtYXpvbkVDUy9sYXRlc3QvZGV2ZWxvcGVyZ3VpZGUvZWNzLWFnZW50LWluc3RhbGwuaHRtbApjdXJsIC1PIGh0dHBzOi8vczMudXMtd2VzdC0yLmFtYXpvbmF3cy5jb20vYW1hem9uLWVjcy1hZ2VudC11cy13ZXN0LTIvYW1hem9uLWVjcy1pbml0LWxhdGVzdC5hbWQ2NC5kZWIKZHBrZyAtaSBhbWF6b24tZWNzLWluaXQtbGF0ZXN0LmFtZDY0LmRlYgpybSBhbWF6b24tZWNzLWluaXQtbGF0ZXN0LmFtZDY0LmRlYgoKIyBTZXQgdXNlcmRhdGEKIyBodHRwczovL2Rldi50by9hd3Njb21tdW5pdHktYXNlYW4vbmV3LWVjcy1pbnN0YW5jZS1yZWdpc3RyYXRpb24td2l0aC1lY3MtY2x1c3Rlci11c2luZy1jbWQtYW5kLWNvbnNvbGUtMTlmNQojIGh0dHBzOi8vZG9jcy5hd3MuYW1hem9uLmNvbS9BbWF6b25FQ1MvbGF0ZXN0L2RldmVsb3Blcmd1aWRlL2Vjcy1hZ2VudC1jb25maWcuaHRtbApjYXQgPDxFT0YgPi9ldGMvZWNzL2Vjcy5jb25maWcKRUNTX0NMVVNURVI9Zmxhc2stY2x1c3RlcgpFQ1NfRU5BQkxFX0NPTlRBSU5FUl9NRVRBREFUQT10cnVlCkVDU19JTlNUQU5DRV9BVFRSSUJVVEVTPXsic2VydmljZSI6ImZsYXNrLXNlcnZpY2UifQpFQ1NfUkVTRVJWRURfTUVNT1JZPTAKRUNTX0NPTlRBSU5FUl9JTlNUQU5DRV9UQUdTPXsic2VydmljZSI6ImZsYXNrLXNlcnZpY2UifQoKRU9GCiMgU3RhcnQgRUNTCnN1ZG8gc3lzdGVtY3RsIHN0YXJ0IGVjcw=="

      + iam_instance_profile {
          + arn = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_protocol_ipv6          = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + tag_specifications {
          + resource_type = "instance"
          + tags          = {
              + "Name" = "flask-service"
            }
        }
      + tag_specifications {
          + resource_type = "volume"
          + tags          = {
              + "Name" = "flask-service"
            }
        }
    }

  # aws_lb.lb-flask will be created
  + resource "aws_lb" "lb-flask" {
      + arn                        = (known after apply)
      + arn_suffix                 = (known after apply)
      + desync_mitigation_mode     = "defensive"
      + dns_name                   = (known after apply)
      + drop_invalid_header_fields = false
      + enable_deletion_protection = false
      + enable_http2               = true
      + enable_waf_fail_open       = false
      + id                         = (known after apply)
      + idle_timeout               = 60
      + internal                   = false
      + ip_address_type            = (known after apply)
      + load_balancer_type         = "application"
      + name                       = "flask-web-lb"
      + preserve_host_header       = false
      + security_groups            = (known after apply)
      + subnets                    = [
          + "subnet-114d2148",
          + "subnet-1c60d17a",
          + "subnet-ccb61784",
        ]
      + tags                       = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all                   = {
          + "SERVICE_ID" = "flask"
        }
      + vpc_id                     = (known after apply)
      + zone_id                    = (known after apply)

      + subnet_mapping {
          + allocation_id        = (known after apply)
          + ipv6_address         = (known after apply)
          + outpost_id           = (known after apply)
          + private_ipv4_address = (known after apply)
          + subnet_id            = (known after apply)
        }
    }

  # aws_lb_listener.lb-listener-flask will be created
  + resource "aws_lb_listener" "lb-listener-flask" {
      + arn               = (known after apply)
      + id                = (known after apply)
      + load_balancer_arn = (known after apply)
      + port              = 80
      + protocol          = "HTTP"
      + ssl_policy        = (known after apply)
      + tags              = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all          = {
          + "SERVICE_ID" = "flask"
        }

      + default_action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }
    }

  # aws_lb_listener_rule.lb-listener-rule-flask will be created
  + resource "aws_lb_listener_rule" "lb-listener-rule-flask" {
      + arn          = (known after apply)
      + id           = (known after apply)
      + listener_arn = (known after apply)
      + priority     = 100
      + tags         = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all     = {
          + "SERVICE_ID" = "flask"
        }

      + action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }

      + condition {

          + path_pattern {
              + values = [
                  + "/",
                  + "/*",
                ]
            }
        }
    }

  # aws_lb_target_group.alb-tg-flask will be created
  + resource "aws_lb_target_group" "alb-tg-flask" {
      + arn                                = (known after apply)
      + arn_suffix                         = (known after apply)
      + connection_termination             = false
      + deregistration_delay               = "300"
      + id                                 = (known after apply)
      + ip_address_type                    = (known after apply)
      + lambda_multi_value_headers_enabled = false
      + load_balancing_algorithm_type      = (known after apply)
      + name                               = "flask-web-alb-tg"
      + port                               = 80
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTP"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags                               = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all                           = {
          + "SERVICE_ID" = "flask"
        }
      + target_type                        = "instance"
      + vpc_id                             = "vpc-b68844d0"

      + health_check {
          + enabled             = true
          + healthy_threshold   = 5
          + interval            = 90
          + matcher             = "200-499"
          + path                = "/"
          + port                = "traffic-port"
          + protocol            = "HTTP"
          + timeout             = 60
          + unhealthy_threshold = 2
        }

      + stickiness {
          + cookie_duration = 86400
          + enabled         = true
          + type            = "lb_cookie"
        }

      + target_failover {
          + on_deregistration = (known after apply)
          + on_unhealthy      = (known after apply)
        }
    }

  # aws_security_group.security_group will be created
  + resource "aws_security_group" "security_group" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = (known after apply)
      + name_prefix            = "flask-sg"
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "SERVICE_ID" = "flask"
        }
      + tags_all               = {
          + "SERVICE_ID" = "flask"
        }
      + vpc_id                 = "vpc-b68844d0"
    }

  # aws_security_group_rule.security-group-flask["flask"] will be created
  + resource "aws_security_group_rule" "security-group-flask" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = -1
      + id                       = (known after apply)
      + protocol                 = "-1"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = -1
      + type                     = "ingress"
    }

  # aws_security_group_rule.security-group-flask["http"] will be created
  + resource "aws_security_group_rule" "security-group-flask" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 80
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 80
      + type                     = "egress"
    }

  # aws_security_group_rule.security-group-flask["https"] will be created
  + resource "aws_security_group_rule" "security-group-flask" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 443
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 443
      + type                     = "egress"
    }

Plan: 18 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_ecs_cluster.common-ecs-cluster-flask: Creating...
aws_security_group.security_group: Creating...
aws_iam_role.iam-role-flask: Creating...
aws_lb_target_group.alb-tg-flask: Creating...
aws_lb_target_group.alb-tg-flask: Creation complete after 1s [id=arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:targetgroup/flask-web-alb-tg/52b8015027deedfa]
aws_iam_role.iam-role-flask: Creation complete after 2s [id=flask-service-role20230313031014607100000002]
aws_iam_role_policy_attachment.flask-permissions["arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"]: Creating...
aws_iam_instance_profile.iam-instance-profile-assume-role-flask: Creating...
aws_iam_role_policy.flask-permissions: Creating...
aws_ecs_task_definition.common-ecs-task-definition-flask: Creating...
aws_ecs_task_definition.common-ecs-task-definition-flask: Creation complete after 0s [id=flask-service]
aws_security_group.security_group: Creation complete after 2s [id=sg-0826a24bf5850b385]
aws_security_group_rule.security-group-flask["https"]: Creating...
aws_security_group_rule.security-group-flask["flask"]: Creating...
aws_security_group_rule.security-group-flask["http"]: Creating...
aws_lb.lb-flask: Creating...
aws_iam_role_policy_attachment.flask-permissions["arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"]: Creation complete after 1s [id=flask-service-role20230313031014607100000002-20230313031017179000000003]
aws_security_group_rule.security-group-flask["https"]: Creation complete after 1s [id=sgrule-2894222127]
aws_iam_role_policy.flask-permissions: Creation complete after 1s [id=flask-service-role20230313031014607100000002:flask-service-role]
aws_iam_instance_profile.iam-instance-profile-assume-role-flask: Creation complete after 1s [id=flask-service-role]
aws_launch_template.aws-launch-template-flask: Creating...
aws_security_group_rule.security-group-flask["flask"]: Creation complete after 1s [id=sgrule-3448814527]
aws_launch_template.aws-launch-template-flask: Creation complete after 1s [id=lt-0e62fa3e028334e5d]
aws_autoscaling_group.asg-flask: Creating...
aws_security_group_rule.security-group-flask["http"]: Creation complete after 2s [id=sgrule-3970281291]
aws_ecs_cluster.common-ecs-cluster-flask: Still creating... [10s elapsed]
aws_ecs_cluster.common-ecs-cluster-flask: Creation complete after 11s [id=arn:aws:ecs:ap-southeast-1:123456789012:cluster/flask-cluster]
aws_cloudwatch_log_group.cloudwatch-log-group-flask: Creating...
aws_ecs_service.service-ec2-no-scaling: Creating...
aws_cloudwatch_log_group.cloudwatch-log-group-flask: Creation complete after 1s [id=flask-cluster]
aws_lb.lb-flask: Still creating... [10s elapsed]
aws_autoscaling_group.asg-flask: Still creating... [10s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [10s elapsed]
aws_lb.lb-flask: Still creating... [20s elapsed]
aws_autoscaling_group.asg-flask: Still creating... [20s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [20s elapsed]
aws_lb.lb-flask: Still creating... [30s elapsed]
aws_autoscaling_group.asg-flask: Still creating... [30s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [30s elapsed]
aws_lb.lb-flask: Still creating... [40s elapsed]
aws_autoscaling_group.asg-flask: Still creating... [40s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [40s elapsed]
aws_lb.lb-flask: Still creating... [50s elapsed]
aws_autoscaling_group.asg-flask: Still creating... [50s elapsed]
aws_autoscaling_group.asg-flask: Creation complete after 53s [id=flask-service]
aws_ecs_service.service-ec2-no-scaling: Still creating... [50s elapsed]
aws_lb.lb-flask: Still creating... [1m0s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [1m0s elapsed]
aws_lb.lb-flask: Still creating... [1m10s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [1m10s elapsed]
aws_lb.lb-flask: Still creating... [1m20s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [1m20s elapsed]
aws_lb.lb-flask: Still creating... [1m30s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [1m30s elapsed]
aws_lb.lb-flask: Still creating... [1m40s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [1m40s elapsed]
aws_lb.lb-flask: Still creating... [1m50s elapsed]
aws_ecs_service.service-ec2-no-scaling: Still creating... [1m50s elapsed]
aws_lb.lb-flask: Still creating... [2m0s elapsed]
aws_lb.lb-flask: Creation complete after 2m2s [id=arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:loadbalancer/app/flask-web-lb/1bfb8192e6df13f2]
aws_lb_listener.lb-listener-flask: Creating...
aws_lb_listener.lb-listener-flask: Creation complete after 1s [id=arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:listener/app/flask-web-lb/1bfb8192e6df13f2/be6c53ae8e5d95d1]
aws_lb_listener_rule.lb-listener-rule-flask: Creating...
aws_lb_listener_rule.lb-listener-rule-flask: Creation complete after 0s [id=arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:listener-rule/app/flask-web-lb/1bfb8192e6df13f2/be6c53ae8e5d95d1/79a29ac15918c2a4]
aws_ecs_service.service-ec2-no-scaling: Creation complete after 1m54s [id=arn:aws:ecs:ap-southeast-1:123456789012:service/flask-cluster/flask-service]

Apply complete! Resources: 18 added, 0 changed, 0 destroyed.
---------- end to deploy 02-ecs ----------
root@777777777777:/data/demos/aws-demo/terraform/scripts#
```
</details>

Deployment succeeded. Find the dns name with the `aws elbv2 describe-load-balancers` command.
```
root@777777777777:/data/demos/aws-demo/terraform/scripts# aws elbv2 describe-load-balancers
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:loadbalancer/app/flask-web-lb/862eb2968059d2f9",
            "DNSName": "flask-web-lb-1111122222.ap-southeast-1.elb.amazonaws.com",
            "CanonicalHostedZoneId": "Z1LMS91P8CMLE5",
            "CreatedTime": "2023-03-09T12:13:46.690000+00:00",
            "LoadBalancerName": "flask-web-lb",
            "Scheme": "internet-facing",
            "VpcId": "vpc-b68844d0",
            "State": {
                "Code": "active"
            },
            "Type": "application",
            "AvailabilityZones": [
                {
                    "ZoneName": "ap-southeast-1c",
                    "SubnetId": "subnet-114d2148",
                    "LoadBalancerAddresses": []
                },
                {
                    "ZoneName": "ap-southeast-1b",
                    "SubnetId": "subnet-1c60d17a",
                    "LoadBalancerAddresses": []
                },
                {
                    "ZoneName": "ap-southeast-1a",
                    "SubnetId": "subnet-ccb61784",
                    "LoadBalancerAddresses": []
                }
            ],
            "SecurityGroups": [
                "sg-0aa5606ac524436de"
            ],
            "IpAddressType": "ipv4"
        }
    ]
}
root@777777777777:/data/demos/aws-demo/terraform/scripts#
```
## Verify the service
- Try health check
```
curl --location --request GET 'http://flask-web-lb-1111122222.ap-southeast-1.elb.amazonaws.com/'
```
Response as below
```
Welcome!
```
- Create S3 bucket
```
curl --location --request POST 'http://flask-web-lb-1111122222.ap-southeast-1.elb.amazonaws.com/s3-bucket/my-2023-test-bucket'
```
Response as below
```
{
    "Location": "/my-2023-test-bucket",
    "ResponseMetadata": {
        "HTTPHeaders": {
            "content-length": "0",
            "date": "Mon, 13 Mar 2023 03:30:44 GMT",
            "location": "/my-2023-test-bucket",
            "server": "AmazonS3",
            "x-amz-id-2": "ArFI2RLzbGgXAyoZmnUJsxzR9pVBUBo75vZFGdyAgtPpzaN584y/Mb9NtSF/Heh4lQ/33FylfV0=",
            "x-amz-request-id": "AAP85VCQB793SBX2"
        },
        "HTTPStatusCode": 200,
        "HostId": "ArFI2RLzbGgXAyoZmnUJsxzR9pVBUBo75vZFGdyAgtPpzaN584y/Mb9NtSF/Heh4lQ/33FylfV0=",
        "RequestId": "AAP85VCQB793SBX2",
        "RetryAttempts": 0
    }
}
```
Log in cloudwatch as below
```
2023-03-13 03:30:42,169 INFO service_name="uwsgi_file__app_app" class="uwsgi_file__app_app" event_description="create my-2023-test-bucket"
```
- Check the S3 bucket
```
curl --location --request GET 'http://flask-web-lb-1111122222.ap-southeast-1.elb.amazonaws.com/s3-bucket/my-2023-test-bucket'
```
Response as below
```
{
    "LocationConstraint": null,
    "ResponseMetadata": {
        "HTTPHeaders": {
            "content-type": "application/xml",
            "date": "Mon, 13 Mar 2023 03:31:32 GMT",
            "server": "AmazonS3",
            "transfer-encoding": "chunked",
            "x-amz-id-2": "tK1p1XQ5b3kDZuQywkpmkJWnhH9d1c5EsztlRJZli14ZX0nEJIApSzTRfI6+JFPQ79bl6DVOPr8=",
            "x-amz-request-id": "1EQ8W9S6CQ9DQCE9"
        },
        "HTTPStatusCode": 200,
        "HostId": "tK1p1XQ5b3kDZuQywkpmkJWnhH9d1c5EsztlRJZli14ZX0nEJIApSzTRfI6+JFPQ79bl6DVOPr8=",
        "RequestId": "1EQ8W9S6CQ9DQCE9",
        "RetryAttempts": 0
    }
}
```
Log in cloudwatch as below
```
2023-03-13 03:31:30,483 INFO service_name="uwsgi_file__app_app" class="uwsgi_file__app_app" event_description="get my-2023-test-bucket"
```
- Delete S3 bucket
```
curl --location --request DELETE 'http://flask-web-lb-1111122222.ap-southeast-1.elb.amazonaws.com/s3-bucket/my-2023-test-bucket'
```
Response as below
```
{
    "ResponseMetadata": {
        "HTTPHeaders": {
            "date": "Mon, 13 Mar 2023 03:32:13 GMT",
            "server": "AmazonS3",
            "x-amz-id-2": "3yJNYx98hs78cOQeEvRYoDICbuYBNY0xT8UQfoTkQNOwvUswujn/1MJA2b51LE3f5dH+BoMtERI=",
            "x-amz-request-id": "1KR1M587251AMQS8"
        },
        "HTTPStatusCode": 204,
        "HostId": "3yJNYx98hs78cOQeEvRYoDICbuYBNY0xT8UQfoTkQNOwvUswujn/1MJA2b51LE3f5dH+BoMtERI=",
        "RequestId": "1KR1M587251AMQS8",
        "RetryAttempts": 0
    }
}
```
Log in cloudwatch as below
```
2023-03-13 03:32:12,051 INFO service_name="uwsgi_file__app_app" class="uwsgi_file__app_app" event_description="delete my-2023-test-bucket"
```
