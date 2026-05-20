# Azure NSG & ASG Complete Guide with Practical Demos

> GitHub Ready Markdown Documentation  
> Step-by-step explanation with real practical examples.

---

# Table of Contents

1. What is NSG?
2. Why NSG is Required
3. Inbound vs Outbound Traffic
4. NSG Rule Components
5. NSG Default Rules
6. Subnet-Level NSG
7. NIC-Level NSG
8. Traffic Evaluation Flow
9. Stateful Nature of NSG
10. Practical Demo 1 – SSH Allow & Deny
11. Practical Demo 2 – NGINX Allow & Deny
12. Practical Demo 3 – Web VM to App VM Communication
13. What is ASG?
14. Why ASG is Required
15. NSG + ASG Combination
16. Practical Demo 4 – ASG + NSG
17. Important Interview Questions
18. Best Practices

---

# 1. What is NSG?

NSG (Network Security Group) is an Azure network-level firewall service.

Using NSG we can:

- Allow traffic
- Deny traffic
- Control inbound traffic
- Control outbound traffic
- Restrict ports
- Restrict source and destination

---

# 2. Real Life Example

Imagine a building security gate.

The security guard decides:

- Who can enter
- Who cannot enter
- Which gate is allowed
- Which timing is allowed

In Azure:

- VM = Building
- NSG = Security Guard

---

# 3. Inbound vs Outbound Traffic

## Inbound Traffic

Traffic coming from outside to VM.

Examples:

- SSH to Linux VM
- RDP to Windows VM
- Browser access to website

---

## Outbound Traffic

Traffic going from VM to outside.

Examples:

- apt update
- API calls
- Package download

---

# 4. NSG Rule Components

| Component | Description |
|---|---|
| Priority | Rule order |
| Source | Traffic source |
| Destination | Traffic destination |
| Source Port | Source port |
| Destination Port | Destination port |
| Protocol | TCP / UDP |
| Action | Allow / Deny |

---

# 5. NSG Default Rules

Azure automatically creates default rules.

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

# 6. Subnet-Level NSG

When NSG is attached at subnet level:

```text
Subnet → NSG
```

All VMs inside subnet inherit the rules.

---

# 7. NIC-Level NSG

When NSG is attached to NIC:

```text
VM NIC → NSG
```

Rules apply only to that VM.

---

# 8. Traffic Evaluation Flow

## Inbound Traffic

```text
Subnet NSG
    ↓
NIC NSG
```

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

# 9. NSG is Stateful

Azure NSG is stateful.

Meaning:

If inbound traffic is allowed:

```text
Return traffic is automatically allowed.
```

No separate outbound rule needed.

---

# 10. Practical Demo 1 – SSH Allow & Deny

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

Attach to:

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

# 11. Practical Demo 2 – NGINX Allow & Deny

## Step 1 – Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

---

## Step 2 – Open Browser

```text
http://PUBLIC-IP
```

Expected:

❌ Website not accessible

---

## Step 3 – Add HTTP Allow Rule

| Port | Action | Priority |
|---|---|---|
| 80 | Allow | 110 |

---

## Step 4 – Browser Test

Expected Result:

✅ NGINX Welcome Page opens

---

## Step 5 – Add HTTP Deny Rule

| Port | Action | Priority |
|---|---|---|
| 80 | Deny | 100 |

---

## Step 6 – Browser Test Again

Expected Result:

❌ Website blocked

---

# 12. Practical Demo 3 – Web VM to App VM Communication

## Architecture

```text
web-vm → web-subnet → web-nsg
app-vm → app-subnet → app-nsg
```

---

## Objective

Allow:

```text
web-vm → app-vm → Port 8080
```

---

## Step 1 – Create App Subnet

```text
app-subnet
```

CIDR:

```text
10.0.2.0/24
```

---

## Step 2 – Create app-nsg

Attach to:

```text
app-subnet
```

---

## Step 3 – Create app-vm

No public IP required.

---

## Step 4 – Run Python Test App

```bash
python3 -m http.server 8080
```

---

## Step 5 – Test From web-vm

```bash
curl http://APP-PRIVATE-IP:8080
```

Expected Result:

❌ Connection failed

---

## Step 6 – Add Inbound Rule in app-nsg

| Source | Destination Port | Action |
|---|---|---|
| web-subnet | 8080 | Allow |

---

## Step 7 – Test Again

```bash
curl http://APP-PRIVATE-IP:8080
```

Expected Result:

✅ Connection successful

---

## Step 8 – Change Rule to Deny

| Source | Destination Port | Action |
|---|---|---|
| web-subnet | 8080 | Deny |

---

## Step 9 – Test Again

```bash
curl http://APP-PRIVATE-IP:8080
```

Expected Result:

❌ Connection failed

---

# 13. What is ASG?

ASG = Application Security Group

ASG is NOT a firewall.

ASG only performs:

```text
Logical grouping of VMs
```

---

# 14. Why ASG is Required?

Without ASG:

We must create IP-based NSG rules.

Example:

| Source IP | Destination IP | Port |
|---|---|---|
| 10.0.1.4 | 10.0.2.4 | 8080 |
| 10.0.1.5 | 10.0.2.5 | 8080 |

---

# 15. NSG + ASG Combination

Instead of IP-based rules:

| Source | Destination | Port |
|---|---|---|
| web-asg | app-asg | 8080 |

---

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

# 16. Practical Demo 4 – ASG + NSG

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

### app-vm

```text
VM
→ Networking
→ Application Security Groups
→ Configure
→ app-asg
```

---

## Step 3 – Run Python Test App

```bash
python3 -m http.server 8080
```

---

## Step 4 – Create NSG Rule

| Field | Value |
|---|---|
| Source | Application Security Group |
| Source ASG | web-asg |
| Destination | Application Security Group |
| Destination ASG | app-asg |
| Destination Port | 8080 |
| Protocol | TCP |
| Action | Allow |
| Priority | 100 |

---

## Step 5 – Test Communication

```bash
curl http://APP-PRIVATE-IP:8080
```

Expected Result:

✅ Connection successful

---

## Step 6 – Change Rule to Deny

Expected Result:

❌ Connection failed

---
# Temporary Python HTTP Server for NSG Demo

## Objective

Run a temporary application on the App Server to test:

- NSG Allow Rule
- NSG Deny Rule
- Subnet-to-Subnet Communication
- ASG + NSG Communication

---

# Start Temporary Python Application

SSH into the App Server and run:

```bash
python3 -m http.server 9090
```

---

# Expected Output

```text
Serving HTTP on 0.0.0.0 port 9090
```

---

# What This Command Does

This command instantly creates a temporary web server on:

```text
Port 9090
```

No internet required.

No package installation required.

---

# Create Demo HTML File

Before starting the server, create a simple HTML page.

Run:

```bash
echo "<h1>Azure NSG Demo Successful</h1>" > index.html
```

---

# Test From Web VM

SSH into the Web VM and run:

```bash
curl http://APP-PRIVATE-IP:9090
```

Example:

```bash
curl http://10.0.2.4:9090
```

---

# Expected Result (Allow Rule)

✅ HTML response received

---

# Browser Test

Open browser:

```text
http://APP-PRIVATE-IP:9090
```

Example:

```text
http://10.0.2.4:9090
```

---

# Expected Result

✅ Webpage opens successfully

---

# NSG Allow Rule Example

Go to:

```text
app-nsg
→ Inbound Rules
→ Add
```

Create rule:

| Field | Value |
|---|---|
| Source | web-subnet |
| Destination Port | 9090 |
| Protocol | TCP |
| Action | Allow |
| Priority | 100 |

---

# Result

✅ Web VM can access App VM

---

# NSG Deny Rule Example

Modify rule:

| Field | Value |
|---|---|
| Action | Deny |

OR create higher-priority deny rule.

---

# Test Again

```bash
curl http://APP-PRIVATE-IP:9090
```

---

# Expected Result

❌ Connection failed

---

# Learning Outcome

Using this demo you can practically show:

- NSG traffic filtering
- Allow vs Deny behavior
- Subnet-level security
- East-West traffic control
- Application communication testing
- Real-time NSG rule impact

# 17. Important Interview Questions

## Q1 – What is NSG?

Azure network-level firewall service.

---

## Q2 – Is NSG stateful or stateless?

✅ Stateful

---

## Q3 – Difference between subnet NSG and NIC NSG?

Subnet NSG applies to all VMs in subnet.

NIC NSG applies to specific VM.

---

## Q4 – If subnet NSG denies traffic and NIC NSG allows it?

❌ Traffic blocked

---

## Q5 – Why use ASG?

To avoid complex IP-based NSG rules.

---

# 18. Best Practices

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
