# AWS Three-Tier Architecture

**Cloud infrastructure | AWS networking | EC2 | Amazon RDS (MariaDB) | Security groups**

## Project Overview

This project documents the deployment of an AWS three-tier infrastructure using separate presentation, application, and data layers. The implementation includes an Amazon VPC, public and private subnets, routing components, a bastion host, EC2 web and application servers, and a private Amazon RDS MariaDB database.

The primary focus is **network segmentation, access control, secure administration, and connectivity testing**. The provided evidence confirms infrastructure configuration and selected network/database connectivity tests; it does **not** yet demonstrate a complete user-facing application request flowing through all three tiers.

## Architecture

| Tier | Component | Role |
| --- | --- | --- |
| **Tier 1 — User / Presentation** | EC2 Web Server | Web-facing server hosted in the public subnet |
| **Tier 2 — Application** | EC2 App Server | Private EC2 instance used for application-tier access and database connectivity |
| **Tier 3 — Data** | Amazon RDS MariaDB | Managed relational database in private database subnets |
| **Administration** | EC2 Bastion Host | SSH entry point for administering private instances |

### Logical layout

```text
                         Users / Internet
                                |
                         Internet Gateway
                                |
                     VPC: 192.168.0.0/16
                                |
             +------------------+------------------+
             |                                     |
    Public subnet                             Private subnets
    192.168.1.0/24                           192.168.2.0/24
      |                                      192.168.3.0/24
      +-- Bastion Host                       192.168.4.0/24
      +-- Web Server                              |
      +-- NAT Gateway (Elastic IP)                +-- App Server
             |                                     +-- RDS DB subnet group
             +---- outbound access for             |    (two private subnets)
                   private subnet routes           +-- RDS MariaDB
```

> This is a **simplified logical diagram**. Refer to the source architecture diagram and AWS console screenshots for the specific deployed placement and associations.

**Architecture evidence:** See page 1 of `AWS Three Tier Architecture.pdf`.

## AWS Resources and Network Design

### VPC and subnets

The screenshots document a VPC with CIDR **`192.168.0.0/16`** and four subnets:

| Subnet | CIDR | Availability Zone relationship | Purpose / evidence |
| --- | --- | --- | --- |
| `public-subnet` | `192.168.1.0/24` | Same AZ as private-subnet1 and private-subnet2 | Bastion host and web server |
| `private-subnet1` | `192.168.2.0/24` | Same AZ as public subnet | Application server |
| `private-subnet2` | `192.168.3.0/24` | Same AZ as public subnet | RDS DB subnet group member |
| `private-subnet3` | `192.168.4.0/24` | Different AZ | RDS DB subnet group member |

The PDF records enabling automatic public IPv4 address assignment for the public subnet. See pages 2–3.

### Internet and NAT connectivity

- **Internet Gateway:** Created and attached to the VPC.
- **Elastic IP:** Allocated for use with a public NAT Gateway.
- **NAT Gateway:** Created in the public subnet.
- **Public route table:** Includes `0.0.0.0/0` targeting the Internet Gateway; the public subnet is associated with this table.
- **Private route table:** Includes `0.0.0.0/0` targeting the NAT Gateway; the screenshots show associations with the three private subnets.

**Evidence:** Pages 4–6 of the PDF.

> **Design consideration:** The documented private route table also associates database-related private subnets with the NAT route. For a more restrictive future design, consider a dedicated database route table without a default internet route, if the database workloads do not require outbound internet access.

## Security Groups

The project includes four named security groups:

1. **BastionHost-SG** — administrative access to the bastion instance.
2. **WebServer-SG** — rules for web-server traffic and administration.
3. **AppServer-SG** — rules for application-tier traffic and administration.
4. **DatabaseServer-SG** — database connectivity rules.

The PDF shows security-group configurations on pages 7–8. The database group includes **MySQL/Aurora port 3306** rules; this AWS rule label is also used for MariaDB-compatible traffic.

For the repository, explain each rule's **protocol, port, source security group or CIDR, and reason for access**. Avoid broad SSH exposure when possible. Review and tighten any permissive rules before treating this as a production reference architecture.

> The README does not claim that every security-group rule follows least privilege: the uploaded screenshots show the configuration, but no security audit or hardening validation is included.

## EC2 Instances

Three EC2 instances were created:

| Instance | Tier / purpose | Network placement |
| --- | --- | --- |
| **Bastion Host** | SSH jump host | Public subnet |
| **Web Server** | Presentation tier | Public subnet |
| **App Server** | Application tier | Private subnet |

The instance screenshots show the instances running. See pages 9–10.

The SSH terminal evidence uses **Amazon Linux 2023**.

## Amazon RDS — Data Tier

The deployment includes:

- An **RDS DB subnet group** containing two private subnets in different Availability Zones.
- An **Amazon RDS MariaDB** instance.
- A database endpoint on **port 3306**.
- **Public accessibility disabled**, as shown in the database networking screenshot.
- A database security group attached to the RDS instance.

**Evidence:** Pages 10–11.

> The screenshot demonstrates a single available database instance; it does not establish a Multi-AZ database deployment. Having DB subnets in multiple Availability Zones is not, by itself, proof of Multi-AZ failover configuration.

## Implementation Walkthrough

The following sequence reflects the deployment stages shown in the uploaded documentation.

### 1. Create the VPC and subnets

- Create the VPC and configure its CIDR block.
- Create one public subnet and three private subnets.
- Configure automatic public IPv4 assignment for the public subnet.
- Place the third private subnet in a different Availability Zone.

**Evidence:** PDF pages 1–3.

### 2. Configure network gateways and routing

- Allocate an Elastic IP.
- Create the NAT Gateway in the public subnet.
- Create and attach an Internet Gateway.
- Create a public route table, add a default route to the Internet Gateway, and associate the public subnet.
- Create a private route table, add a default route to the NAT Gateway, and associate the private subnets.

**Evidence:** PDF pages 4–6.

### 3. Configure security groups

Create dedicated security groups for the bastion host, web server, application server, and database server. Document allowed traffic and the purpose of each rule.

**Evidence:** PDF pages 7–8.

### 4. Deploy EC2 instances

Launch the bastion host and web server in the public subnet, and the application server in a private subnet.

**Evidence:** PDF pages 9–10.

### 5. Deploy the database

Create the RDS DB subnet group using private subnets across two Availability Zones, then deploy the RDS MariaDB instance with private networking and the database security group.

**Evidence:** PDF pages 10–11.

## Connectivity Tests and Results

The uploaded terminal screenshots support the following results:

| Test | Result documented | Evidence |
| --- | --- | --- |
| SSH from local Windows terminal to bastion host | **Successful** — Amazon Linux 2023 shell displayed | PDF page 12 |
| SSH from bastion host to private application server | **Successful** — application-server shell displayed | PDF page 12 |
| ICMP ping from application server to web server (`192.168.1.82`) | **Successful** — 6 packets transmitted, 6 received, 0% packet loss | PDF page 13 |
| Database client connection from application server to RDS | **Successful** — MariaDB prompt reached and `SHOW DATABASES;` returned results | PDF page 13 |

### SSH access through the bastion host

The screenshots show Windows SSH key-permission adjustments followed by SSH access to the bastion, then a second SSH connection from the bastion to the application server.

For documentation, use generic placeholders rather than publishing actual private keys:

```bash
ssh -i /path/to/private-key.pem ec2-user@<BASTION_PUBLIC_IP>
# From the bastion host, using an authorized key:
ssh -i /path/to/private-key.pem ec2-user@<APP_PRIVATE_IP>
```

**Security note:** Do not commit `.pem` or `.ppk` private keys. Consider SSH agent forwarding, EC2 Instance Connect, or AWS Systems Manager Session Manager instead of storing private keys on the bastion.

### Application-to-web connectivity

The screenshot records a ping from the application server to the web server at `192.168.1.82` with **0% packet loss** across six packets. This demonstrates ICMP reachability, **not** that an HTTP application request succeeded.

### Application-to-database connectivity

The screenshot shows a MariaDB client connection to the RDS endpoint and a successful `SHOW DATABASES;` query. This confirms that the application-server environment was able to establish a database session and execute a read-only metadata query.

Do not publish the database password, connection secrets, or sensitive endpoint details in documentation.

## What Has and Has Not Been Verified

### Verified in the supplied evidence

- VPC and subnet creation
- Internet Gateway, NAT Gateway, Elastic IP, and route-table setup
- Four security groups created
- Bastion, web, and application EC2 instances running
- RDS DB subnet group and MariaDB instance available
- Bastion-to-application SSH access
- Application-to-web ICMP connectivity
- Application-to-RDS database connection and query

### Not yet demonstrated in the supplied evidence

- A browser successfully loading the deployed web application
- HTTP or HTTPS communication from the web tier to an application service
- A user request traversing **presentation → application → database** end to end
- Load balancing, Auto Scaling, or high availability
- Automated deployment with Infrastructure as Code
- Database failover or backup-restore testing

## Challenges and Lessons Learned

The screenshots show troubleshooting around **Windows SSH private-key permissions** before successful bastion access. The documented connectivity checks also demonstrate practical validation of private network reachability and database access.

Key skills illustrated by the project:

- Designing VPC CIDR blocks and subnet boundaries
- Separating public and private resources
- Configuring gateway routes and subnet associations
- Using security groups to govern cross-tier communication
- Accessing private EC2 instances through a bastion host
- Testing ICMP reachability and MariaDB connectivity

## Suggested Next Steps

1. Deploy or document an actual web application that calls the application tier and queries the database.
2. Capture a browser screenshot and application logs proving an end-to-end request.
3. Review security-group inbound rules and restrict sources to the minimum required.
4. Consider a separate database route table without a NAT default route.
5. Add monitoring and logs (for example, CloudWatch and VPC Flow Logs).
6. Document backup and recovery, and consider Multi-AZ and load balancing as future enhancements.
7. Recreate the infrastructure using Terraform or AWS CloudFormation for repeatability.

## Repository Structure

```text
AWS-Three-Tier-Architecture/
├── README.md
├── diagrams/
│   └── aws-three-tier-architecture.png
├── screenshots/
│   ├── 01-vpc-and-subnets.png
│   ├── 02-gateways.png
│   ├── 03-route-tables.png
│   ├── 04-security-groups.png
│   ├── 05-ec2-instances.png
│   ├── 06-rds.png
│   ├── 07-ssh-access.png
│   └── 08-connectivity-tests.png
└── docs/
    └── AWS-Three-Tier-Architecture.pdf
```

**Note:** The proposed image filenames above are a suggested organization, not files already present in the repository. Export or crop relevant screenshots from your PDF before adding image references to this README.

## Evidence Source

All deployment and testing claims in this README are based on the supplied **13-page `AWS Three Tier Architecture.pdf`**. The original PDF contains AWS Console screenshots and terminal output; review and redact public IP addresses, account identifiers, database endpoints, and other sensitive details before publishing it publicly.

---

**Project status:** Infrastructure deployed; selected network and database connectivity verified. Full end-to-end application behavior not yet documented.
