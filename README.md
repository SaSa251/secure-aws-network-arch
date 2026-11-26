# Secure AWS Network Architecture & Hybrid Connectivity

A 4-week hands-on project that designs, builds, secures and monitors a multi-layer network on AWS, from a single VPC up to hybrid connectivity, edge protection and incident response.

## Team

| Name | Role |
|------|------|
| Mustafa | Project Lead, Week 1: VPC Core Architecture |
| Omar Assaf | Week 2: Advanced Connectivity & Hybrid Networking |
| Muhammad Haliem | Week 3: Edge Security, WAF, DDoS & Content Delivery |
| Abdullah Abdelrehim | Week 4: Network Monitoring, Detection & IR |
| Basheer Tariq | Documentation, Architecture Diagrams & Final Report |

## Project Overview

| Week | Topic | Folder |
|------|-------|--------|
| 1 | VPC Core Architecture | [`week1-vpc/`](./week1-vpc) |
| 2 | Advanced Connectivity & Hybrid Networking | [`week2-hybrid/`](./week2-hybrid) |
| 3 | Edge Security, WAF, DDoS & Content Delivery | [`week3-edge/`](./week3-edge) |
| 4 | Network Monitoring, Detection & Incident Response | [`week4-monitoring/`](./week4-monitoring) |
| - | Final secure network report | [`final-report/`](./final-report) |

## Week 1: VPC Core Architecture

**Scope:** VPC with public/private subnets, route tables, Internet Gateway, NAT Gateway, SSM access, EC2, ALB, RDS, Security Groups, NACLs, least privilege.

**Deliverables:**
- [ ] VPC design and subnet/CIDR plan
- [ ] EC2 / ALB / RDS deployment
- [ ] Security Group and NACL rules
- [ ] Architecture diagram
- [ ] Connectivity test results

## Week 2: Advanced Connectivity & Hybrid Networking

**Scope:** VPC peering, VPC endpoints, AWS PrivateLink, Transit Gateway, Site-to-Site VPN, Client VPN, Direct Connect concepts, routing and segmentation tests.

**Deliverables:**
- [ ] Hub-and-spoke / TGW diagram
- [ ] Peering and endpoint setup
- [ ] VPN configuration
- [ ] Route table documentation
- [ ] Hybrid connectivity test report

## Week 3: Edge Security, WAF, DDoS & Content Delivery

**Scope:** CloudFront with S3 origin, AWS WAF rules, Shield concepts, Firewall Manager, Route 53, Global Accelerator comparison, edge protection tests.

**Deliverables:**
- [ ] CloudFront + S3 lab
- [ ] WAF rule set
- [ ] DDoS mitigation plan
- [ ] Edge architecture
- [ ] Route 53 / health check design

## Week 4: Network Monitoring, Detection & IR

**Scope:** VPC Flow Logs, Traffic Mirroring, CloudWatch, Athena log analysis, compromised-instance simulation, isolation, SG changes, snapshots and evidence collection.

**Deliverables:**
- [ ] Flow log queries
- [ ] Detection use cases
- [ ] Compromised-instance IR runbook
- [ ] Evidence collection notes
- [ ] Final secure network report

## Repository Structure

```
secure-aws-network-arch/
├── README.md
├── .gitignore
├── week1-vpc/
├── week2-hybrid/
├── week3-edge/
├── week4-monitoring/
└── final-report/
```

Each week folder contains:
- `diagrams/` : architecture diagrams
- `screenshots/` : proof of configuration and test results
- `docs/` : CIDR plans, rule tables, test reports
- `iac/` : optional CLI scripts / CloudFormation / Terraform (no secrets)

## Tools & Services

AWS VPC, EC2, ALB, RDS, Systems Manager (SSM), Transit Gateway, VPN, PrivateLink, CloudFront, S3, WAF, Shield, Route 53, CloudWatch, VPC Flow Logs, Athena.

## Security Notice

This repository must **never** contain AWS access keys, secret keys, passwords, `.pem` files, or real account IDs. Redact them from screenshots before committing.

## Contribution Workflow

1. Create a branch: `git checkout -b week1-yourname`
2. Commit your work with a clear message
3. Push and open a Pull Request
4. Another teammate reviews, then merges into `main`

## License

Educational project. Add a license here if required by the initiative.
