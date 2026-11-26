# Week 1: Connectivity Test Results

**Date:** 2026-11-26 | **Region:** eu-central-1 | **Tester:** Mustafa

Tests were run from the EC2 instance through AWS Systems Manager Session Manager, and from a web browser. Status: ✅ passed, ⬜ still to run (fill in the result and attach the screenshot).

## 1. Results

| # | Test | Command / action | Expected | Result | Status |
|---|------|------------------|----------|--------|--------|
| 1 | Web page through the load balancer | Open `http://<ALB-DNS-name>` in a browser | Page "Secure AWS Network - app-server-a" | | ⬜ |
| 2 | Local web server | `curl localhost` (from the EC2 instance) | Same page | `<h1>Secure AWS Network - app-server-a</h1>` | ✅ |
| 3 | Outbound internet through the NAT Gateway | `curl https://ifconfig.me` | Returns the Elastic IP of `secure-nat` | | ⬜ |
| 4 | Outbound HTTP (port 80) is blocked | `curl ifconfig.me` | Request hangs (timeout), because `app-sg` allows only 443 and 3306 outbound | The command hung and had to be interrupted | ✅ |
| 5 | Database connectivity from the app tier | `mysql -h <RDS-endpoint> -u admin -p`, then `SHOW DATABASES;` | Connection succeeds, `appdb` is listed | | ⬜ |
| 6 | Load balancer health check | Target groups > `app-tg` > Targets | Status **Healthy** | | ⬜ |
| 7 | EC2 has no public address | Instance details | Public IPv4 address is empty | | ⬜ |
| 8 | Database is not public | RDS > `secure-db` > Connectivity | Publicly accessible: **No** | | ⬜ |
| 9 | Re-test after NACLs were applied | Repeat tests 1, 3, 5 and 6 | Same results as before the NACLs | | ⬜ |

## 2. Evidence

Add screenshots here (blur or crop the AWS Account ID before uploading):

- Test 1: browser showing the ALB page
- Test 3: terminal showing the IP, next to the NAT Gateway details with the same Elastic IP
- Test 5: `SHOW DATABASES;` output
- Test 6: target group with a Healthy target
- Test 7 and 8: EC2 details and RDS connectivity page

## 3. Interpretation

- **Tests 1 and 6** show the full public path works: Internet, Internet Gateway, ALB, `app-sg`, EC2.
- **Test 3** shows a private server with no public IP reaches the internet only through the NAT Gateway, and that the returned address is the NAT's Elastic IP, not the instance's.
- **Test 4** is a deliberate negative test. The same request over port 80 fails, proving outbound traffic is restricted to what is needed.
- **Test 5** shows the application tier can reach the database on port 3306, while **test 8** and the route table (`db-rt` has no internet route) show the database cannot be reached from outside.
- **Test 9** confirms the Network ACLs, including the ephemeral-port rules for reply traffic, do not break any allowed path.
