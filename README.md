## Overview
This repository demonstrates how to accelerate your static and dynamic web content in a secure, scalable, and repeatable way using Amazon CloudFront, Amazon S3, Amazon EC2, Amazon VPC, and AWS CloudFormation services.

- [What is Amazon CloudFront?](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [What is Amazon EC2?](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [What is Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [What is AWS CloudFormation?](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)

## High-Level Architecture
The demonstration application can be divided into two parts: static content hosted on S3 and dynamic content hosted on EC2. The static webpage is backed by a simple Express.js application that listens to HTTP requests and sends back a JSON response that includes the headers received in the request. All content is distributed using CloudFront to lower the end-user latency (the time it takes to load the first byte of the file) and achieve higher data transfer rates.

![Screenshot](architecture.jpg)

## AWS Design Features
The design includes the follwing set of features: 
- The CloudFront distribution is configured to require HTTPS for communication between viewers and CloudFront. To avoid information exposure, CloudFront is configured with a custom error response behavior and `index.html` as the default root object. 
- The S3 bucket is configured with Public Access Block and has its bucket policy restrict to Origin Access Identity for CloudFront.
- The Application Load Balancer is configured across 3 Availability Zones with health checks on target groups. The Security Group of the Application Load Balance is restricted to AWS-managed prefix list for Amazon CloudFront using CloudFormation custom resources.
- The Auto Scaling Group is configured across 3 Availability Zones with a minimum size of 3 instances and maximum size of 9 instances. The Auto Scaling group triggers scaling when the average outbound network traffic from each instance is higher than 6 MiB or lower than 2 MiB over a period of five minutes. 
- The EC2 instance type is EC2 M6g.large powered by Arm-based AWS Graviton2 processors which delivers up to 40% better price performance. EC2 user data is used to install and configure Express.js at instance launch time. The EC2 Security Group is restricted to port 80 and private IP address range.

![Screenshot](design.jpg)

## Deployment
1. Use the provided `cloudformation-template.yaml` file to deploy a CloudFormation stack in your AWS region of choice. Make sure the latest AWS features used in template, such as EC2 M6g instances and CloudFront managed prefix lists, are supported in the selected AWS region.
2. Once the CloudFormation stack is created successfully, click on the **Outputs** tab which enables you to get access to information about resources within the stack.
3. Go to the S3 console to identify the S3 bucket which should store your static content. Upload the files provided in the `content` directory to the S3 bucket.
4. Go to the EC2 console to check if the EC2 instances created by CloudFormation finished initializing and entered the running state.  
5. Click on the CloudFront distribution domain name to access all content of your website. The static content is avaialble under `/index.html` while the dynamic content is available under `/api`.  

## Test
To test if the distribution is ready to be used in your region, you can use the `nslookup xxxxxxxxxxxx.cloudfront.net` command to lookup the CloudFront distribution domain name. You should be able to observe that CloudFront returns multiple IPs for each DNS query.

To check the average start transfer time of the application endpoint, you can use the commands below:
- Using Windows Command Prompt:
```
for /L %i in (1,1,10) do @echo %i && curl -s -o NUL --write-out "size_download: %{size_download} // time_total: %{time_total} // time_starttransfer: %{time_starttransfer}\n" https://xxxxxxxxxxxx.cloudfront.net
```
- For Linux bash command: 
```
for i in `seq 1 10`; do echo $i; curl -s -o /dev/null --write-out "size_download: %{size_download} // time_total: %{time_total} // time_starttransfer: %{time_starttransfer}\n" https://xxxxxxxxxxxx.cloudfront.net; done
```

## Cost
TDC
