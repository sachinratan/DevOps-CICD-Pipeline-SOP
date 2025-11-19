## Create the application load balancer, deployed the ECS service with load balancer configuration. Following the ECS service deployment, update the ECS service to replace the existing load balancer target group with a newly created target group

#### Created Application Load Balancer (ecs-svc-lb-test) with single target group (targetgroup/ecs-svc-lb-tg-test).

#### Created ECS service with Load Balancer configuration.
```
$ aws ecs create-service --cluster ecs-test-cluster --service-name ecs-svc-lb-test --task-definition nginx-td:2 --desired-count 1 --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:us-east-1:<AWS_ACCOUNT_NO>:targetgroup/ecs-svc-lb-tg-test/fb0cxxxxxxx80d4,containerName=nginx,containerPort=80 --region us-east-1
```
#### Verified the created ECS service has provided Load Balancer Target group configuration.
```
$ aws ecs describe-services --cluster ecs-test-cluster --service ecs-svc-lb-test --query services[].loadBalancers[]
[
    {
        "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:<AWS_ACCOUNT_NO>:targetgroup/ecs-svc-lb-tg-test/fb0cxxxxxxx80d4",
        "containerName": "nginx",
        "containerPort": 80
    }
]
```

#### Next, manually created new target group (targetgroup/ecs-svc-lb-tg-test-001) and associted with Load Balancer created in step 1.

#### Updated the ECS service to replace the existing target group with newly created target group.
```
$ aws ecs update-service --cluster ecs-test-cluster --service ecs-svc-lb-test --load-balancers targetGroupArn=arn:aws:elasticloadbalancing:us-east-1:<AWS_ACCOUNT_NO>:targetgroup/ecs-svc-lb-tg-test-001/2485xxxxxx4610,containerName=nginx,containerPort=80 --region us-east-1
```

Note: In order to Register/Update the Load Balancer configuration with ECS service either Load Balancer name or Target Group ARN is required.

#### Lastly, verified the ECS service has registered the newly created target group.
```
$ aws ecs describe-services --cluster ecs-test-cluster --service ecs-svc-lb-test --query services[].loadBalancers[]
[
    {
        "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:<AWS_ACCOUNT_NO>:targetgroup/ecs-svc-lb-tg-test-001/2485xxxxxx4610",
        "containerName": "nginx",
        "containerPort": 80
    }
]
```
