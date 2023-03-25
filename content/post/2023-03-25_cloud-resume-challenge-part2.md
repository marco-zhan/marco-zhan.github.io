---
title:       "The Cloud Resume Challenge, Part 2 - Hosting Static Webpage with AWS S3, CI/CD in AWS and Frontend"
subtitle:    ""
description: "This post talks about how to set up CI/CD in AWS, how to host Static webpage with AWS S3 and create a React app"
date:        2023-03-25
author: "Marco"
image:       ""
tags:        ["AWS", "Cloud", "CI/CD"]
categories:  ["AWS"]
---

## Frontend
The frontend is pretty straight forward,  I used React for writing up my resume webpage, but you can choose from 
any JavaScript framework of your preference. I am not going to talk too much on frontend because this is not the main
focus of this challenge.

## CI/CD
I used AWS CodePipeline for CI/CD of this challenge. The CI/CD pipeline is triggered by code change in the nominated code
repository, in my case this is on AWS CodeCommit, but you can use GitHub or other Git services. When the pipeline is triggered,
it will initiate the CodeBuild stage, which we will build the React app. The artifacts from the build stage is then passed on to
the deploy stage, which we will deploy the built app to AWS S3.

We will need to create build_spec.yml file to put in the build steps for our project and also specifies the output of the 
CodeBuild stage, below is the build_spec I used

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo "Entered the install phase..."
      - apt-get update -y
      - cd cloud-resume-frontend
      - npm install
  build:
    commands:
      - echo "Entered the build phase..."
      - npm run build
      - echo "Finished the build phase..."
  post_build:
    commands:
      - echo "Entered the post-build phase"
      - echo "Finished the post-build phase"

artifacts:
  files:
    - "**/*"
  base-directory: "cloud-resume-frontend/build" # output everything from the built app
```

We will also need to set up the role in order for our pipeline to use access other AWS services, e.g. S3
Below is the configuration of our AWS CodePipeline in AWS CloudFormation
```yaml
Resources:
  CloudResumeCodePipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      Name: CloudResumeCodePipeline
      RoleArn:
        !GetAtt CloudResumeCodePipelineRole.Arn
      Stages:
        - Name: Source
          Actions:
            - Name: SourceAction
              ActionTypeId:
                Category: Source
                Owner: AWS
                Provider: CodeCommit # GitHub, BitBucket etc.
                Version: 1
              OutputArtifacts:
                -
                  Name: SourceArtifact
              Configuration:
                RepositoryName: cloud-resume 
                BranchName: master
        - Name: CodeBuild
          Actions:
            - Name: CodeBuildAction
              InputArtifacts:
                -
                  Name: SourceArtifact
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: 1
              OutputArtifacts:
                  -
                    Name: CodeBuildArtifacts
              Configuration:
                ProjectName: !Ref CloudResumeCodeBuildProject
        - Name: CodeDeploy
          Actions:
            - Name: CodeDeployAction
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: S3
                Version: 1
              InputArtifacts: 
                -
                  Name: CodeBuildArtifacts
              Configuration:
                  BucketName: !Ref CloudResumeOutputArtifactsBucket
                  Extract: true

      ArtifactStore:
        Type: S3
        Location:
          !Ref CloudResumeOutputArtifactsBucket
    
    # Our CodeBuild Project, we will need to specify the VM type, and the steps for our build stage
    CloudResumeCodeBuildProject:
      Type: AWS::CodeBuild::Project
      Properties:
          Artifacts:
            Type: CODEPIPELINE
          Description: Cloud Resume CodeBuild Project
          Environment:
              ComputeType: BUILD_GENERAL1_SMALL
              Image: aws/codebuild/standard:6.0-23.02.16
              Type: LINUX_CONTAINER
              PrivilegedMode: true
          Name: cloud-resume-build-project
          ServiceRole:
            !Ref CloudResumeCodeBuildRole
          Source:
              Type: CODEPIPELINE
              BuildSpec: ./cloudformation/codebuild/cloud_resume_build_spec.yml
          QueuedTimeoutInMinutes: 10
    
    CloudResumeCodeBuildRole:
      Type: AWS::IAM::Role
      Properties:
        AssumeRolePolicyDocument:
          Version: 2012-10-17
          Statement:
          - Effect: Allow
          Principal:
            Service:
              - codebuild.amazonaws.com
          Action:
            - sts:AssumeRole
          Description: IAM Role for Cloud Resume CodeBuild
          Path: '/'
          Policies:
          - PolicyName: root
          PolicyDocument:
              Version: '2012-10-17'
              Statement:
                - Effect: Allow
                  Action:
                    - codebuild:*
                    - logs:*
                    - '*'
              Resource: '*'
    
    CloudResumeCodePipelineRole:
      Type: 'AWS::IAM::Role'
      Properties:
        AssumeRolePolicyDocument:
          Version: 2012-10-17
          Statement:
          - Effect: Allow
          Principal:
            Service:
              - codepipeline.amazonaws.com
          Action: 'sts:AssumeRole'
        Description: IAM Role for Cloud Resume CodePipeline
        Path: /
        Policies:
          - PolicyName: CloudResumeCodePipelineServicePolicy
            PolicyDocument:
              Version: 2012-10-17
              Statement:
                - Effect: Allow
                  Action:
                    - 'codecommit:CancelUploadArchive'
                    - 'codecommit:GetBranch'
                    - 'codecommit:GetCommit'
                    - 'codecommit:GetUploadArchiveStatus'
                    - 'codecommit:UploadArchive'
                  Resource: '*'
                - Effect: Allow
                  Action:
                    - 'codedeploy:CreateDeployment'
                    - 'codedeploy:GetApplicationRevision'
                    - 'codedeploy:GetDeployment'
                    - 'codedeploy:GetDeploymentConfig'
                    - 'codedeploy:RegisterApplicationRevision'
                  Resource: '*'
                - Effect: Allow
                  Action:
                    - 'codebuild:BatchGetBuilds'
                    - 'codebuild:StartBuild'
                  Resource: '*'
                - Effect: Allow
                  Action: # if you want to perform action with AWS Lambda (optional)
                    - 'lambda:InvokeFunction'
                    - 'lambda:ListFunctions'
                    - 'lambda:CreateFunction'
                    - 'lambda:UpdateFunctionConfiguration'
                    - 'lambda:UpdateFunctionCode'
                    - 'lambda:TagResource'
                    - 'lambda:PublishVersion'
                    - 'lambda:GetFunctionConfiguration'
                    - 'lambda:GetFunction'
                  Resource: '*'
                - Effect: Allow
                  Action:
                    - 'iam:PassRole'
                  Resource: '*'
                - Effect: Allow
                  Action:
                    - 'ec2:*' # optional
                    - 'cloudwatch:*'
                    - 's3:*'
                    - 'cloudformation:*' #optional
                  Resource: '*'
```

## AWS S3
We will use AWS S3 to store and host our resume webpage. Once the deploy stage of our CI/CD pipeline is completed, it will
store the built React app to our designated S3 bucket. Under the Properties tab of the bucket on AWS Console, on the very bottom
of the page, we can enable stable website hosting for our bucket and set the index document to index.html and save it. 
After that, we can hit the url of our bucket, and we should see our resume webpage showing up.

However, as we will be using AWS CloudFront for accessing our S3 bucket, the static web hosting option of our bucket should
be turned off, and we should also specify a policy that only allows AWS CloudFront to access our bucket.

Below is the configuration of our bucket and the bucket policy required in AWS CloudFormation
```yaml
Resources:
  CloudResumeOutputArtifactsBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: [BUCKET_NAME]
      AccessControl: Private
      # WebsiteConfiguration: Use this option if you want to enable static web hosting

  CloudResumeOutputArtifactBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref CloudResumeOutputArtifactsBucket
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Action:
              - 's3:GetObject'
            Effect: Allow
            Resource: !Join
              - ''
              - - 'arn:aws:s3:::'
                - !Ref CloudResumeOutputArtifactsBucket
                - /*
            Principal:
              Service: "cloudfront.amazonaws.com" # only allow access from AWS CloudFront
```


