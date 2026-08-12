# Network Architecture

## Overview

The infrastructure runs inside a dedicated Amazon VPC and spans two Availability Zones.

Each Availability Zone contains three network layers:

| Layer | Availability Zone 1 | Availability Zone 2 |
|---|---|---|
| Web | Public Web Subnet | Public Web Subnet |
| Application | Private App Subnet | Private App Subnet |
| Database | Private DB Subnet | Private DB Subnet |

This provides six subnets in total.

---

## Traffic Flow

### Internet to Web Tier

```text
Internet
    |
Internet Gateway
    |
Internet-facing ALB
    |
Web EC2 instances
```

Only the Internet-facing Application Load Balancer accepts incoming Internet traffic.

---

## Web to Application Tier

```text
Web EC2
    |
Internal ALB
    |
Application EC2
```

Application instances are not directly exposed to the Internet.

The Internal ALB provides a stable endpoint for communication between the Web and Application tiers.

---

## Application to Database Tier

```text
Application EC2
    |
    | MySQL :3306
    |
Aurora MySQL
```

Aurora only accepts database connections from the Application Tier Security Group.

---

## Private Subnet Internet Access

Application instances can initiate outbound Internet connections through a NAT Gateway.

```text
Private App EC2
      |
Private Route Table
      |
NAT Gateway
      |
Internet Gateway
      |
Internet
```

The NAT Gateway allows outbound connections without making the Application instances directly reachable from the Internet.

---

# Security Groups

Communication between layers is controlled using Security Groups.

| Security Group | Port | Source |
|---|---:|---|
| internet-alb-sg | 80 | 0.0.0.0/0 |
| web-tier-sg | 80 | internet-alb-sg |
| internal-alb-sg | 80 | web-tier-sg |
| app-tier-sg | 4000 | internal-alb-sg |
| db-tier-sg | 3306 | app-tier-sg |

This creates the following trust chain:

```text
Internet
   |
   v
internet-alb-sg
   |
   v
web-tier-sg
   |
   v
internal-alb-sg
   |
   v
app-tier-sg
   |
   v
db-tier-sg
```

The Application and Database tiers therefore remain isolated from direct Internet access.

---

# Administration

EC2 instances are administered using AWS Systems Manager Session Manager.

This avoids exposing SSH port 22 to the Internet.