# AWS-Three-Tier-Architecture

## Project Overview
Designed and deployed a three-tier architecture on AWS,
separating the presentation, application, and database layers
using Amazon VPC, EC2, and RDS.

## Architecture
- **Tier 1 – Presentation:** EC2 Web Server
- **Tier 2 – Application:** EC2 Application Server
- **Tier 3 – Data:** Amazon RDS MariaDB

The infrastructure includes a bastion host, public and
private subnets, an Internet Gateway, NAT Gateway,
route tables, and security groups.

## AWS Services Used
- Amazon VPC
- Amazon EC2
- Amazon RDS (MariaDB)
- Internet Gateway
- NAT Gateway
- Security Groups
- Elastic IP

## Implementation
1. Created a VPC with public and private subnets.
2. Configured Internet Gateway, NAT Gateway, and routing.
3. Created security groups for each infrastructure component.
4. Deployed EC2 instances for the bastion host, web server,
   and application server.
5. Created an Amazon RDS MariaDB database.

## Testing & Validation
- Successfully connected to the bastion host using SSH.
- Accessed the private application server through the bastion.
- Verified connectivity from the application server to the
  web server using ping.
- Successfully connected to Amazon RDS MariaDB and executed
  the SHOW DATABASES command.

## Key Learnings
- AWS VPC networking and subnet configuration
- Network routing and security groups
- Linux administration and SSH access
- EC2-to-RDS database connectivity
- Troubleshooting cloud infrastructure
