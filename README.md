# AWS Cloud Practitioner — Hands-On Projects

Personal learning projects built while studying for the **AWS Certified Cloud Practitioner (CLF-C02)** exam.
Region: `ap-southeast-1` (Singapore) | Environment: AWS Free Tier

---

## Project 1 — Static Website on Amazon S3

A static website deployed to S3 using the AWS CLI, with public bucket policy, versioning, and static website hosting enabled.

### Architecture

```
Internet Users
      ↓
[ S3 Bucket — Static Website Hosting ]
  index.html + error.html <- INSIDE PROJECT01 FOLDER
```

### What was practiced

- Creating an IAM user with least-privilege permissions (S3 only)
- Creating and configuring an S3 bucket in ap-southeast-1
- Writing and applying an S3 Bucket Policy (JSON) for public read access
- Enabling Static Website Hosting
- Deploying files using the AWS CLI (`aws s3 sync`)
- Enabling S3 Versioning and testing rollback to a previous version

### Deploy commands

```bash
# Configure CLI with IAM user credentials
aws configure

# Upload files
aws s3 sync . s3://bucket-name/

# Verify upload
aws s3 ls s3://bucket-name/
```

### Key concepts demonstrated

| Concept | Detail |
|---|---|
| Least Privilege | IAM user with AmazonS3FullAccess only — no EC2, no IAM |
| Bucket Policy | `s3:GetObject` allowed for `Principal: *` to enable public access |
| Block Public Access | Disabled to allow public website hosting |
| Versioning | Enabled — delete marker added on "delete", old version recoverable |

---

## Project 2 — EC2 Web Server with Application Load Balancer + ASG

A highly available web application where the Auto Scaling Group owns the EC2 instance lifecycle — instances are never launched manually. ASG creates and registers them automatically behind the Application Load Balancer.

### Architecture

```
Internet Users
      ↓
[ Application Load Balancer ]   ← single DNS endpoint
     /              \
[EC2 Instance 1]  [EC2 Instance 2]   ← Target Group
                                        (auto-registered by ASG)
      ↑
Auto Scaling Group (Min=1, Desired=2, Max=4)
  └─ Launch Template (defines EC2 blueprint + User Data)
```

### Workflow (build order)

```
1. Security Groups       — ALB-SG (open to internet), EC2-SG (accepts from ALB-SG only)
         ↓
2. Launch Template       — defines AMI, instance type, EC2-SG, User Data script
         ↓
3. Target Group          — empty at first, HTTP port 80, health check on /
         ↓
4. Application Load Balancer — listener port 80 → forward to Target Group
         ↓
5. Auto Scaling Group    — uses Launch Template, attached to Target Group
                           ASG auto-creates instances + auto-registers them to ALB
```

> ASG owns the instances — they are never created manually. This is the production approach.

### What was practiced

- Creating Security Groups with SG-to-SG referencing (EC2 only accepts traffic from ALB-SG)
- Writing a Launch Template with User Data to auto-install and configure Apache httpd
- Creating an empty Target Group and attaching it to an ALB
- Creating an Application Load Balancer (Layer 7) before instances exist
- Creating an ASG that auto-provisions and auto-registers instances into the Target Group
- Demonstrating health checks — stopping one EC2 reroutes traffic automatically

### EC2 User Data script (inside Launch Template)

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2 CCP Project 02 - Server $(hostname -f) </h1>" > /var/www/html/index.html
```

`$(hostname -f)` displays the private DNS name of whichever instance handled the request — visible when refreshing the ALB URL, confirming load balancing is working.

### Key concepts demonstrated

| Concept | Detail |
|---|---|
| Security Group chaining | EC2-SG source set to ALB-SG ID — EC2 never exposed directly to internet |
| Launch Template | Reusable EC2 blueprint — ASG uses this to spawn instances |
| ASG-managed lifecycle | ASG creates, registers, and terminates instances — no manual EC2 launch |
| Health Checks | ALB polls `/` every 30s — unhealthy instances removed from rotation |
| High Availability | Instances across AZs — one failure does not affect the site |
| Auto Scaling | Target Tracking keeps average CPU at 50%, scales in/out automatically |

---

## Skills Covered

`IAM` `S3` `EC2` `EBS` `ELB` `ASG` `Security Groups` `AWS CLI` `Static Website Hosting` `Versioning` `Bucket Policy`

---

## Study Context

Following Stephane Maarek's AWS Cloud Practitioner course on Udemy + Linux Foundation LFS101 on edX.
Target certification: **AWS Certified Cloud Practitioner (CLF-C02)**
