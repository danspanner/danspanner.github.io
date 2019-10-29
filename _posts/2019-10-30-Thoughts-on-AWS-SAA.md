---
layout: post
title: Thoughts on the AWS Solutions Architect Associate cert
---

After having recently taken the AWS Solutions Architect Associate exam, I though it best to add a few lines here to share my experience and takeways.

This is the first certificate I've earned, so a bit of a novel experience after 10 or so years of being a 'Professional Computer Guy'.

Since some of the other staff in my office are preparing for the same exam, I had this advice to offer-

* The A Cloud Guru course is fantastic, and I highly recommend it. I got the course through Udemy (heavily discounted of course) and it gives a good overview of what you'll need for the exam, aimed at beginners. That being said...

* As good as the A Cloud Guru course is, it isn't enough for the exam. The course instructor, Ryan Kroonenburg reiterates this throughout the course- **read the FAQs**.

* The FAQs are not enough to get through the course either- if you're like me, you'll learn by doing. Watching someone with years of experience blitz through setting up a VPC is good and all, what's much better is **doing it yourself**.

* When you're doing the practical stuff, remember the fundamentals- VPCs are just networks, the rules of networks apply. EC2 is just VMs, the usual caveats of VMs apply. Route 53 is just DNS, the rules of DNS still apply. **If you don't know the fundamentals going in, you're gonna have a bad time**.

* Although this isn't meant as a study guide...take notes. Then type the notes. Then read the notes. Rinse and repeat.

* Don't be afraid to spin up bits and pieces as you go. A good 90% of the services covered in the A Cloud Guru course are free-tier, meaning there is either a free use compute or storage limit per month for that service, or new accounts can trial the service with no cost associated- **spin up those services**.

* The aim of the A Cloud Guru course (and the certification) is to get you to the point where you can *Architect* a *Solution* using AWS and its services. Keep that in mind- if you are getting this certification to further your career or as a requirement of your current role, you will be expected to roll with the punches in building solutions.

Last but not least, here is a list of AWS services that are covered in the exam. I'm not saying you need to be utterly conversant in *all* these services, but you definitely need to know which is which and what they do. There are a couple of questions I came across where they have obviously thrown in one or two similarly named services to try to catch you out (for example, SNS and SQS, or CloudTrail and CloudWatch)

* Compute
	* [EC2](https://aws.amazon.com/ec2/faqs/)
    * [Lambda](https://aws.amazon.com/lambda/faqs/)
	* [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/faqs/)
* Storage
	* [S3](https://aws.amazon.com/s3/faqs/)
    * [EFS](https://aws.amazon.com/efs/faq/)
    * [Glacier](https://aws.amazon.com/glacier/faqs/)
* Database
    * [RDS](https://aws.amazon.com/rds/faqs/)
    * [DynamoDB](https://aws.amazon.com/dynamodb/faqs/)
    * [ElastiCache](https://aws.amazon.com/elasticache/faqs/)
    * [Redshift](https://aws.amazon.com/redshift/faqs/)
* Network and Content Delivery
    * [VPC](https://aws.amazon.com/vpc/faqs/)
    * [Cloudfront](https://aws.amazon.com/cloudfront/faqs/)
    * [Route 53](https://aws.amazon.com/route53/faqs/)
    * [API Gateway](https://aws.amazon.com/api-gateway/faqs/)
    * [Direct Connect](https://aws.amazon.com/directconnect/faqs/)
* Management and Governance
    * [CloudWatch](https://aws.amazon.com/cloudwatch/faqs/)
    * [CloudTrail](https://aws.amazon.com/cloudtrail/faqs/)
* Security, Identity and Compliance
    * [IAM](https://aws.amazon.com/iam/faqs/)
* Application Integration
    * [Simple Notification Service](https://aws.amazon.com/sns/faqs/)
    * [Simple Queue Service](https://aws.amazon.com/sqs/faqs/)
