# Week 1: Security Group & NACL Rules

Security is applied in two layers: **Security Groups** (stateful, attached to resources) and **Network ACLs** (stateless, attached to subnets). Both follow the principle of least privilege.

## 1. Security Groups

Security Groups reference each other instead of IP ranges, so access follows the *role* of a resource, not its address.

### `alb-sg` (Application Load Balancer)

| Direction | Protocol / Port | Source or destination | Purpose |
|-----------|-----------------|-----------------------|---------|
| Inbound | TCP 80 | `0.0.0.0/0` | Public HTTP traffic |
| Outbound | TCP 80 | `app-sg` | Forward requests to the application servers only |

### `app-sg` (EC2 application servers)

| Direction | Protocol / Port | Source or destination | Purpose |
|-----------|-----------------|-----------------------|---------|
| Inbound | TCP 80 | `alb-sg` | Accept web traffic only from the load balancer |
| Outbound | TCP 443 | `0.0.0.0/0` | Package repositories and Systems Manager (via NAT) |
| Outbound | TCP 3306 | `db-sg` | MySQL connections to the database |

### `db-sg` (RDS MySQL)

| Direction | Protocol / Port | Source or destination | Purpose |
|-----------|-----------------|-----------------------|---------|
| Inbound | TCP 3306 | `app-sg` | Accept database connections only from the application tier |
| Outbound | none | none | The database initiates no connections. Replies to allowed inbound traffic are sent automatically because Security Groups are stateful. |

The default "allow all outbound" rule was removed from all three groups.

## 2. Network ACLs

NACLs are **stateless**: return traffic must be allowed explicitly. Clients connect from random *ephemeral* ports (1024-65535), so each NACL must allow that range for replies. This is not a weakness: it is required for the connection to work, and the Security Groups still restrict who can actually reach each resource.

Every NACL ends with the default rule `*` (all traffic, **Deny**).

### `public-nacl` (`public-a`, `public-b`)

| Direction | Rule | Protocol / Port | Source or destination | Action |
|-----------|------|-----------------|-----------------------|--------|
| Inbound | 100 | TCP 80 | `0.0.0.0/0` | Allow |
| Inbound | 110 | TCP 443 | `0.0.0.0/0` | Allow |
| Inbound | 120 | TCP 1024-65535 | `0.0.0.0/0` | Allow |
| Outbound | 100 | TCP 80 | `0.0.0.0/0` | Allow |
| Outbound | 110 | TCP 443 | `0.0.0.0/0` | Allow |
| Outbound | 120 | TCP 1024-65535 | `0.0.0.0/0` | Allow |

### `app-nacl` (`app-a`, `app-b`)

| Direction | Rule | Protocol / Port | Source or destination | Action |
|-----------|------|-----------------|-----------------------|--------|
| Inbound | 100 | TCP 80 | `10.0.1.0/24` (`public-a`) | Allow |
| Inbound | 110 | TCP 80 | `10.0.2.0/24` (`public-b`) | Allow |
| Inbound | 120 | TCP 1024-65535 | `0.0.0.0/0` | Allow (replies from the internet via NAT and from the database) |
| Outbound | 100 | TCP 443 | `0.0.0.0/0` | Allow |
| Outbound | 110 | TCP 3306 | `10.0.21.0/24` (`db-a`) | Allow |
| Outbound | 120 | TCP 3306 | `10.0.22.0/24` (`db-b`) | Allow |
| Outbound | 130 | TCP 1024-65535 | `10.0.1.0/24` (`public-a`) | Allow (replies to the load balancer) |
| Outbound | 140 | TCP 1024-65535 | `10.0.2.0/24` (`public-b`) | Allow (replies to the load balancer) |

### `db-nacl` (`db-a`, `db-b`)

| Direction | Rule | Protocol / Port | Source or destination | Action |
|-----------|------|-----------------|-----------------------|--------|
| Inbound | 100 | TCP 3306 | `10.0.11.0/24` (`app-a`) | Allow |
| Inbound | 110 | TCP 3306 | `10.0.12.0/24` (`app-b`) | Allow |
| Outbound | 100 | TCP 1024-65535 | `10.0.11.0/24` (`app-a`) | Allow (replies) |
| Outbound | 110 | TCP 1024-65535 | `10.0.12.0/24` (`app-b`) | Allow (replies) |

The database subnets accept traffic **only** from the application subnets, and can answer **only** to them.

## 3. IAM (Least Privilege)

| Role | Attached policy | Used by |
|------|-----------------|---------|
| `ec2-ssm-role` | `AmazonSSMManagedInstanceCore` | EC2 instance profile, enables Session Manager access without SSH |

## 4. Defense in Depth: Request Path

```
Internet --80--> [public-nacl] --> alb-sg --> ALB
ALB --80--> [app-nacl] --> app-sg --> EC2
EC2 --3306--> [db-nacl] --> db-sg --> RDS
```

A request must pass a NACL *and* a Security Group at each hop. A mistake in one layer is still caught by the other.

## 5. Findings

- The database is unreachable from the internet and from anything except the application tier, at both the subnet layer (NACL, route table) and the resource layer (Security Group, no public IP).
- Application servers cannot be reached directly from the internet. Only the load balancer can reach them on port 80.
- Outbound access from the application servers is limited to HTTPS and MySQL. A plain HTTP request to the internet (port 80) is blocked (see `test-results.md`).
