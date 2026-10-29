# AWS VPC Peering with Private EC2 Connectivity

A hands-on AWS networking project demonstrating secure communication between two Amazon Virtual Private Clouds (VPCs) using VPC Peering, private EC2 instances, and a bastion host with SSH ProxyJump.

##  Project Overview

This project establishes private network connectivity between two VPCs in the AWS Sydney Region (`ap-southeast-2`). VPC Peering enables resources in both VPCs to communicate using private IPv4 addresses without requiring public IP addresses on the private EC2 instances.

A bastion host in the public subnet of VPC A provides secure SSH access to the private instances in both VPCs.

##  Architecture

The infrastructure consists of two VPCs connected through an active VPC Peering connection.

### Network Configuration

| Component | Configuration |
|---|---|
| AWS Region | Asia Pacific (Sydney) — `ap-southeast-2` |
| VPC A | `ewb-vpc-a` |
| VPC A CIDR | `10.10.0.0/16` |
| Public Subnet A | `10.10.1.0/24` |
| Private Subnet A | `10.10.2.0/24` |
| VPC B | `ewb-vpc-b` |
| VPC B CIDR | `10.20.0.0/16` |
| Private Subnet B | `10.20.1.0/24` |
| VPC Peering | `ewb-peering-a-b` |
| Bastion Host | `ewb-bastion-a` |
| Private Instance A | `ewb-private-a` |
| Private Instance B | `ewb-private-b` |

### Network Diagram

Add your architecture diagram to the repository's `images` folder and name it `architecture.png`.

```text
                         Windows PowerShell
                                  |
                          SSH / ProxyJump
                                  |
                                  v
                 +-----------------------------+
                 | VPC A: 10.10.0.0/16         |
                 |                             |
                 |  Public Subnet              |
                 |  10.10.1.0/24               |
                 |  +----------------------+   |
                 |  | Bastion EC2          |   |
                 |  +----------------------+   |
                 |             |               |
                 |  Private Subnet             |
                 |  10.10.2.0/24               |
                 |  +----------------------+   |
                 |  | Private EC2 A        |   |
                 |  +----------------------+   |
                 +-------------+---------------+
                               |
                         VPC Peering
                               |
                 +-------------+---------------+
                 | VPC B: 10.20.0.0/16         |
                 |                             |
                 |  Private Subnet             |
                 |  10.20.1.0/24               |
                 |  +----------------------+   |
                 |  | Private EC2 B        |   |
                 |  +----------------------+   |
                 +-----------------------------+
```

![AWS VPC Peering Architecture](images/architecture.png)

## ⚙️ AWS Services Used

- **Amazon VPC** — Isolated virtual networks and subnet configuration.
- **Amazon EC2** — Bastion host and private compute instances.
- **VPC Peering** — Private IPv4 communication between VPC A and VPC B.
- **Internet Gateway** — Internet connectivity for the public subnet in VPC A.
- **Route Tables** — Routing between subnets and across the peering connection.
- **Security Groups** — Controls for SSH and ICMP traffic.
- **Amazon Linux 2023** — Operating system used by the EC2 instances.
- **OpenSSH ProxyJump** — Secure access to private instances through the bastion host.

##  Implementation Steps

### 1. Create the VPCs and Subnets

Created two VPCs with non-overlapping CIDR blocks:

- VPC A: `10.10.0.0/16`
- VPC B: `10.20.0.0/16`

Configured one public subnet and one private subnet in VPC A, along with a private subnet in VPC B.

### 2. Configure Internet Access for the Bastion

Attached an Internet Gateway to VPC A and configured the public route table with a default route to the Internet Gateway.

The bastion host is deployed in the public subnet. The private EC2 instances do not have public IPv4 addresses.

### 3. Establish VPC Peering

Created and activated the VPC Peering connection between VPC A and VPC B.

### 4. Configure Route Tables

Added routes for cross-VPC communication:

- VPC A public route table: `10.20.0.0/16` through the peering connection.
- VPC A private route table: `10.20.0.0/16` through the peering connection.
- VPC B route table: `10.10.0.0/16` through the peering connection.

The VPC-local routes remain in place.

### 5. Configure Security Groups

Configured security groups to permit the required traffic:

- SSH access to the bastion host from an authorized source.
- SSH access to the private instances through the bastion.
- ICMP traffic between the private instances for connectivity testing.
- Outbound traffic as required for the tests.

Security group rules were used to demonstrate how traffic filtering affects connectivity.

### 6. Configure SSH ProxyJump

Configured SSH aliases on the local Windows machine to connect to the private instances through the bastion host.

Example configuration:

```sshconfig
Host ewb-bastion
    HostName <BASTION_PUBLIC_IP>
    User ec2-user
    IdentityFile "<PATH_TO_PRIVATE_KEY>"
    IdentitiesOnly yes

Host ewb-app-a
    HostName 10.10.2.209
    User ec2-user
    IdentityFile "<PATH_TO_PRIVATE_KEY>"
    IdentitiesOnly yes
    ProxyJump ewb-bastion

Host ewb-app-b
    HostName 10.20.1.241
    User ec2-user
    IdentityFile "<PATH_TO_PRIVATE_KEY>"
    IdentitiesOnly yes
    ProxyJump ewb-bastion
```

Replace the placeholders with your current bastion public IP and local private-key path.

Connect to the instances from Windows PowerShell:

```bash
ssh ewb-bastion
ssh ewb-app-a
ssh ewb-app-b
```

The private instance aliases use ProxyJump to route SSH connections through the bastion host.

##  Testing and Validation

The project includes the following connectivity and security tests.

| Test | Result |
|---|---|
| SSH connection to Private EC2 A through the bastion | Successful |
| SSH connection to Private EC2 B through the bastion | Successful |
| Ping from Private A to Private B | Successful |
| Ping from Private B to Private A | Successful |
| Ping after temporarily removing the required ICMP rule | Failed as expected |
| Ping after restoring the ICMP rule | Successful |

### Connectivity Commands

From Private EC2 A:

```bash
ping -c 4 10.20.1.241
```

From Private EC2 B:

```bash
ping -c 4 10.10.2.209
```

The tests demonstrate bidirectional private communication across VPC Peering and the impact of security group rules on ICMP traffic.

##  Security Considerations

- Private EC2 instances do not require public IP addresses.
- SSH access to private instances is routed through the bastion host.
- Security groups restrict inbound traffic to the required sources and ports.
- SSH private keys should remain private and must never be committed to GitHub.
- Public IP addresses and local machine paths should be reviewed before publishing screenshots.
- The temporary ICMP rule removal was used only to demonstrate a controlled connectivity failure; the rule was subsequently restored.

##  Project Evidence

The `screenshots` folder contains the project evidence, including:

- VPC and subnet configurations.
- Route table configurations.
- Active VPC Peering connection.
- Security group rules.
- EC2 instance details.
- Successful SSH connections to both private instances.
- Bidirectional ping tests.
- Controlled ICMP failure and recovery.
- Restored security group rule.

##  Key Learnings

- Configuring VPCs, subnets, and route tables in AWS.
- Establishing private communication through VPC Peering.
- Understanding routing and security group behavior.
- Accessing private EC2 instances through a bastion host.
- Configuring SSH ProxyJump on Windows.
- Troubleshooting network connectivity through controlled failure testing.
- Validating secure communication between isolated cloud networks.

##  Future Improvements

- Automate infrastructure provisioning with Terraform or AWS CloudFormation.
- Implement centralized monitoring and logging.
- Explore AWS Systems Manager Session Manager as an alternative to direct SSH access.
- Add automated network connectivity checks.

##  Author

Sreehari K

Computer Science and Engineering Graduate

Project: AWS VPC Peering and Private EC2 Connectivity
