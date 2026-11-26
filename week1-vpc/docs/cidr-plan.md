# Week 1: VPC Design & CIDR Plan

**Region:** eu-central-1 (Frankfurt)
**VPC:** `secure-vpc`, `10.0.0.0/16` (65,536 addresses)

## 1. Design Goals

- Separate the network into three tiers: **public** (load balancer, NAT), **application** (EC2) and **data** (RDS).
- Spread every tier across **two Availability Zones** so one AZ failure does not take the service down.
- Only the load balancer is reachable from the internet. Application servers and the database have **no public IP**.
- The database tier has **no route to the internet** at all.
- Administrative access through **AWS Systems Manager Session Manager** (no SSH, no bastion host, no open port 22).

## 2. Subnet Plan

| Subnet | AZ | CIDR | Tier | Usable IPs* | Route table | NACL |
|--------|----|------|------|-------------|-------------|------|
| `public-a` | eu-central-1a | `10.0.1.0/24` | Public | 251 | `public-rt` | `public-nacl` |
| `public-b` | eu-central-1b | `10.0.2.0/24` | Public | 251 | `public-rt` | `public-nacl` |
| `app-a` | eu-central-1a | `10.0.11.0/24` | Private (application) | 251 | `app-rt` | `app-nacl` |
| `app-b` | eu-central-1b | `10.0.12.0/24` | Private (application) | 251 | `app-rt` | `app-nacl` |
| `db-a` | eu-central-1a | `10.0.21.0/24` | Private (data) | 251 | `db-rt` | `db-nacl` |
| `db-b` | eu-central-1b | `10.0.22.0/24` | Private (data) | 251 | `db-rt` | `db-nacl` |

\*AWS reserves 5 addresses in every subnet (network address, VPC router, DNS, future use, broadcast), so a /24 gives 256 - 5 = 251 usable IPs.

The numbering is grouped by tier: the third octet is 1-2 for public subnets, 11-12 for application subnets and 21-22 for data subnets. Further subnets (for example a third AZ) can be added in the unused ranges without renumbering.

## 3. Gateways

| Resource | Name | Placement | Purpose |
|----------|------|-----------|---------|
| Internet Gateway | `secure-igw` | Attached to `secure-vpc` | Internet access for the public subnets |
| NAT Gateway | `secure-nat` | `public-a`, with an Elastic IP | Outbound-only internet access for the application subnets |

## 4. Route Tables

| Route table | Destination | Target | Associated subnets |
|-------------|-------------|--------|--------------------|
| `public-rt` | `10.0.0.0/16` | local | `public-a`, `public-b` |
| | `0.0.0.0/0` | `secure-igw` | |
| `app-rt` | `10.0.0.0/16` | local | `app-a`, `app-b` |
| | `0.0.0.0/0` | `secure-nat` | |
| `db-rt` | `10.0.0.0/16` | local only | `db-a`, `db-b` |
| Main (default) | `10.0.0.0/16` | local | none (left unused) |

The main route table has no subnets associated, so any new subnet must be associated explicitly with the right table. This avoids accidentally exposing a new subnet.

## 5. Design Decisions & Trade-offs

| Decision | Reason | Trade-off |
|----------|--------|-----------|
| One NAT Gateway (in `public-a`) | Lower cost for a lab | Single point of failure. Production should use one NAT per AZ. |
| Database subnets with no internet route | The database never needs to initiate internet traffic | Patching/updates go through RDS maintenance, not the instance. |
| SSM Session Manager instead of a bastion | No inbound port, no keys to manage, every session is logged | Needs outbound HTTPS (443) from the instance through the NAT. |
| `/24` subnets | Simple to read and plenty for a lab | Production sizing depends on expected instance count. |
