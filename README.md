# Enterprise Windows Domain Infrastructure Lab - Complete Implementation Guide  
## Project Overview  
The project demonstrates the complete implementation of an enterprise Windows domain infrastructure from scratch. Built as a comprehensive learning resource. This lab showcases Active Directory administration, network services configuration, and centralized management capabilities.  

## Business Scenario  
### TechStart Consulting - A 25-employee IT services company requiring:  
- Centralized user authentication and management
- Department-based file sharing with security controls
- Automated IP address management
- Centralized desktop policy enforcement
- Network printer sharing
- Scalable infrastructure design

## Skills Demonstrated  
- Active Directory Domain Services - Domain controller deployment, OU design, user/group management
- DNS Services - Domain name resolution and forward/reverse lookup zones
- DHCP Services - Dynamic IP allocation with scope management and reservations
- File Services - Departmental shares with NTFS and share-level permissions
- Group Policy - Centralized desktop management and security policy enforcement
- Print Services - Network printer deployment and management
- Network Design - Enterprise network segmentation and service integration
- Troubleshooting - Network connectivity and service resolution

## Network Architecture  
Domain: techstart.local  
Network Segment: 192.168.10.0/24  
Domain Controller: 192.168.10.10  
Client Range: 192.168.10.100-200 (DHCP)  

## Prerequisites  
### Hardware Requirements  
- Host Machine: 16GB RAM minimum (8GB for basic functionality)
- Storage: 200GB free space
- Virtualization: VirtualBox, VMware Workstation, or Hyper-V
## Software Requirements  
- Windows Server 2022 ISO  
- Windows 11 ISO (x2 instances)  
- Virtualization platform of choice
## Knowledge Prerequisites  
- Basic Windows administration  
- Fundamental networking concepts (IP addressing, DNS)  
- Virtual machine management
## Implementation Guide  
### Phase 1: Infrastructure Setup  
### Step 1: Domain Controller Virtual Machine Creation  
The first step involves creating our primary domain controller which will serve as the foundation for all domain services.  
### VM Configuration:  
- Name: TechStart-DC01  
- OS: Windows Server 2022 Standard  
- RAM: 4GB minimum  
- Storage: 60GB  
- Network: Internal Network (VirtualBox)
### Step 2: Windows Server 2022 Installation  
Install Windows Server 2022 Standard with Desktop Experience to provide GUI management capabilities essential for administration and demonstration purposes.  
### Critical Installation Choice:  
- Select "Windows Server 2022 Standard Evaluation (Desktop Experience)"  <img width="602" height="27" alt="Screenshot 2025-09-27 125825" src="https://github.com/user-attachments/assets/89e1762e-e08f-49cb-9fc8-539a227b5568" />  
- NOT Server Core (command-line only)
### Step 3: Initial Server Configuration  
After installation, Server Manager provides the central hub for all server role and feature management.  
<img width="1915" height="949" alt="Screenshot 2025-09-25 145421" src="https://github.com/user-attachments/assets/7d831250-8f96-4945-9f98-7a15a1cba904" />  
### Key Configuration Steps:  
- Computer Name: Change from default to TechStart-DC01  
- Windows Updates: Configure automatic updates (enterprise consideration)  
### Step 4: Network Configuration  
Static IP configuration is essential for domain controllers to ensure consistent service availability.  
### Network Settings:  
- IP Address: 192.168.10.10  
- Subnet Mask: 255.255.255.0  
- Default Gateway: 192.168.10.1  
- Primary DNS: 192.168.10.10 (self-referencing)  
- Secondary DNS: 8.8.8.8 (external fallback)
<img width="390" height="445" alt="Screenshot 2025-09-25 150110" src="https://github.com/user-attachments/assets/3916ce4f-0838-4fe9-b792-77d27297ec8f" />

### Phase 2: Active Directory Domain Services  
### Step 5: Installing Active Directory Domain Services Role  
Active Directory Domain Services (AD DS) provides the foundation for domain authentication, authorization, and directory services.  
### Installation Process:  
- Server Manager → Add Roles and Features  
- Select "Active Directory Domain Services"  
- Include management tools when prompted  
- Complete installation (requires restart)  
<img width="778" height="552" alt="Screenshot 2025-09-25 150311" src="https://github.com/user-attachments/assets/c638514f-d14e-47e5-b122-28628ba5b48d" />

### What AD DS Provides:  
- Centralized authentication  
- User and computer account management  
- Group Policy infrastructure  
- DNS integration  
- Certificate Services foundation

### Step 6: Domain Controller Promotion  
Promoting the server to domain controller creates the forest root domain and initializes the Active Directory database.  
### Domain Configuration:  
- Deployment Type: Add a new forest  
- Root Domain Name: techstart.local  
- Forest/Domain Functional Level: Windows Server 2016 or higher  
- Domain Controller Options: DNS Server (checked)  
- DSRM Password: Strong password for Directory Services Restore Mode  
<img width="753" height="551" alt="Screenshot 2025-09-25 150529" src="https://github.com/user-attachments/assets/f5c5edec-f03d-49ca-9d64-031e8ecd0c71" />

### Why .local TLD (Top-Level Domain):   
While not internet-routable, .local provides clear separation between internal and external namespaces in lab environments.  

### Step 7: Post-Promotion Verification  
After restart, verify successful domain controller promotion and service functionality.  
<img width="747" height="541" alt="Screenshot 2025-09-25 154052" src="https://github.com/user-attachments/assets/a6690220-12e4-4881-9fcb-e99022e78285" />  

### Verification Checklist:  
- Login shows TECHSTART\Administrator  
- DNS service running  
- Active Directory Domain Services started  
- Event logs show successful promotion

### Phase 3: Active Directory Organizational Structure  
### Step 8: Organizational Unit Design  
Proper OU design reflects business structure and enables efficient Group Policy application and administrative delegation.  
### OU Structure Design:  

