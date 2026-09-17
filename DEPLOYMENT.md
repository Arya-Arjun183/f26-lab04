# Deployment Evidence

Fill this in as you go. Paste real output, not descriptions of output. A TA reads this
file with you at recitation.

## 1. Deployed URL and instance id

**InstanceId:** `i-09a51b6743438a067`
**ServiceUrl:** `http://ec2-13-218-97-55.compute-1.amazonaws.com:8080`

## 2. External health check

Run the check from your own machine, not from the instance. Paste the command and the
response.

```bash
curl http://ec2-3-89-241-242.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}
```

## 3. What the template created

Three or four sentences, your own words. What compute, what network access, and what
glue made the service start.

The CloudFormation template provisioned a t3.micro EC2 instance to serve as the compute layer. To control network access, it created a Security Group that allows inbound TCP traffic on port 8080 for the service and port 22 for SSH. As the glue, it utilized an EC2 UserData shell script to automatically install Docker and run the `lab04-service` container when the instance booted up.

## 4. Scenario 2 diagnosis

**The failing curl** (command and output):

```bash
curl http://ec2-13-218-97-55.compute-1.amazonaws.com:8080/api/health
curl: (7) Failed to connect to ec2-13-218-97-55.compute-1.amazonaws.com port 8080 after 32 ms: Couldn't connect to server
```

**The log line that told you what was wrong:**

```
lab04-service listening on 9090
```

**What was wrong, and the fix you applied:**

The container was configured via the `PortOverride` parameter to listen on port 9090 internally, but the Docker `-p` flag in UserData was still forwarding host port 8080 to container port 8080. Since the port mapping was mismatched, traffic couldn't reach the service. To fix it, I deleted the broken stack and deployed using the healthy parameters (where `PortOverride` is empty, defaulting the service to listen on 8080, matching the port mapping).

**The healthy curl after the fix:**

```bash
curl http://ec2-52-91-192-249.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}
```

## 5. Teardown proof

Paste the delete output, or describe the console evidence that the resources are gone.

```bash
$ aws cloudformation describe-stacks --stack-name lab04-service
An error occurred (ValidationError) when calling the DescribeStacks operation: Stack with id lab04-service does not exist
```
