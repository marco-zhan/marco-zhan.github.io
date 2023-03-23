---
title:       "The Cloud Resume Challenge, Part 1 - Introduction"
subtitle:    ""
description: "A quick intro on The Cloud Resume Challenge, the overall architecture and roadmap ..."
date:        2023-03-23
author: "Marco"
image:       ""
tags:        ["AWS", "Cloud"]
categories:  ["AWS"]
---

## Introduction
I recently completed [The Cloud Resume Challenge](https://cloudresumechallenge.dev/) and found it very interesting.
I bought my first ever domain, and explored some basic AWS services. This project definitely gets me started on AWS and
how powerful it is, so I encourage you to try too. In the next few blogs, I am going to write up how I completed this prject
and what I have learned.

## Why This Challenge
I have always been very keen in Cloud related technology ever since I first heard the term from university course.
Since then, I did some AWS intro courses on Udemy, bought myself a subscription on [A Cloud Guru](https://acloudguru.com/),
but I didn't manage to finish any of the courses. It's very boring just to learn it without doing any practical work, 
and I must say I have forgotten most of the things I have learned already.

So, I googled for AWS cloud project and found a nice and simple one, while touching a variety of AWS Services, 
[The Cloud Resume Challenge](https://cloudresumechallenge.dev/). I thought this project will get me started on some basic
AWS services.

## The Cloud Resume Challenge
The idea is very simple, host a small one-page resume on AWS with a view counter and some CI/CD pipeline. 

Acceptance Criteria (AWS services involved):
1. Write a resume webpage with React with some simple CSS
2. Host the webpage on AWS S3 (AWS S3)
3. Use HTTPS for accessing AWS S3 (AWS CloudFront)
4. Point a custom DNS to CloudFront distribution (AWS Route 53)
5. Have a view counter on the resume webpage
6. Store view count in a database for retrieving and updating (AWS DynamoDB)
7. Create an API to read from the database (AWS API Gateway, AWS Lambda)
8. Use IaC for creating and configuring all the resources mentioned above (AWS CloudFormation)
9. CI/CD (AWS CodePipeline)

## Overall Architecture
I looked at how many AWS services this challenge touches, all the configuration, and because I have touched a bit of IaC
in work and university, I told myself IaC is the right way to approach this task. I decided to use AWS CloudFormation 
because I have tried it before and of course it's the AWS version of IaC.

For simplicity, I stored my codebase on AWS CodeCommit.

The overall architecture looks like

![Overall Architecture](/cloud-resume-architecture.png)

## What's Next
I am going to break down this challenge in a few blogs and explain how everything is setup. Stay tuned...
