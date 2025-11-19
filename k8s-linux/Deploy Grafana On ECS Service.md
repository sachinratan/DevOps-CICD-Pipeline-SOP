## This guide will provide the overview of 'To deploy the Grafana dashboard with EFS volume mount and access it over the ALB DNS address

#### Created EFS file system.

#### Create the Dockerfile for Grafana to change the "/var/lib/grafana" directory ownership and permission.
```
$ cat << EOF >> Dockerfile.grafana
FROM grafana/grafana:latest
USER root
RUN mkdir -p /var/lib/grafana && \
        chown -R grafana:root /var/lib/grafana && \
        chmod -R 755 /var/lib/grafana
USER grafana
EOF
```

#### Built and pushed the image to the ECR registry.
```
$ aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACC_NO>.dkr.ecr.us-east-1.amazonaws.com

$ docker build -t grafana .

$ docker tag grafana:latest <AWS_ACC_NO>.dkr.ecr.us-east-1.amazonaws.com/grafana:latest

$ docker push <AWS_ACC_NO>.dkr.ecr.us-east-1.amazonaws.com/grafana:latest
```

#### Created the task definition and configured the EFS volume.
```
{
    "family": "grafana-task",
    "containerDefinitions": [
        {
            "name": "grafana",
            "image": "<AWS_ACC_NO>.dkr.ecr.us-east-1.amazonaws.com/grafana:latest",
            "cpu": 0,
            "portMappings": [
                {
                    "containerPort": 3000,
                    "hostPort": 3000,
                    "protocol": "tcp"
                }
            ],
            "essential": true,
            "environment": [],
            "mountPoints": [
                {
                    "sourceVolume": "grafana-vol",
                    "containerPath": "/var/lib/grafana"
                }
            ],
            "volumesFrom": [],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "/ecs/grafana",
                    "awslogs-create-group": "true",
                    "awslogs-region": "us-east-1",
                    "awslogs-stream-prefix": "ecs"
                }
            },
            "systemControls": []
        }
    ],
    "taskRoleArn": "arn:aws:iam::<AWS_ACC_NO>:role/ecs_task_role",
    "executionRoleArn": "arn:aws:iam::<AWS_ACC_NO>:role/ecsTaskExecutionRole",
    "networkMode": "awsvpc",
    "volumes": [
        {
            "name": "grafana-vol",
            "efsVolumeConfiguration": {
                "fileSystemId": "fs-0338xxxxfe15",
                "rootDirectory": "/",
                "transitEncryption": "ENABLED"
            }
        }
    ],
    "placementConstraints": [],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "1024",
    "memory": "2048",
    "enableFaultInjection": false
}
```

#### Deployed the ECS service out of the above task definition.
```
aws ecs create-service --cluster ecs-test-cluster --service-name grafana-efs-test-001 --task-definition grafan-task:3 --desired-count 1 --launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[subnet-12344321],securityGroups=[sg-12344321]}" 
```
#### Associated the ALB with the ECS service.
```
$ aws ecs update-service --cluster ecs-test-cluster --region us-east-1 --service grafana-efs-test-001 --load-balancers '
[{
    "containerName": "grafana",
    "containerPort": 3000,
    "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:<AWS_ACC_NO>:targetgroup/ecs-test-lb-tg/419b55e41b0be1c6"
}]'
```

#### Lastly, access the Grafana dashboard over the ALB DNS address.
