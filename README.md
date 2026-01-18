# Enterprise Windows Domain Infrastructure Lab - Complete Implementation Guide  
## Project Overview  
This project demonstrates the complete implementation of an enterprise Windows domain infrastructure from scratch. It showcases Active Directory administration, network services configuration, and centralized management capabilities through hands-on implementation.  

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

## Phase 1: Infrastructure Setup  
### Step 1: Domain Controller Virtual Machine Creation  
The first step involves creating our primary domain controller which will serve as the foundation for all domain services.  
### VM Configuration:  
- Name: TechStart-DC01  
- OS: Windows Server 2022 Standard  
- RAM: 6GB minimum  
- Storage: 50GB  
- Network: Internal Network (VirtualBox)
### Step 2: Windows Server 2022 Installation  
Install Windows Server 2022 Standard with Desktop Experience to provide GUI management capabilities essential for administration and demonstration purposes.  
### Critical Installation Choice:  
- Select "Windows Server 2022 Standard Evaluation (Desktop Experience)"    <img width="619" height="92" alt="Screenshot 2026-01-16 151928" src="https://github.com/user-attachments/assets/8ccb3575-f87c-493a-8b38-b56d466147e1" />  
### Step 3: Initial Server Configuration  
After installation, Server Manager provides the central hub for all server role and feature management.  
<img width="1899" height="769" alt="Screenshot 2026-01-16 152132" src="https://github.com/user-attachments/assets/2c0dee53-3bed-49b2-a211-bd7a3580b587" />  
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
<img width="397" height="451" alt="Screenshot 2026-01-16 152324" src="https://github.com/user-attachments/assets/8ce005d5-b5bf-4e33-bc47-7cfd0a8c109c" />  

## Phase 2: Active Directory Domain Services  
### Step 5: Installing Active Directory Domain Services Role  
Active Directory Domain Services (AD DS) provides the foundation for domain authentication, authorization, and directory services.  
### Installation Process:  
- Server Manager → Add Roles and Features  
- Select "Active Directory Domain Services"  
- Include management tools when prompted  
- Complete installation  
<img width="781" height="547" alt="Screenshot 2026-01-16 152559" src="https://github.com/user-attachments/assets/f9582b84-5697-4144-9ea0-79446cc61acf" />  

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

### Why .local TLD (Top-Level Domain):   
While not internet-routable, .local provides clear separation between internal and external namespaces in lab environments.  

### Step 7: Post-Promotion Verification  
After restart, verify successful domain controller promotion and service functionality.  
<img width="748" height="484" alt="Screenshot 2026-01-16 154109" src="https://github.com/user-attachments/assets/eceea78f-4fc7-4ecf-9203-c47300c5a334" />  


### Verification Checklist:  
- Login shows TECHSTART\Administrator  
- DNS service running  
- Active Directory Domain Services started  
- Event logs show successful promotion

## Phase 3: Active Directory Organizational Structure  
### Step 8: Organizational Unit Design  
Proper OU design reflects business structure and enables efficient Group Policy application and administrative delegation.  
### OU Structure Design:  
<img width="749" height="517" alt="Screenshot 2026-01-16 161843" src="https://github.com/user-attachments/assets/c23e3bd5-ba34-4913-8d8f-5ba8c937fa2b" />  


### Design Principles:  
- Department-based separation enables targeted Group Policy application  
- Computer separation allows different policies for workstations vs servers  

### Step 9: User Account Creation  
<img width="710" height="526" alt="Screenshot 2026-01-16 162757" src="https://github.com/user-attachments/assets/93c503e3-c8f0-4e24-bd4b-cd42fa54326c" />  


### Step 10: Security Group Creation  
Security groups enable efficient permission management and follow the principle of least privilege.  
### Security Groups:  

| **Group Name**         | **Type**    | **Scope** | **Purpose**                       |
|------------------------|-------------|------------|-----------------------------------|
| Management_Users       | Security    | Global     | Management department access       |
| Sales_Users            | Security    | Global     | Sales department resources         |
| Technical_Users        | Security    | Global     | Technical team resources           |
| Admin_Users            | Security    | Global     | Administrative staff access        |
| File_Server_Access     | Security    | Global     | Basic file server access           |
| Printer_Users          | Security    | Global     | Network printer access             |

<img width="695" height="523" alt="Screenshot 2026-01-16 161857" src="https://github.com/user-attachments/assets/68673913-dd0a-490b-a06c-13032edde5e2" />  

  

### Group Strategy:   
Department-specific groups enable granular access control while File_Server_Access provides baseline connectivity.  

### Step 11: Group Membership Assignment  
Proper group membership assignment implements role-based access control (RBAC) principles.  

### Membership Strategy:  
- Each user belongs to their department group  
- All users added to File_Server_Access for baseline connectivity  
- Future expansion allows for cross-departmental project groups

## Phase 4: Network Services Configuration  
### Step 12: DHCP Server Installation  
DHCP provides automated IP address management, reducing administrative overhead and ensuring consistent network configuration.  

<img width="775" height="550" alt="Screenshot 2026-01-16 162855" src="https://github.com/user-attachments/assets/4f29bbf4-4763-4248-bb7a-ad542670af40" />  

### DHCP Benefits:  
- Automated IP assignment reduces configuration errors  
- Centralized management of network settings  
- Automatic DNS and gateway distribution  
- IP address conflict prevention

### Step 13: DHCP Scope Configuration  
DHCP scopes define IP address ranges and associated network configuration for client systems.  
- Scope Name: TechStart Network  
- IP Range: 192.168.10.100 - 192.168.10.200  
- Subnet Mask: 255.255.255.0  
- Default Gateway: 192.168.10.1  
- DNS Servers: 192.168.10.10  
- Lease Duration: 8 days (default)
<h2>  
  
<div>
<img width="509" height="424" alt="Screenshot 2026-01-16 163723" src="https://github.com/user-attachments/assets/2d6d32b7-3d09-4cb8-81d0-ee5cc4cc7abe" />  
</div>
<div>
<img width="514" height="414" alt="Screenshot 2026-01-16 163749" src="https://github.com/user-attachments/assets/020a9c0e-cfef-4911-8a2f-61ce8fca9065" />  
</div>
<div>
<img width="511" height="414" alt="Screenshot 2026-01-16 163845" src="https://github.com/user-attachments/assets/aedc24e4-dad3-44f1-8084-fa6f7f09e46e" />  
</div>
<div>
<img width="513" height="421" alt="Screenshot 2026-01-16 163924" src="https://github.com/user-attachments/assets/a5c7a9aa-28d7-4e7e-9304-4d1c3006580b" />  
</div>
<div>
<img width="514" height="417" alt="Screenshot 2026-01-16 163901" src="https://github.com/user-attachments/assets/eca175d4-75d2-4786-a763-3e3bf9f9586a" />  
</div>

### Scope Design Rationale:  
- 100-200 range reserves 1-99 for static assignments (servers, printers, network equipment)  
- 8-day lease balances IP conservation with renewal overhead
### Step 14: DHCP Server Authorization  
DHCP servers must be authorized in Active Directory to prevent rogue DHCP services from disrupting network operations.  
### Authorization Process:  
- Server Manager notification → Complete DHCP Configuration  
- Use TECHSTART\Administrator credentials  
- Verify green ticks in DHCP console
<img width="293" height="262" alt="Screenshot 2026-01-16 171800" src="https://github.com/user-attachments/assets/48463f91-6764-4043-a510-92982f117b5c" />  

### Security Importance:   
Authorization prevents unauthorized DHCP servers from assigning incorrect network configurations.  

## Phase 5: Client Integration  

### Step 15: Windows 11 Client Deployment  
Client workstations represent end-user systems that will authenticate against the domain and utilize centralized services.  
### Client VM Configuration:  
- Names: TechStart-01, TechStart-02  
- OS: Windows 11 Pro (required for domain join)  
- RAM: 6GB each  
- Network: NAT during installation (internet required), Switch to Internal Network after Windows 11 setup completes
### Step 16: Domain Join Process  
Joining clients to the domain establishes trust relationships and enables centralized authentication and management.  
### Domain Join Steps:  
- System Properties → Change Settings  
- Domain: techstart.local  
- Credentials: TECHSTART\Administrator  
- Restart required
<img width="727" height="383" alt="Screenshot 2026-01-16 175710" src="https://github.com/user-attachments/assets/602b0480-4263-4a73-85fd-dba175a322e1" />  

### Post-Join Verification:  
- Login screen shows "Other user" option  
- Computer appears in AD Users and Computers  
- Event logs confirm successful domain join  
<img width="1015" height="764" alt="Screenshot 2026-01-16 180111" src="https://github.com/user-attachments/assets/166813b3-e6de-4bc4-a017-643b99b81f43" />  

### Step 17: DHCP Lease Verification  
Confirming DHCP functionality  
<img width="693" height="323" alt="Screenshot 2026-01-16 180450" src="https://github.com/user-attachments/assets/4d17fbbd-e75b-4c67-90f6-07d25f208e56" />  

### Verification Points:  
- Clients receive IPs in 192.168.10.100-200 range  
- DNS server automatically configured as 192.168.10.10  
- Lease information shows client hostnames

## Phase 6: File Services Implementation  

### Step 18: File Share Structure Creation  
Departmental file shares provide centralized storage with appropriate security boundaries reflecting business organizational structure.  
### Share Structure:  
<img width="421" height="267" alt="Screenshot 2026-01-16 181225" src="https://github.com/user-attachments/assets/0077f22d-25b6-4d49-866f-6c93fb75748a" />  

### Design Principles:  
- Department separation maintains data confidentiality  
- Public share enables company-wide collaboration  
- Centralized location simplifies backup and management

### Step 19: Share-Level Permissions  
Share permissions control network access to folders, providing the first layer of access control.  
### Share Permission Strategy:  
- Remove "Everyone" group (security best practice)  
- File_Server_Access group: Full Control (refined by NTFS)  
- Domain Admins: Full Control (administrative access)
<img width="353" height="441" alt="Screenshot 2026-01-16 182005" src="https://github.com/user-attachments/assets/6e146c9c-722e-408c-8cac-6401d7d7ef6c" />  

- Hidden share ($): Reduces casual browsing, improves security
<img width="350" height="359" alt="Screenshot 2026-01-16 182012" src="https://github.com/user-attachments/assets/728d95e9-a748-4cb4-979a-1b225f942955" />  

### Step 20: NTFS Permission Configuration  
NTFS permissions provide granular access control at the file system level, implementing the principle of least privilege.  
### Permission Strategy per Department Folder:  
- Inheritance disabled for explicit control
<img width="762" height="521" alt="Screenshot 2026-01-17 164203" src="https://github.com/user-attachments/assets/d71eedc4-37aa-425f-8c44-b192888a419e" />  

- Department groups: Full Control to respective folders only
<img width="763" height="513" alt="Screenshot 2026-01-17 164416" src="https://github.com/user-attachments/assets/021708d9-65ab-4c12-b2af-ce64c5212ef2" />  

- Domain Admins: Full Control (administrative override)  
- SYSTEM: Full Control (required for proper function)

### Security Benefit:  
Users can only access their department's data, preventing accidental or intentional cross-department access.  

### Step 21: File Share Access Testing  
Validating access controls ensures security policies function as designed and users have appropriate access to resources.  
<img width="487" height="245" alt="Screenshot 2026-01-17 175909" src="https://github.com/user-attachments/assets/1f129417-64b8-4b7f-8db9-e15ee23ef001" />  

<img width="514" height="187" alt="Screenshot 2026-01-17 180023" src="https://github.com/user-attachments/assets/e76d8356-c9cd-44e2-a623-4c1b5090bc27" />  


## Phase 7: Group Policy Management  

### Step 22: Group Policy Object Creation  
Group Policy provides centralized configuration management for domain-joined computers and users, reducing administrative overhead and ensuring consistent configurations.  
<img width="748" height="360" alt="Screenshot 2026-01-17 180409" src="https://github.com/user-attachments/assets/881fdf39-8589-4072-a8c2-1404ad0719df" />  
GPO Naming Convention: "TechStart Workstation Policy" clearly identifies scope and purpose for future administrators.  

### Step 23: Policy Configuration  
Implementing common business policies  
### Policies Configured:  
| **Policy**              | **Location**             | **Setting**               | **Business Justification**                                |
|--------------------------|--------------------------|----------------------------|------------------------------------------------------------|
| Desktop Wallpaper        | User Configuration       | Company branding           | Professional appearance, brand consistency                 |                         |
| Password Length          | Computer Configuration   | 12 characters minimum       | Security compliance requirement                            |

<img width="756" height="457" alt="Screenshot 2026-01-17 181039" src="https://github.com/user-attachments/assets/95a899f7-ce2d-4fc3-828e-e3a7b344c0e6" />  
### Wallpaper Name: C:\Windows\Web\Wallpaper\Windows\img0.jpg  
<img width="968" height="500" alt="Screenshot 2026-01-17 181609" src="https://github.com/user-attachments/assets/09362c8d-66cc-4fa3-b353-e3387887e2ba" />  
 

### Step 24: GPO Linking and Application  
Linking GPOs to domain containers applies policies to all contained objects  
<img width="752" height="515" alt="Screenshot 2026-01-17 182030" src="https://github.com/user-attachments/assets/78fca136-2f0f-4c24-b33f-b8c0d8200974" />  

### Linking Strategy:  
- Domain-level linking applies to all domain computers  
- Future flexibility allows OU-specific policies as organization grows  
- Immediate application via gpupdate /force command
<img width="582" height="267" alt="Screenshot 2026-01-17 182304" src="https://github.com/user-attachments/assets/a8e744be-e610-48c0-acec-4a9a663a811e" />  

### Step 25: Policy Verification  
### Verification Results:  
- Desktop wallpaper changed to corporate standard  
<img width="1018" height="761" alt="Screenshot 2026-01-17 182603" src="https://github.com/user-attachments/assets/3ea48d32-5924-49d0-a49a-fb1e073c6aa2" />  

- Minimum password lenght set to 12 characters  

## Phase 8: Print Services  
### Step 26: Print Server Configuration  
Network printing capabilities provide centralized print management and enable resource sharing across the organization.  
<img width="779" height="557" alt="Screenshot 2026-01-17 183125" src="https://github.com/user-attachments/assets/63c201f6-dafe-48ff-b16a-0c9e43d2e0b0" />  


### Print Server Benefits:  
- Centralized management of printer drivers and settings  
- User access control through security groups  
- Print job monitoring and troubleshooting capabilities
<img width="767" height="520" alt="Screenshot 2026-01-18 160026" src="https://github.com/user-attachments/assets/a512dbdc-a0e5-4543-856c-e8f8b1d5c47f" />  

- Cost control through usage tracking

### Step 27: Network Printer Deployment  
Deployment Method: UNC path (\\TechStart-DC01\TechStart-Printer-01) provides direct network printer access.  
<img width="560" height="339" alt="Screenshot 2026-01-18 160602" src="https://github.com/user-attachments/assets/81b46d1b-3522-4f5d-a03c-0a29c7d69c6a" />  

## Phase 9: Final Integration Testing  
### Step 28: Multi-User Domain Authentication  
### Test Scenarios:  
- Management user login with appropriate file access  
- Sales user login with department-specific resources  
- Technical user login with proper group memberships  
- Cross-department access restrictions functioning

### Step 29: Network Services Integration  
Final verification confirms all services work together cohesively in an integrated enterprise environment.  
### Integration Verification:  
- DNS Resolution: Clients resolve domain names correctly  
- DHCP Assignment: Automatic IP configuration functioning  
- File Services: Department shares accessible with proper security  
- Print Services: Network printing operational  
- Group Policy: Centralized management policies applied  
- Active Directory: User authentication and authorization working

## Project Outcomes  
### Technical Skills
- Enterprise Infrastructure Design - Creating scalable, secure domain architecture  
- Active Directory Administration - User/group management, OU design, delegation  
- Network Services Management - DNS, DHCP configuration and troubleshooting  
- File Services Security - Share and NTFS permissions, access control principles  
- Group Policy Administration - Centralized management, policy design and application  
- Print Services Management - Network printing, centralized administration  
- Integration Testing - End-to-end validation, troubleshooting methodology  
- Documentation - Technical writing, process documentation

### Business Value
- Centralized Authentication - Single sign-on capability for all users  
- Automated Network Configuration - Reduced manual IP management overhead  
- Departmental Data Security - Information access controls matching business structure  
- Standardized Desktop Environment - Consistent user experience and security policies  
- Shared Resource Management - Efficient printer and file sharing  
- Scalable Architecture - Foundation supports organizational growth  

### Real-World Applications  
This infrastructure directly applies to:  
- Small to Medium Business environments (25-500 users)  
- Branch Office implementations  
- Departmental IT Services within larger organizations  
- Managed Service Provider client environments  
- Educational Institution computer labs and administration

### Troubleshooting Guide  

### Network Connectivity Problems
Problem: Clients cannot reach domain controller  
Solution: Verify all VMs use same Internal Network setting  

### DHCP Authorization Failures
Problem: "Cannot contact Active Directory" errors  
Cause: Local user accounts lack domain privileges  
Solution: Use TECHSTART\Administrator credentials for authorization  

### Group Policy Not Applying  
Problem: Desktop policies not visible on clients  
Solution: Run gpupdate /force and verify GPO linking  
Check: Event Viewer → Windows Logs → System for policy errors  

### File Share Access Issues  
Symptom: Users cannot access department folders  
Solution: Verify both share and NTFS permissions are correctly configured  
Debug: Use net use command to test UNC path connectivity  

## Lessons Learned  
- Planning Phase Critical - Proper network design prevents connectivity issues
- Security Layering - Multiple permission levels provide defense in depth  
- Testing Methodology - Systematic verification catches configuration errors early  
- Documentation Value - Clear processes enable consistent reproduction

### Professional Development  
- Problem Solving Skills - Network troubleshooting requires systematic approach  
- Attention to Detail - Small configuration errors have large impacts  
- Process Documentation - Clear procedures essential for team environments  
- Security Mindset - Default configurations often insufficient for business use

### Best Practices Reinforced  
- Principle of Least Privilege - Grant minimum necessary access rights  
- Change Management - Document all configuration modifications  
- Backup Strategy - Regular system state backups prevent data loss  
- Monitoring Importance - Proactive monitoring prevents service disruptions

## Conclusion  
This Windows domain lab shows practical implementation of enterprise Active Directory infrastructure. The project showcases essential IT administration skills including domain services, network configuration, security implementation, and centralized management capabilities.  

The hands-on experience gained through building this environment directly translates to real-world IT support, system administration, and infrastructure management roles. The systematic approach to documentation, troubleshooting, and testing reflects professional IT practices valued by employers.  

Key Takeaway: Modern business environments require robust, secure, and manageable IT infrastructure. This lab provides the foundation for understanding and implementing these critical enterprise services.  

