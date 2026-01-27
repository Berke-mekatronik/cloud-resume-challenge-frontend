# Cloud Resume Challenge - Frontend
This repository contains the frontend of my Cloud Resume Challenge project. The website is a static resume built with HTML, CSS, and JavaScript and is hosted entirely on AWS using modern cloud best practices. You can reach to the project from there: [Berke Ozturk Resume](https://s3.domain-of-berke.com/)

This project is based on the original challenge created by **Forrest Brazeal**: [Cloud Resume Challenge](https://cloudresumechallenge.dev/docs/the-challenge/)

## Architecture Diagram
![Architecture Diagram](/img/Architecture_Diagram.png)

## Infrastructure Components Created
The frontend infrastructure is fully provisioned using **Terraform** and consists of:

* **Amazon S3** for static website hosting
* **Amazon CloudFront** for Global CDN and HTTPS termination
* **Amazon Route 53** for DNS management
* Infrastructure provisioned using **Terraform**
* Designed to be deployed automatically via CI/CD (GitHub Actions)

## Static Website Hosting
The resume is hosted in an S3 bucket configured for static website hosting.  
Direct public access to the bucket is restricted, and all content is delivered securely through CloudFront.

CloudFront enforces HTTPS and caches content globally for low latency access.

![S3 Settings](/img/S3.png)

## DNS & HTTPS
A custom domain is managed using **Amazon Route 53**, where an alias record points to the CloudFront distribution.

![CloudFront Settings](/img/CloudFront.png)

CloudFront handles HTTPS using its certificate configuration and enforces secure access.

![HTTPS](/img/HTTPS.png)

## CI/CD Deployment Flow
The frontend deployment is fully automated using GitHub Actions:

1. Code is pushed to the repository
2. GitHub Actions syncs static files to the S3 bucket
3. CloudFront cache is invalidated to reflect the latest changes

This ensures fast, consistent, and repeatable deployments without manual intervention.

## Live Preview

![Preview_of_resume](/img/Preview_of_resume.png)

## Notes

This frontend communicates with a serverless backend API to fetch and display the live visitor count.

The backend implementation can be found in the corresponding backend repository
