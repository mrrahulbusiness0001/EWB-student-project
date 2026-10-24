# AWS VPC Peering Project — Private Connectivity

**Goal:** Build two non-overlapping VPCs connected by VPC peering, keep both application EC2 instances private, and administer them through a bastion host.

This starter kit follows the supplied EWB workbook's address plan:
- VPC A: `10.10.0.0/16`
- VPC B: `10.20.0.0/16`
- Public subnet A: `10.10.1.0/24`
- Private subnet A: `10.10.2.0/24`
- Private subnet B: `10.20.1.0/24`
- Bastion: `10.10.1.10`
- App-A: `10.10.2.10`
- App-B: `10.20.1.10`

## Important safety and cost notes
- This code creates billable AWS resources. Check pricing and credits before applying.
- Run it only in an AWS account you are authorized to use.
- The bastion is exposed to SSH from the single CIDR you provide. Use your current public IP as a `/32`, not `0.0.0.0/0`.
- EC2 instances, public IPv4, and EBS may incur charges. Destroy the lab when finished.
- Terraform does **not** create or download your private key. Create an EC2 key pair in AWS first and provide its name.
- This is a learning lab, not a production-ready design. For production, consider Systems Manager Session Manager, centralized logging, hardened images, and least-privilege egress.

## Prerequisites
1. An AWS account and credentials configured locally (`aws configure` or an AWS profile).
2. Terraform installed.
3. An EC2 key pair already created in the target region.
4. Your current public IPv4 CIDR, e.g. `203.0.113.25/32` (replace with your actual IP).

## Deploy
From this folder:

```bash
cd terraform
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars: set key_name and my_ip_cidr.
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Review the plan carefully before typing `yes`. The default region is Mumbai (`ap-south-1`). The AMI is looked up as the latest Amazon Linux 2023 x86_64 image available in that region.

## Connect
After apply, Terraform prints the bastion public IP and private IPs. Save the bastion public IP.

On Linux/macOS/WSL, use the matching private key file for the key pair you selected:

```bash
chmod 400 ~/.ssh/your-key.pem
ssh -i ~/.ssh/your-key.pem ec2-user@BASTION_PUBLIC_IP
```

From the bastion, you can SSH to the private hosts only if your key is available there. **Do not copy your private key onto the bastion.** Prefer SSH ProxyJump from your laptop instead:

```bash
ssh -i ~/.ssh/your-key.pem -J ec2-user@BASTION_PUBLIC_IP ec2-user@10.10.2.10
ssh -i ~/.ssh/your-key.pem -J ec2-user@BASTION_PUBLIC_IP ec2-user@10.20.1.10
```

If your local SSH client needs the private key specified separately for the jump host and destination, use an SSH config entry (see `ssh_config.example`). The destination security groups permit SSH from the bastion's private IP, as in the workbook. If ProxyJump authentication does not work with your SSH client/key setup, use AWS Systems Manager or follow your instructor's approved method; do not upload the private key.

## Validate the project
The Terraform code creates the network and security configuration, but it cannot claim tests have passed until you run them and capture your own output.

1. Confirm the peering connection is `Active`.
2. Confirm App-A and App-B have no public IPv4 address.
3. SSH through the bastion to App-A and App-B.
4. From App-A, run `ping -c 4 10.20.1.10`.
5. From App-B, run `ping -c 4 10.10.2.10`.
6. For a controlled failure, temporarily remove the App-B inbound ICMP rule allowing App-A (`10.10.2.10/32`), repeat the first ping, then restore the rule and test again.
7. Save screenshots of actual route tables, security groups, peering status, SSH sessions, and ping output.

Expected success criteria: each ping receives replies after routes and security rules are active. Actual results must be recorded from your own run.

## Key routing concept
VPC peering is not enough by itself. The private route table in VPC A needs `10.20.0.0/16 -> peering connection`, and the private route table in VPC B needs `10.10.0.0/16 -> peering connection`. The public route table also needs `10.20.0.0/16 -> peering connection` so the bastion can reach App-B. VPC peering is non-transitive and does not give VPC B internet access through VPC A's internet gateway.

## Cleanup
After saving evidence:

```bash
cd terraform
terraform destroy
```

Review the plan and confirm only this lab's resources will be removed. Check the AWS console for any remaining EBS volumes, Elastic IPs, or resources created manually. Usage/billing data can take time to update.

## Deliverables for a college report
- Project objective and architecture
- CIDR/subnet plan and resource inventory
- Screenshots of route table associations and routes
- Active peering status
- Security-group inbound rules
- Baseline ping failure before peering/routes (if following the manual workbook sequence)
- Forward and reverse ping results after setup
- SSH access to App-B through bastion
- Controlled failure and recovery
- Cleanup evidence and lessons learned

Do not present expected output as real output. Fill in your own AWS IDs, screenshots, date, and test results.
