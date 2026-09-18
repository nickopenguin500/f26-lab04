# Deployment Evidence

Fill this in as you go. Paste real output, not descriptions of output. A TA reads this
file with you at recitation.

## 1. Deployed URL and instance id

<!-- 
-------------------------------------------------------------------------
|                            DescribeStacks                             |
+------------+----------------------------------------------------------+
|  InstanceId|  i-011441c11fdf5177f                                     |
|  ServiceUrl|  http://ec2-34-229-96-188.compute-1.amazonaws.com:8080   |
+------------+----------------------------------------------------------+
------------------------------------------------------------------------
|                            DescribeStacks                            |
+------------+---------------------------------------------------------+
|  InstanceId|  i-02fb7e369d67332ef                                    |
|  ServiceUrl|  http://ec2-3-89-119-122.compute-1.amazonaws.com:8080   |
+------------+---------------------------------------------------------+
------------------------------------------------------------------------
|                            DescribeStacks                            |
+------------+---------------------------------------------------------+
|  InstanceId|  i-014c1e1176ae5d5f0                                    |
|  ServiceUrl|  http://ec2-54-91-83-124.compute-1.amazonaws.com:8080   |
+------------+---------------------------------------------------------+
-->

## 2. External health check

Run the check from your own machine, not from the instance. Paste the command and the
response.

```
curl http://ec2-34-229-96-188.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}%               

```

## 3. What the template created

Three or four sentences, your own words. What compute, what network access, and what
glue made the service start.

The template created a t3.micro EC2 instance running Amazon Linux 2023 to serve as the compute host for the application. It configured a Security Group to provide network access by opening port 8080 for the service's external health check and port 22 for SSH fallback access from the internet. Finally, it used a UserData bash script as the "glue" to automatically install Docker, pull the lab's container image, and start the service on boot while mapping the network ports.

## 4. Scenario 2 diagnosis

**The failing curl** (command and output):

```
curl http://ec2-3-89-119-122.compute-1.amazonaws.com:8080/api/health
curl: (28) Failed to connect to ec2-3-89-119-122.compute-1.amazonaws.com port 8080 after 75028 ms: Couldn't connect to server

```

**The log line that told you what was wrong:**

```
lab04-service listening on 9090
```

**What was wrong, and the fix you applied:**

The log line lab04-service listening on 9090 revealed the application was listening on port 9090, while docker ps showed the container was only configured to receive traffic on port 8080. I fixed this by deleting the stack and redeploying with the healthy parameters (infra/params-healthy.json) so the application listens on the correct port.

**The healthy curl after the fix:**

```
curl http://ec2-54-91-83-124.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}%  
```

## 5. Teardown proof

Paste the delete output, or describe the console evidence that the resources are gone.

```
aws cloudformation describe-stacks --stack-name lab04-service

aws: [ERROR]: An error occurred (ValidationError) when calling the DescribeStacks operation: Stack with id lab04-service does not exist
```
