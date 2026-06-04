# S3 Static Website Deployment using GitHub + CodeBuild

## Project Overview

This project demonstrates a simple CI/CD pipeline for deploying a static website using:

* GitHub Repository
* GitHub Webhook
* AWS CodeBuild
* Amazon S3 Static Website Hosting

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

artifacts:
  files:
    - '**/*'
  base-directory: 'src/main/webapp'
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

# CI/CD Flow

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
    │ Reads buildspec.yml
    │
    │ Collects files from:
    │ src/main/webapp
    ▼
Build Artifact
    │
    ▼
S3 Website Bucket
    │
    ▼
Static Website Hosting
    │
    ▼
Browser
```

---

# What Happens During Build?

## Install Phase

```text
Nothing to install.
```

This project contains only static files:

```text
HTML
CSS
JavaScript
Images
```

No compilation is required.

```text
No Maven
No Gradle
No Spring Boot Packaging
No NodeJS Build
```

---

## Build Phase

```text
Preparing static website files
```

CodeBuild simply prepares the static website assets.

---

# Artifacts Section

```yaml
artifacts:
  files:
    - '**/*'
  base-directory: 'src/main/webapp'
```

Meaning:

```text
Take everything from:

src/main/webapp

including:

index.html
css/
js/
images/
```

and package them into the build artifact.

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

The bucket stores:

```text
index.html
css/style.css
js/app.js
images/logo.png
```

---

# Why Do We Need a Bucket Policy?

There are two different actors involved.

---

# Actor 1: CodeBuild

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

Required permissions:

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

# Actor 2: Website Visitor

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

Without permission:

```text
Browser
    │
    ▼
S3 Bucket
    │
    ▼
403 Access Denied
```

Therefore we must allow:

```text
s3:GetObject
```

for public users.

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
Anyone
```

Examples:

```text
Chrome Browser
Firefox Browser
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
All objects inside the bucket
```

Examples:

```text
index.html
css/style.css
js/app.js
images/logo.png
```

---

# Why Doesn't the Bucket Policy Mention CodeBuild?

Because CodeBuild uses IAM Roles.

CodeBuild authenticates as:

```text
codebuild-my-portfolio-service-role
```

AWS already knows who CodeBuild is.

Therefore:

```text
CodeBuild
    ▼
IAM Role
    ▼
S3
```

No public bucket access is required.

---

# CodeBuild IAM Policy Example

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

# Why Two Resources?

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
Show me all files inside the bucket
```

---

## Object ARN

```text
arn:aws:s3:::my-portfolio-webapp-bucket/*
```

Represents:

```text
Files Inside The Bucket
```

Examples:

```text
index.html
style.css
app.js
logo.png
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
ListBucket
→ Bucket ARN

GetObject
→ Object ARN

PutObject
→ Object ARN

DeleteObject
→ Object ARN
```

---

# Permission Summary

## CodeBuild

Needs:

```text
s3:PutObject
s3:GetObject
s3:ListBucket
```

Purpose:

```text
Upload and manage website files
```

---

## Browser

Needs:

```text
s3:GetObject
```

Purpose:

```text
Download website files
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
   │ Build Artifact
   ▼
S3 Website Bucket
   │
   │ Public Read Access
   ▼
Static Website Hosting
   │
   ▼
Users Access Website
```
