# AWS Secure VPC & EC2 Web Server

**Hands-on AWS networking and deployment case study** by Islam Din.

> **Live portfolio:** https://islam11222.github.io/aws-vpc-ec2-secure-web-server/  
> **GitHub repository:** https://github.com/islam11222/aws-vpc-ec2-secure-web-server
>
> **Honest hosting distinction:** The GitHub Pages demo is hosted on GitHub, **not on EC2**. The Apache website was separately deployed and verified on AWS EC2. The EC2 instance may be stopped to avoid ongoing expenses.

## Project overview

Built and tested a custom Amazon VPC architecture in the Tokyo Region using the AWS Management Console. Deployed an EC2 instance running Amazon Linux 2023 and Apache HTTP Server, then investigated and recovered a failed first-boot user-data deployment.

## Architecture

```text
                 Internet
                    |
             Internet Gateway
                    |
         Public Route Table (0.0.0.0/0 -> IGW)
                    |
             VPC 10.0.0.0/16
      +-------------------------------------+
      | ap-northeast-1a   ap-northeast-1c   |
      | Public subnet 1  Public subnet 2   |
      | EC2 + Apache      No EC2 deployed  |
      | Private subnet 1 Private subnet 2  |
      | No EC2 deployed   No EC2 deployed  |
      +-------------------------------------+
```

- **VPC:** `10.0.0.0/16`
- **Availability Zones:** `ap-northeast-1a`, `ap-northeast-1c`
- **Subnets:** 2 public and 2 private
- **Public subnet 1:** `10.0.0.0/20`
- **Routing:** public IPv4 default route to Internet Gateway; private routes checked without direct IGW route
- **NAT Gateway:** none (cost-conscious lab)
- **EC2:** Amazon Linux 2023, `t3.micro`
- **Root EBS:** 8 GiB `gp3`, encrypted with AWS-managed `aws/ebs`, delete on termination enabled

## Security groups

| Purpose | Protocol | Port | Source |
|---|---|---|---|
| Website (HTTP demo) | TCP | 80 | `0.0.0.0/0` |
| Admin SSH | TCP | 22 | Current admin public IP `/32` |
| Browser-based EC2 Instance Connect | TCP | 22 | AWS regional EC2 Instance Connect managed prefix list |

**Security note:** The original EC2 website was an HTTP-only learning demo. It should not be described as HTTPS-secured or production-ready. No password, private key or secrets belong in this repository.

## Deployment and verification

1. Created VPC, four subnets, Internet Gateway, and route tables using AWS Console.
2. Verified public IPv4 routing and two private route tables.
3. Launched EC2 in public subnet 1 with an auto-assigned public IPv4 address.
4. Configured access rules and encrypted EBS storage.
5. Initially used User Data to install and start Apache.
6. Diagnosed the failure with the EC2 System Log and instance status checks.
7. Connected using browser-based EC2 Instance Connect, installed Apache, enabled the web service, and created `/var/www/html/index.html`.
8. Confirmed `HTTP/1.1 200 OK` from localhost and browser access over HTTP from the public Internet.

## Troubleshooting story

**Problem:** Initial browser requests failed although the EC2 status checks passed.

**Investigation:** Reviewed route tables, security groups, network ACL rules and subnet association. The System Log showed `cloud-init` failed to run `scripts-user`, indicating the User Data setup did not complete successfully. The exact failed script command was not established from the system log alone.

**Recovery:** Installed Apache manually on the same EC2 instance, enabled and started `httpd`, then added a small static `index.html`. Confirmed an HTTP 200 response and a working public browser page.

**Lesson:** Healthy EC2 infrastructure does not guarantee that guest OS software or initialization scripts succeeded. Separate network, operating system, and application checks.

## Live portfolio deployment

The repository-root `index.html` is a **static portfolio showcase**, publicly deployed using GitHub Pages from the `main` branch and `/(root)` directory. The live showcase is available at:

**https://islam11222.github.io/aws-vpc-ec2-secure-web-server/**

GitHub Pages is a separate hosting service. The page is not served by the EC2 instance, and its availability does not demonstrate that EC2 is currently running.

## Evidence screenshots (add after checking for sensitive information)

Place selected, appropriately cropped screenshots in the `screenshots/` folder:

- VPC resource map with public/private subnets
- Public and private route tables
- EC2 instance details and encrypted EBS volume (crop sensitive identifiers as desired)
- Security group rules (review IP details before publishing)
- The original EC2 website working in a browser
- Cloud-init error and subsequent HTTP 200 recovery (redact sensitive details)

## Cost and cleanup

- Running EC2, EBS volumes and public IPv4 addresses may incur usage charges or consume AWS credits.
- Stopped EC2 instances may still incur EBS storage charges. Terminated instances may leave billable resources if cleanup is incomplete.
- GitHub Pages can continue serving this static case-study page after the original EC2 instance is stopped or terminated.
- If the EC2 instance is stopped and restarted, its auto-assigned public IPv4 address can change.

## Skills demonstrated

AWS Management Console · Amazon VPC · CIDR and subnets · Multi-AZ design · Route tables · Internet Gateway · Security groups · Network ACLs · EC2 · Encrypted EBS · Amazon Linux · Apache HTTP Server · cloud-init investigation · EC2 Instance Connect · Cost-awareness
