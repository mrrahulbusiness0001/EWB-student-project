# AWS VPC Peering Project Report

**Student name:** [Enter your name]  
**Course / batch:** [Enter details]  
**AWS Region:** [Enter region]  
**Date:** [Enter date]

## 1. Abstract
This project demonstrates private IPv4 communication between two Amazon Virtual Private Clouds (VPCs) using VPC peering. A bastion host in a public subnet provides an administrative path to two private EC2 instances. Route tables and security groups control reachability.

## 2. Objectives
- Create two non-overlapping VPCs.
- Configure public and private subnets and explicit route table associations.
- Establish an active VPC peering connection.
- Configure forward and return routes.
- Restrict SSH and ICMP access to specified lab hosts.
- Validate private connectivity and demonstrate a controlled failure/recovery.
- Remove lab resources after collecting evidence.

## 3. Architecture and IP plan

| Resource | CIDR / IP | Purpose |
|---|---|---|
| VPC A | 10.10.0.0/16 | Bastion and App-A |
| Public subnet A | 10.10.1.0/24 | Bastion |
| Private subnet A | 10.10.2.0/24 | App-A |
| VPC B | 10.20.0.0/16 | App-B |
| Private subnet B | 10.20.1.0/24 | App-B |
| Bastion | 10.10.1.10 + public IPv4 | Administration |
| App-A | 10.10.2.10 | Private workload |
| App-B | 10.20.1.10 | Private workload |

Insert your architecture screenshot or diagram here.

## 4. Resources created
Fill in actual values from your AWS account. Do not invent resource IDs.

| Resource | Actual ID / value |
|---|---|
| VPC A ID | [Fill in] |
| VPC B ID | [Fill in] |
| Public subnet A ID | [Fill in] |
| Private subnet A ID | [Fill in] |
| Private subnet B ID | [Fill in] |
| Peering connection ID and status | [Fill in] |
| Bastion instance ID / public IP | [Fill in] |
| App-A instance ID / private IP | [Fill in] |
| App-B instance ID / private IP | [Fill in] |

## 5. Routing design
- Public route table: local VPC A route, `0.0.0.0/0` to the Internet Gateway, and `10.20.0.0/16` to the peering connection.
- VPC A private route table: local VPC A route and `10.20.0.0/16` to the peering connection.
- VPC B private route table: local VPC B route and `10.10.0.0/16` to the peering connection.

The return route is required because a successful request also needs a path back to the originating VPC. Peering does not automatically add routes.

## 6. Security design
- Bastion SSH (TCP 22): source restricted to the student's current public IPv4 `/32`.
- App-A SSH (TCP 22): source `10.10.1.10/32`.
- App-B SSH (TCP 22): source `10.10.1.10/32`.
- App-A ICMP echo request: source `10.20.1.10/32`.
- App-B ICMP echo request: source `10.10.2.10/32`.
- App-A and App-B have no public IPv4 addresses.

## 7. Test results
**Important:** Replace this table with actual observed results and attach screenshots. The expected outcomes below are not evidence that the project has been run.

| Test | Command / action | Expected outcome | Actual result |
|---|---|---|---|
| Baseline before peering/routes | From App-A: `ping -c 4 10.20.1.10` | No replies | [Record] |
| Forward path after setup | From App-A: `ping -c 4 10.20.1.10` | Replies | [Record] |
| Reverse path after setup | From App-B: `ping -c 4 10.10.2.10` | Replies | [Record] |
| SSH through bastion | `ssh ewb-app-b` from local SSH config | App-B shell | [Record] |
| Isolation | Inspect EC2 networking | No public IPv4 on App-A/App-B | [Record] |
| Controlled failure | Temporarily remove App-B inbound ICMP from App-A and ping again | Ping fails, then succeeds after rule restored | [Record] |

## 8. Troubleshooting notes
- If bastion SSH times out, check its public IP, source `/32`, public route table association, and Internet Gateway route.
- If private SSH fails, check key pair, username `ec2-user`, route tables, and source IP rules.
- If ping fails, confirm peering is active, both route directions exist, and the destination security group allows ICMP echo requests.
- Peering does not provide transitive routing or internet access to VPC B.

## 9. Conclusion
Summarize what you personally configured, what tests passed, what failed during troubleshooting, and what you learned. Only state outcomes supported by your own evidence.

## 10. Evidence checklist
- [ ] Architecture diagram
- [ ] VPC and subnet CIDRs
- [ ] Three route tables and associations
- [ ] Active peering connection
- [ ] Security group rules
- [ ] Actual baseline and successful ping output
- [ ] SSH to App-B through the bastion
- [ ] Controlled failure and recovery
- [ ] Cleanup evidence
