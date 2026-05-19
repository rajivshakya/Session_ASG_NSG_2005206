# Azure NSG & ASG Complete Training Guide

> GitHub Ready Markdown Documentation  
> Beginner to Production-Level Understanding with Real-Time Demos

---

# Table of Contents

1. Introduction to NSG
2. Why NSG is Required
3. Azure Default Network Behavior
4. Inbound vs Outbound Traffic
5. NSG Rule Components
6. NSG Default Rules
7. Subnet-Level NSG
8. NIC-Level NSG
9. Traffic Evaluation Flow
10. Stateful Nature of NSG
11. Practical Demo 1 – SSH Allow & Deny
12. Practical Demo 2 – NGINX Allow & Deny
13. Understanding Default VNet Communication
14. Why NSG Is Still Required
15. Practical Demo 3 – East-West Traffic Restriction
16. Introduction to ASG
17. Why ASG is Required
18. NSG + ASG Combination
19. Practical Demo 4 – ASG + NSG
20. Important Interview Questions
21. Best Practices

---

# 1. Introduction to NSG

NSG (Network Security Group) is an Azure network-level firewall service.

Using NSG we can:

- Allow traffic
- Deny traffic
- Control inbound traffic
- Control outbound traffic
- Restrict ports
- Restrict source and destination

---

# 2. Why NSG is Required

The primary purpose of NSG is:

```text
Restrict unnecessary communication
```

NOT:

```text
Make connectivity work
```

Azure already allows some communication by default.

NSG is used to:

- Implement security
- Restrict east-west traffic
- Implement least privilege access
- Implement zero trust networking

---

# 3. Azure Default Network Behavior

This is a very important concept.

By default Azure allows:

```text
Communication between VMs inside same VNet
```

because of the default NSG rule:

| Priority | Rule | Action |
|---|---|---|
| 65000 | AllowVnetInBound | Allow |

---

# Important Observation

Suppose:

```text
web-vm → web-subnet
app-vm → app-subnet
```

Both are inside same VNet.

Even if port 9090 is NOT explicitly allowed:

```bash
curl http://10.0.2.4:9090
```

may still work.

Reason:

```text
AllowVnetInBound
```

default rule.

---

# Very Important

Azure default rules are created for:

- Easy deployment
- Easy initial communication
- Beginner-friendly setup

BUT:

Production companies NEVER keep everything open.

---

# 4. Inbound vs Outbound Traffic

## Inbound Traffic

Traffic coming from outside to VM.

Examples:

- SSH to Linux VM
- Browser access to website
- RDP to Windows VM

---

## Outbound Traffic

Traffic going from VM to outside.

Examples:

- apt update
- Package download
- API calls

---

# 5. NSG Rule Components

| Component | Description |
|---|---|
| Priority | Rule order |
| Source | Traffic source |
| Destination | Traffic destination |
| Source Port | Source port |
| Destination Port | Target port |
| Protocol | TCP / UDP |
| Action | Allow / Deny |

---

# Important

Lower priority number = Higher priority

Example:

| Priority | Action |
|---|---|
| 100 | Deny |
| 200 | Allow |

Result:

```text
Deny rule wins
```

---

# 6. NSG Default Rules

## Default Inbound Rules

| Priority | Rule | Action |
|---|---|---|
| 65000 | AllowVnetInBound | Allow |
| 65001 | AllowAzureLoadBalancerInBound | Allow |
| 65500 | DenyAllInBound | Deny |

---

## Default Outbound Rules

| Priority | Rule | Action |
|---|---|---|
| 65000 | AllowVnetOutBound | Allow |
| 65001 | AllowInternetOutBound | Allow |
| 65500 | DenyAllOutBound | Deny |

---

# 7. Subnet-Level NSG

When NSG is attached at subnet level:

```text
Subnet → NSG
```

All VMs inside subnet inherit the rules.

---

# Example

```text
web-subnet
 ├── web-vm1
 ├── web-vm2
 └── web-vm3
```

If subnet NSG allows:

- 80
- 443

then all VMs inside subnet allow HTTP and HTTPS.

---

# 8. NIC-Level NSG

When NSG is attached to NIC:

```text
VM NIC → NSG
```

Rules apply only to that VM.

---

# Example

```text
bastion-vm
```

Allow:

```text
22
```

Only Bastion VM gets SSH access.

---

# 9. Traffic Evaluation Flow

## Inbound Traffic

```text
Subnet NSG
    ↓
NIC NSG
```

---

## Outbound Traffic

```text
NIC NSG
    ↓
Subnet NSG
```

---

# Golden Rule

```text
Traffic is allowed only if BOTH NSGs allow it.
```

---

# Example

## Subnet NSG

| Port | Action |
|---|---|
| 22 | Deny |

---

## NIC NSG

| Port | Action |
|---|---|
| 22 | Allow |

---

## Result

❌ Traffic blocked

Reason:

Subnet-level deny stops traffic before NIC evaluation.

---

# 10. NSG is Stateful

Azure NSG is stateful.

Meaning:

If inbound traffic is allowed:

```text
Return traffic automatically allowed
```

No separate outbound rule required.

---

# 11. Practical Demo 1 – SSH Allow & Deny

## Objective

- Create VM
- Attach NSG
- Allow SSH
- Deny SSH
- Observe behavior

---

## Step 1 – Create Resource Group

```text
rg-nsg-demo
```

---

## Step 2 – Create VNet

```text
vnet-demo
```

Address Space:

```text
10.0.0.0/16
```

---

## Step 3 – Create Subnet

```text
web-subnet
```

CIDR:

```text
10.0.1.0/24
```

---

## Step 4 – Create NSG

```text
web-nsg
```

Attach NSG to:

```text
web-subnet
```

---

## Step 5 – Create Linux VM

```text
web-vm
```

Attach VM to:

```text
web-subnet
```

---

## Step 6 – Add SSH Rule

| Source | Port | Action | Priority |
|---|---|---|---|
| Any | 22 | Allow | 100 |

---

## Step 7 – SSH Test

```bash
ssh azureuser@PUBLIC-IP
```

Expected Result:

✅ SSH works

---

## Step 8 – Change Rule to Deny

| Source | Port | Action | Priority |
|---|---|---|---|
| Any | 22 | Deny | 100 |

---

## Step 9 – Test Again

```bash
ssh azureuser@PUBLIC-IP
```

Expected Result:

❌ Connection timeout

---

# 12. Practical Demo 2 – NGINX Allow & Deny

## Objective

- Install NGINX
- Allow HTTP
- Deny HTTP
- Observe browser behavior

---

## Step 1 – Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

---

## Step 2 – Browser Test

```text
http://PUBLIC-IP
```

Expected Result:

❌ Website not accessible

Reason:

Port 80 not allowed in NSG.

---

## Step 3 – Add HTTP Allow Rule

| Port | Action | Priority |
|---|---|---|
| 80 | Allow | 110 |

---

## Step 4 – Browser Test Again

Expected Result:

✅ NGINX Welcome Page opens

---

## Step 5 – Add Higher Priority Deny Rule

| Port | Action | Priority |
|---|---|---|
| 80 | Deny | 100 |

---

## Step 6 – Browser Test Again

Expected Result:

❌ Website blocked

---

# 13. Understanding Default VNet Communication

Suppose:

```text
web-vm → web-subnet
app-vm → app-subnet
```

Both subnets are inside same VNet.

---

# Run Temporary Python Application

SSH into app-vm and run:

```bash
echo "<h1>Azure NSG Demo Successful</h1>" > index.html
python3 -m http.server 9090
```

---

# Test From web-vm

```bash
curl http://APP-PRIVATE-IP:9090
```

Example:

```bash
curl http://10.0.2.4:9090
```

---

# Expected Result

✅ Connection successful

---

# Why Did This Work Without Any Custom Rule?

Because Azure default rule:

```text
AllowVnetInBound
```

already allows communication inside same VNet.

---

# Important Learning

Azure allows communication by default.

Production environments usually DO NOT allow unrestricted communication.

---

# 14. Why NSG Is Still Required

Suppose:

| VM | Role |
|---|---|
| web-vm | Web Server |
| app-vm | App Server |
| db-vm | Database |

---

# Azure Default Behavior

```text
web-vm ↔ app-vm
web-vm ↔ db-vm
app-vm ↔ db-vm
```

Everything allowed.

---

# Production Security Problem

If:

```text
web-vm hacked
```

attacker can:

- Reach database
- Scan internal network
- Spread malware
- Move laterally

---

# Production Goal

```text
Allow only required communication
```

Everything else:

```text
DENY
```

---

# This is called:

```text
Zero Trust Networking
```

---

# 15. Practical Demo 3 – East-West Traffic Restriction

## Objective

Show how NSG can restrict internal VNet communication.

---

## Current State

This works:

```bash
curl http://10.0.2.4:9090
```

because Azure default VNet traffic is allowed.

---

## Step 1 – Add Deny Rule in app-nsg

Go to:

```text
app-nsg
→ Inbound Rules
→ Add
```

---

## Create Rule

| Field | Value |
|---|---|
| Source | Any |
| Destination Port | 9090 |
| Protocol | TCP |
| Action | Deny |
| Priority | 100 |

---

## Step 2 – Test Again

```bash
curl http://10.0.2.4:9090
```

---

## Expected Result

❌ Connection failed

---

# Why?

Because:

```text
Custom deny rule overrides AllowVnetInBound
```

---

## Step 3 – Remove Deny Rule

Delete deny rule.

---

## Step 4 – Test Again

```bash
curl http://10.0.2.4:9090
```

---

## Expected Result

✅ Connection successful again

---

# Learning Outcome

Using this demo you can explain:

- Azure default connectivity
- Why NSG is required
- Real-time traffic restriction
- East-west traffic control
- Production security concepts

---

# 16. Introduction to ASG

ASG = Application Security Group

ASG is NOT a firewall.

ASG does NOT contain security rules.

ASG only performs:

```text
Logical grouping of VMs
```

---

# Example

| ASG | VMs |
|---|---|
| web-asg | web-vm1, web-vm2 |
| app-asg | app-vm1, app-vm2 |

---

# 17. Why ASG is Required

Without ASG:

We must create IP-based NSG rules.

Example:

| Source IP | Destination IP | Port |
|---|---|---|
| 10.0.1.4 | 10.0.2.4 | 8080 |
| 10.0.1.5 | 10.0.2.5 | 8080 |

Difficult to manage at scale.

---

# ASG Solution

Instead of IP-based rules:

| Source | Destination | Port |
|---|---|---|
| web-asg | app-asg | 8080 |

---

# Meaning

```text
All web VMs
can communicate with
all app VMs
on 8080
```

---

# 18. NSG + ASG Combination

## Traffic Flow

```text
web-vm
   ↓
web-asg
   ↓
NSG Rule
   ↓
app-asg
   ↓
app-vm
```

---

# Real Production Architecture

```text
Internet
   ↓
Web Tier
   ↓
App Tier
   ↓
Database Tier
```

---

# NSG Rules

| Source | Destination | Port |
|---|---|---|
| Internet | web-asg | 443 |
| web-asg | app-asg | 8080 |
| app-asg | db-asg | 3306 |

---

# 19. Practical Demo 4 – ASG + NSG

## Objective

Allow only:

```text
web-asg → app-asg → 9090
```

---

## Step 1 – Create ASGs

- web-asg
- app-asg

---

## Step 2 – Add VMs into ASGs

### web-vm

```text
VM
→ Networking
→ Application Security Groups
→ Configure
→ web-asg
```

---

### app-vm

```text
VM
→ Networking
→ Application Security Groups
→ Configure
→ app-asg
```

---

## Step 3 – Create NSG Rule

Go to:

```text
app-nsg
→ Inbound Rules
→ Add
```

---

## Create Rule

| Field | Value |
|---|---|
| Source | Application Security Group |
| Source ASG | web-asg |
| Destination | Application Security Group |
| Destination ASG | app-asg |
| Destination Port | 9090 |
| Protocol | TCP |
| Action | Allow |
| Priority | 100 |

---

## Step 4 – Test Communication

```bash
curl http://APP-PRIVATE-IP:9090
```

---

## Expected Result

✅ Connection successful

---

## Step 5 – Change Rule to Deny

Modify NSG rule action:

```text
Deny
```

---

## Step 6 – Test Again

```bash
curl http://APP-PRIVATE-IP:9090
```

---

## Expected Result

❌ Connection failed

---

# Learning Outcome

- ASG simplifies NSG management
- No need for IP-based rules
- Application-tier communication
- Scalable network security

---

# 20. Important Interview Questions

## Q1 – What is NSG?

Azure network-level firewall service.

---

## Q2 – Is NSG stateful or stateless?

✅ Stateful

---

## Q3 – Difference between subnet NSG and NIC NSG?

Subnet NSG applies to all VMs in subnet.

NIC NSG applies to a specific VM.

---

## Q4 – If subnet NSG denies traffic and NIC NSG allows it?

❌ Traffic blocked

---

## Q5 – Why use ASG?

To avoid complex IP-based NSG rules.

---

# 21. Best Practices

✅ Use subnet-level NSG for common rules

✅ Use NIC-level NSG only for exceptions

✅ Use ASG for scalable application-tier communication

✅ Restrict management ports

✅ Follow least privilege access

---

# Final Summary

## NSG

- Controls traffic
- Allow/Deny rules
- Stateful
- Works at subnet and NIC level

---

## ASG

- Logical grouping of VMs
- Simplifies NSG rules
- Removes IP dependency

---

# End of Documentation
# Azure NSG & ASG Interview Questions and Answers

---

# Q1. What is NSG in Azure?

NSG (Network Security Group) is an Azure network-level firewall service used to control inbound and outbound traffic.

Using NSG we can:

- Allow traffic
- Deny traffic
- Restrict ports
- Restrict source and destination communication

---

# Q2. Is NSG stateful or stateless?

✅ NSG is Stateful.

Meaning:

If inbound traffic is allowed, return traffic is automatically allowed.

No separate outbound rule is required for response traffic.

---

# Q3. At which levels can NSG be attached?

NSG can be attached at:

- Subnet level
- NIC level

---

# Q4. What is the difference between subnet-level NSG and NIC-level NSG?

| Subnet NSG | NIC NSG |
|---|---|
| Applies to all VMs inside subnet | Applies to specific VM |
| Used for common rules | Used for VM-specific rules |
| Easy centralized management | Fine-grained control |

---

# Q5. Can multiple NSGs be attached to one subnet?

❌ No

Only one NSG can be attached to one subnet.

---

# Q6. Can multiple NSGs be attached to one NIC?

❌ No

Only one NSG can be attached to one NIC.

---

# Q7. Which NSG is evaluated first during inbound traffic?

For inbound traffic:

```text
Subnet NSG
    ↓
NIC NSG
```

Subnet-level NSG is evaluated first.

---

# Q8. Which NSG is evaluated first during outbound traffic?

For outbound traffic:

```text
NIC NSG
    ↓
Subnet NSG
```

NIC-level NSG is evaluated first.

---

# Q9. If subnet NSG denies traffic and NIC NSG allows it, what will happen?

❌ Traffic will be blocked.

Because traffic must be allowed by BOTH NSGs.

---

# Q10. What is the purpose of NSG?

The main purpose of NSG is:

```text
Restrict unnecessary communication
```

NOT just enabling connectivity.

---

# Q11. What are Azure default NSG rules?

## Default Inbound Rules

| Priority | Rule | Action |
|---|---|---|
| 65000 | AllowVnetInBound | Allow |
| 65001 | AllowAzureLoadBalancerInBound | Allow |
| 65500 | DenyAllInBound | Deny |

---

## Default Outbound Rules

| Priority | Rule | Action |
|---|---|---|
| 65000 | AllowVnetOutBound | Allow |
| 65001 | AllowInternetOutBound | Allow |
| 65500 | DenyAllOutBound | Deny |

---

# Q12. Why does communication work inside same VNet without custom NSG rules?

Because Azure default rule:

```text
AllowVnetInBound
```

allows communication between resources inside same VNet.

---

# Q13. If Azure already allows internal traffic by default, why do we still need NSG?

Production environments do not trust default open communication.

NSG is used to:

- Restrict east-west traffic
- Implement least privilege access
- Prevent lateral movement
- Implement zero trust networking

---

# Q14. What is east-west traffic?

Communication between internal resources inside the same network or VNet is called east-west traffic.

Example:

```text
web-vm → app-vm
```

---

# Q15. What is north-south traffic?

Traffic between internet and internal infrastructure is called north-south traffic.

Example:

```text
Internet → web-vm
```

---

# Q16. What is the purpose of NSG priority?

Priority determines rule evaluation order.

Lower priority number = Higher priority.

Example:

| Priority | Rule | Action |
|---|---|---|
| 100 | Deny SSH | Deny |
| 200 | Allow SSH | Allow |

Result:

```text
Deny rule wins
```

---

# Q17. Can custom NSG rules override Azure default rules?

✅ Yes

Custom rules with higher priority override default rules.

Example:

| Priority | Rule | Action |
|---|---|---|
| 100 | Deny Port 22 | Deny |
| 65000 | AllowVnetInBound | Allow |

Result:

❌ Port 22 blocked

---

# Q18. What is a default deny model?

A security model where:

```text
First deny everything
Then allow only required traffic
```

This is a common production security practice.

---

# Q19. What is Zero Trust Networking?

A security model where:

```text
Trust nothing by default
```

Only explicitly allowed communication is permitted.

---

# Q20. What is ASG in Azure?

ASG (Application Security Group) is a logical grouping service for virtual machines.

ASG is NOT a firewall.

ASG does NOT contain security rules.

---

# Q21. Why is ASG required?

Without ASG:

We must create IP-based NSG rules.

Example:

| Source IP | Destination IP |
|---|---|
| 10.0.1.4 | 10.0.2.4 |

Difficult to manage at scale.

ASG removes dependency on IP addresses.

---

# Q22. What is the benefit of ASG?

Benefits of ASG:

- Simplified NSG management
- No IP-based rules
- Dynamic VM grouping
- Scalable security architecture

---

# Q23. How does ASG work with NSG?

ASG is used inside NSG rules.

Example:

| Source | Destination | Port |
|---|---|---|
| web-asg | app-asg | 8080 |

Meaning:

```text
All web VMs
can communicate with
all app VMs
on port 8080
```

---

# Q24. What is the difference between NSG and ASG?

| NSG | ASG |
|---|---|
| Firewall service | Logical grouping service |
| Controls traffic | Groups VMs |
| Contains allow/deny rules | Does not contain rules |

---

# Q25. What is micro-segmentation?

Restricting communication between application tiers using security policies.

Example:

```text
Web Tier → App Tier → DB Tier
```

Only required communication allowed.

Everything else denied.

---

# Q26. Explain a real production NSG architecture.

Example:

| Source | Destination | Port |
|---|---|---|
| Internet | Web Tier | 443 |
| Web Tier | App Tier | 8080 |
| App Tier | DB Tier | 3306 |
| Bastion | Servers | 22 |

Everything else:

```text
DENY
```

---

# Q27. What is the purpose of a Bastion Host?

Bastion Host provides secure administrative access to private servers.

Production environments usually allow SSH/RDP only through bastion servers.

---

# Q28. What happens if no NSG is attached?

Azure platform default behavior applies.

Internal VNet communication may still work because of Azure default rules.

---

# Q29. Can NSG filter outbound traffic?

✅ Yes

NSG can filter both:

- Inbound traffic
- Outbound traffic

---

# Q30. What is the best practice for production NSG design?

✅ Use subnet-level NSGs for common rules

✅ Use NIC-level NSGs only for exceptions

✅ Follow least privilege access

✅ Implement default deny model

✅ Allow only required communication

✅ Use ASG for scalable architecture

---

# End of Interview Questions
