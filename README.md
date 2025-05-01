# AWS Web Application Deployment

This repository contains the files and documentation for deploying a highly available web application on AWS using S3, Auto Scaling Group (ASG), and Application Load Balancer (ALB).

## Assignment Overview

## Part 1: S3 Setup (Static Assets)

### Steps:

1.  **Created an S3 bucket:**
    * I created a bucket named `2028lamya-clarusway-assets` in the `eu-north-1` region. 
    * Screenshot: `s3_bucket_creation.png`
2.  **Uploaded files:**
    * Uploaded the provided `index.html`, `logo.png`, and `sda.png` files to the bucket.
3.  **Configured S3 website hosting:**
    * Enabled static website hosting for the bucket.
    * Set `index.html` as the index document.
    * Screenshot: `s3_website_hosting_config.png`
4.  **Set bucket policy:**
    * Applied a bucket policy to allow public read access to the files.
    * ```json
        {
          "Version": "2012-10-17",
          "Statement": [{
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::2028lamya-clarusway-assets/*"  
          }]
        }
        ```
    * This policy allows anyone to view the files in the bucket, which is necessary for a public website.

### Deliverables:

* S3 website URL screenshot: `s3_website_url.png`
* `curl -I` output showing 200 OK: `curl_output.png`

## Part 2: Auto Scaling Group

### Steps:

1.  **Created Launch Template:**
    * Created a Launch Template to define the configuration for the EC2 instances in the ASG.
    * Used the Amazon Linux 2 AMI.
    * Included the following User Data script to automate the setup of NGINX:
        ```bash
        #!/bin/bash
        yum update -y
        yum install nginx -y
        systemctl start nginx
        systemctl enable nginx
        aws s3 cp s3://2028lamya-clarusway-assets/index.html /usr/share/nginx/html/
        ```
        * This script updates the system, installs NGINX, starts and enables NGINX, and copies the `index.html` file from the S3 bucket to the NGINX web root directory.
    * Screenshot: `launch_template_config.png`
2.  **Configured Auto Scaling Group:**
    * Created an ASG using the Launch Template.
    * Set the minimum size to 1, the maximum size to 3, and the desired capacity to 2.
    * Configured health checks to use both EC2 and ELB health checks. This ensures that ASG replaces instances that are unhealthy or fail the load balancer's health checks.
    * Screenshot: `asg_config.png`

### Deliverables:

* Screenshot of 2 running instances: `running_instances.png`
* ASG configuration details: `asg_details.png`

## Part 3: Application Load Balancer

### Steps:

1.  **Created internet-facing ALB:**
    * Created an internet-facing ALB to distribute traffic to the NGINX instances.
    * Configured an HTTP listener on port 80.
    * Created a target group and registered the ASG with it.
    * Configured health checks for the target group to use the `/` path.
    * Screenshot: `alb_config.png`
2.  **Verified access via ALB DNS name:**
    * Accessed the website using the ALB's DNS name.
    * Screenshot: `alb_access.png`
3.  **Verified round-robin traffic distribution:**
    * Used `curl` commands to send multiple requests to the ALB and observed that the requests were distributed across different instances.
    * Command used:
        ```bash
        for /L %i in (1,1,5) do curl -s http://your-alb-dns | grep "your-text-to-identify-instance"
        ```
        * (Replace `your-text-to-identify-instance` with a string that helps you identify which instance is serving the request, if applicable).
    * Screenshot: `curl_distribution_verification.png`

### Deliverables:

* ALB DNS output screenshot: `alb_dns_output.png`
* `curl` tests showing different instance IDs (or other evidence of distribution): `curl_tests.png`

## Success Criteria

* The website is accessible via both the S3 endpoint (static version) and the ALB endpoint (dynamic version via ASG).
* The ASG automatically replaces terminated instances (verified by testing).
* All assets (HTML and logos) load properly.

## Cleanup

* Deleted the S3 bucket.
* Terminated the ASG (which automatically deleted the instances).
* Removed the ALB.

## Configuration Files

* `index.html`
* `logo.png`
* `sda.png`
