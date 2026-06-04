# S3 Static Website Deployment using GitHub + CodeBuild

## Project Overview

This project demonstrates a simple CI/CD pipeline for deploying a static website using:

* GitHub Repository
* GitHub Webhook
* AWS CodeBuild
* Amazon S3
* S3 Static Website Hosting

Whenever code is pushed to GitHub, CodeBuild is automatically triggered through a GitHub webhook. CodeBuild then uploads the latest website files to an S3 bucket, which serves the website through S3 Static Website Hosting.

---

# Architecture

```text
Developer
    │
    ▼
GitHub Repository
    │
    │ Push Code
    ▼
GitHub Webhook
    │
    ▼
AWS CodeBuild
    │
    │ Build & Deploy
    ▼
Amazon S3 Bucket
    │
    ▼
Static Website Hosting
    │
    ▼
Browser
```

---

# BuildSpec Configuration

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo "Nothing to install"
      - echo "This project is a static website"

  build:
    commands:
      - echo "Preparing static website files"

  post_build:
    commands:
      - echo "Uploading website files to S3"
      - aws s3 cp src/main/webapp/ s3://my-portfolio-webapp-bucket/ --recursive
```

---

# Application Structure

```text
src/main/webapp/
├── index.html
├── css/
├── js/
└── images/
```

---

# What CodeBuild Does

CodeBuild will:

```text
1. Clone the GitHub repository.

2. Execute the install phase.

3. Execute the build phase.

4. Collect website files from:

   src/main/webapp

   Examples:
   - index.html
   - css/
   - js/
   - images/

5. Execute the post_build phase.

6. Upload files to S3 using:

   aws s3 cp src/main/webapp/ \
   s3://my-portfolio-webapp-bucket/ \
   --recursive

7. Store files inside:

   my-portfolio-webapp-bucket
   ├── index.html
   ├── css/
   ├── js/
   └── images/

8. S3 Static Website Hosting serves the files.

9. Users access the website through a browser.
```

---

# Build Lifecycle

## Install Phase

```yaml
install:
  commands:
    - echo "Nothing to install"
```

Purpose:

```text
Prepare build environment.
```

Since this project is a static website:

```text
No Maven
No Gradle
No Spring Boot Packaging
No NodeJS Build
No Compilation
```

---

## Build Phase

```yaml
build:
  commands:
    - echo "Preparing static website files"
```

Purpose:

```text
Validate and prepare website files.
```

---

## Post Build Phase

```yaml
post_build:
  commands:
    - aws s3 cp src/main/webapp/ s3://my-portfolio-webapp-bucket/ --recursive
```

Purpose:

```text
Deploy website files to S3.
```

Example:

```text
src/main/webapp/index.html
      │
      ▼
s3://my-portfolio-webapp-bucket/index.html
```

---

# Automatic Build Trigger

GitHub Webhook is configured.

Whenever code is pushed:

```text
git push
    │
    ▼
GitHub Webhook
    │
    ▼
AWS CodeBuild Triggered
```

No manual build execution is required.

---

# S3 Website Bucket

Bucket Name:

```text
my-portfolio-webapp-bucket
```

Example contents:

```text
my-portfolio-webapp-bucket
├── index.html
├── css/style.css
├── js/app.js
└── images/logo.png
```

---

# Why Do We Need IAM Permissions?

There are two actors interacting with S3.

---

# Actor 1 - AWS CodeBuild

CodeBuild uploads website files to S3.

Flow:

```text
CodeBuild
    │
    ▼
Upload Files
    │
    ▼
S3 Bucket
```

Required Permissions:

```text
s3:PutObject
s3:GetObject
s3:ListBucket
```

These permissions are granted through the:

```text
CodeBuild Service Role
```

Example:

```text
codebuild-my-portfolio-service-role
```

---

# CodeBuild IAM Policy

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:GetObject",
    "s3:ListBucket"
  ],
  "Resource": [
    "arn:aws:s3:::my-portfolio-webapp-bucket",
    "arn:aws:s3:::my-portfolio-webapp-bucket/*"
  ]
}
```

---

# Why Are There Two Resources?

## Bucket ARN

```text
arn:aws:s3:::my-portfolio-webapp-bucket
```

Represents:

```text
The Bucket Itself
```

Used for:

```text
s3:ListBucket
```

Example:

```text
Show me all files inside the bucket.
```

---

## Object ARN

```text
arn:aws:s3:::my-portfolio-webapp-bucket/*
```

Represents:

```text
All files inside the bucket.
```

Examples:

```text
index.html
css/style.css
js/app.js
images/logo.png
```

Used for:

```text
s3:GetObject
s3:PutObject
s3:DeleteObject
```

---

# Easy Way To Remember

```text
Bucket
=
arn:aws:s3:::bucket-name

Objects
=
arn:aws:s3:::bucket-name/*
```

Examples:

```text
s3:ListBucket  → Bucket ARN

s3:GetObject   → Object ARN

s3:PutObject   → Object ARN

s3:DeleteObject → Object ARN
```

---

# Why Do We Need a Bucket Policy?

CodeBuild uploads files successfully because it authenticates using an IAM Role.

However, website visitors are different.

---

# Actor 2 - Website Visitor

When someone opens the website:

```text
Browser
    │
    ▼
Request index.html
    │
    ▼
S3 Bucket
```

The browser is not authenticated with AWS.

Without a bucket policy:

```text
Browser
    │
    ▼
S3 Bucket
    │
    ▼
403 Access Denied
```

because anonymous users cannot read S3 objects.

---

# Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-portfolio-webapp-bucket/*"
    }
  ]
}
```

---

# Bucket Policy Explained

## Principal

```json
"Principal": "*"
```

Meaning:

```text
Anyone on the internet.
```

Examples:

```text
Chrome Browser
Firefox Browser
Edge Browser
Mobile Browser
Any Internet User
```

---

## Action

```json
"Action": "s3:GetObject"
```

Meaning:

```text
Read File
Download File
```

Examples:

```text
index.html
style.css
app.js
logo.png
```

---

## Resource

```json
"Resource": "arn:aws:s3:::my-portfolio-webapp-bucket/*"
```

Meaning:

```text
All objects inside the bucket.
```

Examples:

```text
index.html
css/style.css
js/app.js
images/logo.png
```

---

# Why Doesn't The Bucket Policy Mention CodeBuild?

Because CodeBuild authenticates using IAM.

Flow:

```text
CodeBuild
    │
    ▼
IAM Role
    │
    ▼
S3
```

AWS already knows who CodeBuild is.

Therefore:

```text
No public access is required for CodeBuild.
```

---

# Permission Summary

## CodeBuild Needs

```text
s3:PutObject
s3:GetObject
s3:ListBucket
```

Purpose:

```text
Upload and manage website files.
```

---

## Browser Needs

```text
s3:GetObject
```

Purpose:

```text
Download website files.
```

---

# Final Deployment Flow

```text
GitHub
   │
   │ Push Code
   ▼
GitHub Webhook
   │
   ▼
AWS CodeBuild
   │
   │ Upload Files To S3
   ▼
my-portfolio-webapp-bucket
   │
   │ Public Read Access
   ▼
S3 Static Website Hosting
   │
   ▼
Users Access Website
```

---

# Key Learning Outcomes

```text
GitHub Webhook
→ Automatically triggers CodeBuild

CodeBuild
→ Builds and deploys website files

IAM Role
→ Allows CodeBuild to access S3

Bucket Policy
→ Allows browsers to access website files

S3 Static Website Hosting
→ Serves website content to users

AWS CLI
→ Deploys files using:

aws s3 cp --recursive
```
