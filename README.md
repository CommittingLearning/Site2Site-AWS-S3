# AWS S3 Bucket for CloudFormation Templates with CI/CD Pipeline

This repository contains an AWS CloudFormation template for provisioning an S3 bucket to host CloudFormation templates (CIS compliant). It also includes a GitHub Actions CI/CD pipeline for automated deployment, validation, and security checks.

## Table of Contents

- [Introduction](#introduction)
- [CloudFormation Template](#cloudformation-template)
  - [Resources Created](#resources-created)
  - [Parameters](#parameters)
  - [Outputs](#outputs)
- [CI/CD Pipeline](#cicd-pipeline)
  - [Workflow Triggers](#workflow-triggers)
  - [Pipeline Overview](#pipeline-overview)
  - [Environment Variables and Secrets](#environment-variables-and-secrets)
- [Usage](#usage)
  - [Clone the Repository](#clone-the-repository)
  - [Set Up AWS Credentials](#set-up-aws-credentials)
  - [Configure the S3 Bucket](#configure-the-s3-bucket)
  - [Branch Strategy](#branch-strategy)
  - [Manual Approval](#manual-approval)
- [Notes](#notes)

## Introduction

This project automates the deployment of an AWS S3 bucket designed to host CloudFormation templates, ensuring compliance with CIS benchmarks. The bucket is configured to be private and has versioning enabled.

The GitHub Actions CI/CD pipeline automates validation, security scanning, and deployment processes.

The CI/CD pipeline is designed to:

- Validate and test CloudFormation templates on pull requests.
- Deploy infrastructure on pushes to specific branches.
- Perform security checks using CloudFormation Guard.
- Require manual approval before deployment.

## CloudFormation Template

### Resources Created

The CloudFormation template (`template.yaml`) provisions the following resources:

1. **AWS S3 Bucket:**

   - **Name:** `${BucketName}-${Environment}-${AWS::AccountId}`
   - **Access Control:** Private
   - **Versioning:** Enabled (required by CIS benchmarks)
   - **Public Access Block Configuration:** All public access blocked
   - **Tags:**
     - `Name`: `${BucketName}-${Environment}`
     - `Environment`: Provided via the `Environment` parameter

### Parameters

The template accepts the following parameters:

- **Environment:**

  - **Type:** String
  - **Description:** The environment name (provided via Github Workflow pipeline)
  - **Allowed Values:** `development`, `production`, `default`

- **BucketName:**

  - **Type:** String
  - **Description:** The base name for the S3 bucket
  - **Default:** `s2s-ts101-cftemplates`

### Outputs

- **BucketName:**
  - **Description:** The name of the created S3 bucket
  - **Value:** The reference to the S3 bucket resource

- **BucketArn:**
  - **Description:** The ARN of the created S3 bucket
  - **Value:** The ARN attribute of the S3 bucket resource

## CI/CD Pipeline

The CI/CD pipeline is defined in the GitHub Actions workflow file `.github/workflows/aws-cloudformation.yml`. It automates the deployment process and ensures code quality and security.

### Workflow Triggers

The pipeline is triggered on:

- **Pull Requests** to the following branches:
  - `development`
  - `production`
  - `testing`
- **Pushes** to the following branches:
  - `development`
  - `production`

### Pipeline Overview

The pipeline consists of two primary jobs:

1. **Validate and Test (`validate-and-test`):**

   - **Checkout Code:** Retrieves the repository code.
   - **Configure AWS Credentials:** Assumes an IAM role using OpenID Connect (OIDC) with the provided credentials.
   - **Validate CloudFormation Template:** Validates the syntax of the CloudFormation template.
   - **Run CloudFormation Guard:** Performs security checks using CloudFormation Guard against CIS benchmarks.
   - **Skip Apply in Pull Requests:** Ensures that deployment does not occur on pull requests.

2. **Deploy (`deploy`):**

   - **Depends On:** The `validate-and-test` job must succeed.
   - **Runs On:** Not triggered on pull requests.
   - **Checkout Code:** Retrieves the repository code.
   - **Configure AWS Credentials:** Assumes an IAM role using OIDC with the provided credentials.
   - **Set Environment Variable:** Determines the environment (`development`, `production`, or `default`) based on the branch.
   - **Manual Approval:** Requires manual approval via GitHub Issues before proceeding with the deployment.
   - **Deploy CloudFormation Stack:** Deploys the CloudFormation stack using AWS CLI.
   - **Describe Deployment:** Outputs information about the deployed stack.

### Environment Variables and Secrets

The pipeline uses the following secrets and environment variables:

- **Secrets (Stored in GitHub Secrets):**
  - `AWS_ROLE_TO_ASSUME`: ARN of the IAM role to assume for deployment.
  - `github_TOKEN`: Automatically provided by GitHub for authentication in workflows.

- **Environment Variables:**
  - `ENVIRONMENT`: Set based on the branch (`development`, `production`, or `default`).

## Usage

### Clone the Repository

```bash
git clone https://github.com/CommittingLearning/Site2Site-AWS-S3.git
```

### Set Up AWS Credentials

Ensure that the following secret is added to your GitHub repository under **Settings > Secrets and variables > Actions**:

- `AWS_ROLE_TO_ASSUME`

This should be the ARN of an IAM role that the GitHub Actions workflow can assume, with the necessary permissions to deploy CloudFormation stacks.

### Configure the S3 Bucket

The S3 bucket is configured to be private and compliant with CIS benchmarks. The bucket name is constructed as `${BucketName}-${Environment}-${AWS::AccountId}` to ensure uniqueness.

- **BucketName Parameter:** Adjust the `BucketName` parameter in the `template.yaml` file if you need a different base name.
- **Environment Parameter:** The environment (`development`, `production`, or `default`) is determined based on the branch name.

### Branch Strategy

- **Development Environment:** Use the `development` branch to deploy to the development environment.
- **Production Environment:** Use the `production` branch to deploy to the production environment.
- **Default Environment:** Any other branches will use the `default` environment settings.

### Manual Approval

The pipeline requires manual approval before applying changes:

- A GitHub issue will be created prompting for approval.
- Approvers need to approve the issue to proceed with deployment.

## Notes

- **Security Checks:**
  - The pipeline includes security checks using CloudFormation Guard to ensure compliance with CIS benchmarks.

- **Nested CloudFormation Templates:**
  - This S3 bucket is intended to store child CloudFormation templates that are called by parent stacks.

- **Testing:**
  - Pull requests to `development`, `production`, or `testing` branches will trigger the validation and testing steps without applying changes.

- **IAM Role Configuration:**
  - Ensure that the IAM role specified in `AWS_ROLE_TO_ASSUME` has permissions to deploy CloudFormation stacks and interact with S3.

---

**Disclaimer:** This repository is accessible in a read only format, and therefore, only the admin has the privileges to perform a push on the branches.