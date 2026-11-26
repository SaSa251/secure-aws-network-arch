# Week 1: Deployment Notes (EC2, ALB, RDS)

**Region:** eu-central-1 (Frankfurt) | **VPC:** `secure-vpc` (`10.0.0.0/16`)

## 1. Build Order

1. VPC and DNS hostnames enabled
2. Six subnets (two AZs, three tiers)
3. Internet Gateway attached
4. NAT Gateway with an Elastic IP in `public-a`
5. Route tables (`public-rt`, `app-rt`, `db-rt`) and subnet associations
6. Security Groups (`alb-sg`, `app-sg`, `db-sg`), then outbound rules restricted
7. IAM role `ec2-ssm-role`
8. EC2 instance
9. Target group, then Application Load Balancer
10. DB subnet group, then RDS MySQL
11. Connectivity tests
12. Network ACLs applied last, then tests repeated

NACLs were applied last on purpose: if something had stopped working earlier, the cause would be a Security Group or route, not the NACL.

## 2. EC2 Application Server

| Setting | Value |
|---------|-------|
| Name | `app-server-a` |
| AMI | Amazon Linux 2023 |
| Instance type | `t3.micro` |
| Subnet | `app-a` (`10.0.11.0/24`) |
| Public IP | Disabled |
| Security group | `app-sg` |
| IAM instance profile | `ec2-ssm-role` |
| Key pair | None (access through Session Manager only) |

User data (installs the web server and a test page):

```bash
#!/bin/bash
dnf install -y httpd mariadb105
systemctl enable --now httpd
echo "<h1>Secure AWS Network - app-server-a</h1>" > /var/www/html/index.html
```

## 3. Application Load Balancer

| Setting | Value |
|---------|-------|
| Name | `secure-alb` |
| Scheme | Internet-facing |
| Subnets | `public-a`, `public-b` |
| Security group | `alb-sg` |
| Listener | HTTP : 80, forward to `app-tg` |
| Target group | `app-tg` (HTTP 80, target type: instances) |
| Health check | HTTP, path `/` |
| Registered target | `app-server-a` |

## 4. RDS Database

| Setting | Value |
|---------|-------|
| Identifier | `secure-db` |
| Engine | MySQL |
| Instance class | `db.t3.micro` (single-AZ, lab sizing) |
| DB subnet group | `db-subnet-group` (`db-a`, `db-b`) |
| Public access | **No** |
| Security group | `db-sg` |
| Initial database | `appdb` |
| Credentials | Self-managed master user. The password is **not** stored in this repository. |

## 5. Issues Encountered

| Issue | Cause | Fix |
|-------|-------|-----|
| `curl ifconfig.me` hung from the EC2 instance | The command uses HTTP (port 80). `app-sg` only allows outbound HTTPS (443) and MySQL (3306). | Use `curl https://ifconfig.me`. This confirms the outbound restriction works. |
| First database created with the "Express configuration" | The quick-create option builds an Aurora cluster and skips the VPC, subnet group and security group choices. | Deleted it and used **Standard create** with MySQL so the database lands in `secure-vpc` with `db-sg`. |

## 6. Cost Notes

NAT Gateway, Application Load Balancer and RDS are billed per hour while they exist. After testing, they are deleted in this order: RDS, ALB (and target group), NAT Gateway, then the Elastic IP is released. The VPC, subnets, route tables and Security Groups are free and can stay.
