---
layout: post
title: Firebase on AWS: GCP and AWS playing together
---

My experience with cloud always focused on Amazon AWS, not by preference, but by its widespread use in Cape Town. The best tool for the specific job will prevail, no matter your preference to vendors. Joining a insuretech startup I had my first exposure to GCP through Google's Firebase. Its a interesting and quick PaaS for rapid development of a Proof of concept. As the POC moved closer to production phase, the need for a cloud provider to run a python API on a Virtual Machine lead me back to AWS.

In some ways we might call ourselves a hybrid cloud, but Amazon's AWS was familiar and I found some good ways to integrate the two. My authentication system used JWTs (Jason Web Tokens) and I was happy to discover I could integrate it with Google Firebase's JWT. Lambda and AWS SES was the initial gravitional pull, which lead to more AWS services consumed.

A problem that plagued me was the "firebase-admin-python" sdk/library needed elevated privilages to integrate into Google's Firebase. I say plagued, though from a security perspective it great. My problem was around the fact that the library required two JSON files or dicts to authenticate during its initialisation. Initially I would use .gitignore in the project and securely transfer the files via ssh, but this would be a annoyance with devops tools and at scale. I looked at a couple of ideas, then one afternoon just went for it. I made a wild assumption writing this, that for the most part this would be a niche problem, but I have found a question or two on stackoverflow. 

This solution requires the following building blocks: AWS KMS, AWS S3, AWS IAM role, AWS EC2/Lambda and boto3 (I am a python developer after all.):

Step 1:
Create a KMS key in the same region that your S3 bucket will live

Step 2:
Create a S3 bucket with encryption with the previously mentioned

Step 3:
Make sure the IAM role (with appropriate permissions) is attached to your ec2 instance or lamda function

Step 4: 
Use the below code with pip install firebase-admin

```
import os
import firebase_admin
from firebase_admin import credentials
import boto3
from settings.local_settings import AWS_REGION, ENVIRONMENT
import json

firebase_config_file = 'app-admin-config-{}.json'.format(ENVIRONMENT)
firebase_admin_creds_file = 'app-admin-sdk-{}.json'.format(ENVIRONMENT)

current_dir = os.path.abspath(os.path.dirname(__file__))
files = [f for f in os.listdir(current_dir) if os.path.isfile(f)]

if firebase_config_file not in files and firebase_admin_creds_file not in files:
    s3 = boto3.resource('s3', region_name=AWS_REGION)
    bucket = s3.Bucket('app-s3-secrets')

    firebase_config = json.loads(
        bucket.Object('app-admin-config-{}.json'.format(ENVIRONMENT)).get()['Body'].read())
    firebase_admin_creds = json.loads(
        bucket.Object('app-admin-sdk-{}.json'.format(ENVIRONMENT)).get()['Body'].read().decode())

class Firebase:
    @staticmethod
    def get_connection():
        cred = credentials.Certificate(firebase_admin_creds)
        return firebase_admin.initialize_app(cred, firebase_config)


app = Firebase.get_connection()
```

<!-- ![_config.yml]({{ site.baseurl }}/images/config.png) -->
