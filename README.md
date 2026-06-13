# AWS VPC ( Custom Network Setup )

## Project Overview
This project demonstrates how to set up a secure Amazon VPC (Virtual Private Cloud) with public and private subnets, routing, and internet access. It provides a hands-on understanding of how AWS networking components work together to host EC2 instances in both public and private environments.

---

## Architecture Components
- VPC (Virtual Private Cloud): Custom VPC created with a defined CIDR block (e.g., 10.0.0.0/16).
- Subnets:
  - Public Subnet: For hosting resources accessible from the internet (e.g., web servers).
  - Private Subnet: For internal resources (e.g., databases, application servers).
- Route Tables:
  - Public Route Table -> Associated with the public subnet and routes traffic via the Internet Gateway.
  - Private Route Table -> Associated with the private subnet and routes traffic via the NAT Gateway.
- Internet Gateway (IGW): Provides internet connectivity for the public subnet.
- NAT Gateway: Allows private subnet resources to access the internet (for updates, patching) without exposing them directly.
- EC2 Instances:
  - Public EC2 instance (e.g., a web server).
  - Private EC2 instance (e.g., application/database server).

---

## Steps to Implement

### 1. Create a VPC
- Go to AWS Console -> VPC -> Create VPC.
- Configure your setting selecting VPC only, give it a name, and set the CIDR Block to 10.0.0.0/16.

![VPC Dashboard Setup](images/screenshot1.png)
![Specify VPC Name and CIDR Block](images/screenshot2.png)

---

### 2. Create Subnets
- Create a Public Subnet with CIDR block 10.0.1.0/24 in your chosen availability zone.

![Create Subnet Option](images/screenshot3.png)
![Configure Public Subnet Form](images/screenshot4.png)

- Create a Private Subnet with CIDR block 10.0.2.0/24 in the same availability zone.

![Configure Private Subnet Settings](images/screenshot5.png)
![Review Configured Subnets List](images/screenshot6.png)

---

### 3. Create and Attach Internet Gateway
- Navigate to Internet Gateways -> Create internet gateway. Give it a descriptive name tag.

![Create Internet Gateway View](images/screenshot7.png)

- Once created, note that its initial state is detached.

![Internet Gateway Detached Status Display](images/screenshot8.png)

- Select Actions -> Attach to VPC, choose your custom VPC from the drop-down list, and confirm the attachment.

![Select Attach to VPC Option](images/screenshot9.png)
![Confirm VPC Attachment Panel](images/screenshot10.png)

---

### 4. Create Route Tables
- Go to Route Tables -> Create route table. Create an explicit Route Table for your public traffic.

![Create Public Route Table Form](images/screenshot11.png)

- Create another standalone Route Table for your private traffic paths.

![Create Private Route Table Form](images/screenshot12.png)
![Review Route Tables Dashboard Grid](images/screenshot13.png)

---

### 5. Configure Public Routes and Subnet Associations
- Select your Public Route Table and go to Subnet associations. Click Edit subnet associations, check the box next to your Public Subnet, and save the settings.

![Edit Public Subnet Associations Grid](images/screenshot14.png)
![Confirm Associated Public Subnet View](images/screenshot15.png)

- Go to the Routes tab and click Edit routes. Add a route target of 0.0.0.0/0 pointing directly to your Internet Gateway to enable bidirectional internet access.

![Add Internet Route to Public Table](images/screenshot16.png)

---

### 6. Create NAT Gateway for Private Outbound Routing
- Go to NAT Gateways -> Create nat gateway. 
- Provide a name, select your Public Subnet (since the NAT Gateway must sit in a public space with a public interface to hit the web), select Public connectivity type, and click Allocate Elastic IP to bind a persistent public IP address.

![Create NAT Gateway Configuration Window](images/screenshot17.png)
![NAT Gateway Form Allocation Fields](images/screenshot18.png)

---

### 7. Configure Private Routes and Subnet Associations
- Select your Private Route Table and go to Subnet associations. Click Edit subnet associations, select your Private Subnet, and save.

![Edit Private Subnet Associations Grid](images/screenshot19.png)

- Go to the Routes tab and select Edit routes. Add a default route targeting 0.0.0.0/0 and direct its target destination to your newly created NAT Gateway. This lets resources inside your private subnet communicate securely outbound via your public tier.

![Add NAT Gateway Route to Private Table](images/screenshot20.png)

---

### 8. Modify Subnet IP Auto-Assignment and Firewall Rules
- Select your Public Subnet, go to Actions -> Edit subnet settings, check the box to Enable auto-assign public IPv4 address, and save. This ensures any EC2 machine dropped here gets an external identity automatically.

![Enable Auto Assign Public IP in Settings](images/screenshot21.png)

- Keep auto-assign public IP address disabled for your Private Subnet to ensure zero direct accessibility from external scanners.
- Go to EC2 -> Security Groups -> Edit inbound rules. Modify your security group configurations to allow the ICMP protocol from anywhere (0.0.0.0/0) so you can execute end-to-end network ping validation checks.

![Configure Security Group Inbound ICMP Rules](images/screenshot22.png)
![Save Inbound Security Firewall Rule Changes](images/screenshot23.png)

---

### 9. Launch EC2 Instances to Test Path Routing
- Launch a virtual machine host inside your Public Subnet. View its parameters in the console grid to verify it holds both a public IP and a separate internal private IP address.

![Launch Wizard Subnet Placement Options](images/screenshot24.png)
![Public Subnet EC2 Instance Details Display](images/screenshot25.png)

- Launch a separate virtual machine instance into your Private Subnet. Review its metadata to confirm it has an internal network address but completely lacks a public IPv4 address field.

![Launch Wizard Private Placement Setup](images/screenshot26.png)
![Private Subnet EC2 Instance Details Display](images/screenshot27.png)

---

### 10. Perform Connection and Connectivity Tests
- Open a terminal session on your physical workstation machine and ping the public IP address of your public web server instance. The packets return successfully.

![Local Workstation Ping Test to Public Host Terminal](images/screenshot28.png)

- Use your terminal command prompt to establish a secure SSH connection straight into the public EC2 machine.

![SSH Command Prompt into Public Server Host](images/screenshot29.png)

- Once logged onto the command shell of your public EC2 machine, execute a ping command directed at the internal private IP address of your backend private EC2 instance. The network path resolves natively within the VPC network.

![Internal VPC Ping from Public to Private Host](images/screenshot30.png)

- Execute an SSH hop or tunnel connection from the public host terminal to securely log into the private subnet instance command shell.

![Establish SSH Session on Remote Private Machine](images/screenshot31.png)
![Confirm Successful Shell Landing on Private Host](images/screenshot32.png)

- Run a standard query test command inside the private machine shell (such as pinging an external domain or service). The connection completes successfully. This verifies that your NAT Gateway route configuration is actively translating and proxying outbound internet traffic.

![Outbound Web Query Ping from Isolated Private Host](images/screenshot33.png)
![Verify DNS Resolution on Private Machine Terminal](images/screenshot34.png)

---

### 11. Final Verification Checks
- Check the internal system configurations and network adapters on both machines to confirm routing parameters are fully initialized.

![Check Private Instance Operating System Properties](images/screenshot35.png)
![Verify Subnet Adapter Metrics inside Terminal Session](images/screenshot36.png)
![Complete End to End Traffic Routing Verification Check](images/screenshot37.png)
![Final Active Network Mesh Resource Overview](images/screenshot38.png)

---

## Benefits
- Network Level Security Isolation: Backend database tiers are protected from raw internet exposure by lack of direct public addresses.
- Controlled Updates: Private workloads safely fetch security patches and system updates over a one-way outbound NAT proxy.
- Highly Custom Routing Architecture: Explicit management of traffic paths through standalone routing rules per application layer.

## Key Learnings
- Subnet Segmentation: Designing and defining network boundaries using custom IPv4 CIDR allocations.
- Gateway Selection: Discerning when to apply a bidirectional Internet Gateway versus a unidirectional NAT Gateway.
- Route Propagation: Mapping infrastructure components back to explicit route tables to direct internal VPC traffic flows.
- Host Access Control: Testing nested host configurations via SSH jumps and ICMP loop verification checks.
