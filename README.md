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

## Phase 2: Active Directory Domain Services  
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

## Phase 3: Active Directory Organizational Structure  
### Step 8: Organizational Unit Design  
Proper OU design reflects business structure and enables efficient Group Policy application and administrative delegation.  
### OU Structure Design:  
<img width="800" height="592" alt="Screenshot 2025-09-25 154437" src="https://github.com/user-attachments/assets/c6525aef-539e-43c2-847c-43652a04568e" />  

### Design Principles:  
- Department-based separation enables targeted Group Policy application  
- Computer separation allows different policies for workstations vs servers  

### Step 9: User Account Creation  
<img width="349" height="104" alt="Screenshot 2025-09-28 115552" src="https://github.com/user-attachments/assets/b4db8d08-8547-4ded-8033-2143337641d1" />  

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

<img width="798" height="593" alt="Screenshot 2025-09-25 160436" src="https://github.com/user-attachments/assets/75c9828a-1770-4b84-aca0-9442c5092dd1" />  

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

<img width="778" height="546" alt="Screenshot 2025-09-25 160613" src="https://github.com/user-attachments/assets/c9a3b35e-94d9-4bb4-94f1-49b5106835d2" />  

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
<img width="287" height="480" alt="Screenshot 2025-09-25 160804" src="https://github.com/user-attachments/assets/a51e05a4-76d7-4f09-94bf-a4f4b1bca9a6" />
</div>
<div>
<img width="509" height="414" alt="Screenshot 2025-09-25 160914" src="https://github.com/user-attachments/assets/9def34d4-ec39-45c2-b78e-9ccf80f0733f" />
</div>
<div>
<img width="508" height="410" alt="Screenshot 2025-09-25 161133" src="https://github.com/user-attachments/assets/8c137411-4a8f-4d80-8db3-f11be4289686" />
</div>
<div>
<img width="507" height="416" alt="Screenshot 2025-09-25 161331" src="https://github.com/user-attachments/assets/e5796b25-782c-4629-82e2-db9753373b8b" />
</div>
<div>
<img width="444" height="294" alt="Screenshot 2025-09-25 161617" src="https://github.com/user-attachments/assets/99892912-6d9e-4546-ba55-dff94fa4ad05" />
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
<img width="184" height="367" alt="Screenshot 2025-09-28 122137" src="https://github.com/user-attachments/assets/0eff7a75-d0a8-41a7-923d-aa48f77bfb96" />

### Security Importance:   
Authorization prevents unauthorized DHCP servers from assigning incorrect network configurations.  

## Phase 5: Client Integration  

### Step 15: Windows 11 Client Deployment  
Client workstations represent end-user systems that will authenticate against the domain and utilize centralized services.  
### Client VM Configuration:  
- Names: TechStart-Client01, TechStart-Client02  
- OS: Windows 11 Pro (required for domain join)  
- RAM: 4GB each  
- Network: NAT during installation (internet required), Switch to Internal Network after Windows 11 setup completes
### Step 16: Domain Join Process  
Joining clients to the domain establishes trust relationships and enables centralized authentication and management.  
### Domain Join Steps:  
- System Properties → Change Settings  
- Domain: techstart.local  
- Credentials: TECHSTART\Administrator  
- Restart required
<img width="317" height="382" alt="Screenshot 2025-09-26 113534" src="https://github.com/user-attachments/assets/25dc6c79-abe5-445a-b1a2-f692a1897a69" />

### Post-Join Verification:  
- Login screen shows "Other user" option  
- Computer appears in AD Users and Computers  
- Event logs confirm successful domain join  
<img width="598" height="426" alt="Screenshot 2025-09-28 124627" src="https://github.com/user-attachments/assets/1ce2ffa0-3f43-4e4e-a0b4-8ff451074da2" />

### Step 17: DHCP Lease Verification  
Confirming DHCP functionality  
<img width="952" height="376" alt="Screenshot 2025-09-27 112143" src="https://github.com/user-attachments/assets/f671c1ca-5c23-4cb2-81c2-96020bb97dc9" />  

### Verification Points:  
- Clients receive IPs in 192.168.10.100-200 range  
- DNS server automatically configured as 192.168.10.10  
- Lease information shows client hostnames

## Phase 6: File Services Implementation  

### Step 18: File Share Structure Creation  
Departmental file shares provide centralized storage with appropriate security boundaries reflecting business organizational structure.  
### Share Structure:  
<img width="779" height="251" alt="Screenshot 2025-09-27 114320" src="https://github.com/user-attachments/assets/e1d819a0-b2fb-4c18-91f3-f104e2d84e89" />  
<img width="780" height="234" alt="Screenshot 2025-09-27 114327" src="https://github.com/user-attachments/assets/4317a068-32db-4241-b55f-c7a202629d6e" />  

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
<img width="352" height="441" alt="Screenshot 2025-09-27 114246" src="https://github.com/user-attachments/assets/ef5a0bae-9d4c-4edf-aa2c-559b8ae3bdd8" />  

- Hidden share ($): Reduces casual browsing, improves security
<img width="345" height="438" alt="Screenshot 2025-09-27 114650" src="https://github.com/user-attachments/assets/5cb353f6-bdab-4fe0-bc00-45c329d1104a" />  

### Step 20: NTFS Permission Configuration  
NTFS permissions provide granular access control at the file system level, implementing the principle of least privilege.  
### Permission Strategy per Department Folder:  
- Inheritance disabled for explicit control
<img width="763" height="509" alt="Screenshot 2025-09-27 114929" src="https://github.com/user-attachments/assets/14a6e2ee-11dd-4b48-b8c4-655ef87daeb4" />  

- Department groups: Full Control to respective folders only
<img width="759" height="511" alt="Screenshot 2025-09-27 115208" src="https://github.com/user-attachments/assets/15f24896-19d6-4fc3-9ac5-d450403e8657" />  

- Domain Admins: Full Control (administrative override)  
- SYSTEM: Full Control (required for proper function)

### Security Benefit:  
Users can only access their department's data, preventing accidental or intentional cross-department access.  

### Step 21: File Share Access Testing  
Validating access controls ensures security policies function as designed and users have appropriate access to resources.  
<img width="515" height="207" alt="Screenshot 2025-09-27 115643" src="https://github.com/user-attachments/assets/e196b14d-8010-40ef-bdcf-d8b8919cdf2f" />  

## Phase 7: Group Policy Management  

### Step 22: Group Policy Object Creation  
Group Policy provides centralized configuration management for domain-joined computers and users, reducing administrative overhead and ensuring consistent configurations.  
<img width="304" height="428" alt="Screenshot 2025-09-27 115940" src="https://github.com/user-attachments/assets/8a9d2572-438e-4767-bf98-f0de87345835" />  
GPO Naming Convention: "TechStart Workstation Policy" clearly identifies scope and purpose for future administrators.  

### Step 23: Policy Configuration  
Implementing common business policies  
### Policies Configured:  
| **Policy**              | **Location**             | **Setting**               | **Business Justification**                                |
|--------------------------|--------------------------|----------------------------|------------------------------------------------------------|
| Desktop Wallpaper        | User Configuration       | Company branding           | Professional appearance, brand consistency                 |
| Control Panel Access     | User Configuration       | Prohibited                 | Prevents user system modifications                         |
| Password Length          | Computer Configuration   | 8 characters minimum       | Security compliance requirement                            |

<img width="955" height="433" alt="Screenshot 2025-09-27 120225" src="https://github.com/user-attachments/assets/d0f11430-faf3-4a28-b6fa-9e9a9fa68d85" />  
### Wallpaper Name: C:\Windows\Web\Wallpaper\Windows\img0.jpg  
<img width="675" height="627" alt="Screenshot 2025-09-27 120337" src="https://github.com/user-attachments/assets/aef1ac77-3aaf-411c-90a8-f53a2a84b397" />  
<img width="950" height="511" alt="Screenshot 2025-09-27 120414" src="https://github.com/user-attachments/assets/8de2465b-0495-4263-b132-7cce90c21b28" />  
<img width="657" height="669" alt="Screenshot 2025-09-27 120535" src="https://github.com/user-attachments/assets/7145be65-48c3-473e-a4c6-539b1f1f4f0a" />  
<img width="407" height="499" alt="Screenshot 2025-09-27 120551" src="https://github.com/user-attachments/assets/605139b1-0e57-484f-a148-e5a6ee65dc47" />  

### Step 24: GPO Linking and Application  
Linking GPOs to domain containers applies policies to all contained objects  
<img width="442" height="398" alt="Screenshot 2025-09-27 120652" src="https://github.com/user-attachments/assets/a5ef01a8-3d3a-471f-a4f4-cc851ade3ea4" />  

### Linking Strategy:  
- Domain-level linking applies to all domain computers  
- Future flexibility allows OU-specific policies as organization grows  
- Immediate application via gpupdate /force command
<img width="523" height="262" alt="Screenshot 2025-09-27 120757" src="https://github.com/user-attachments/assets/71fe8cf1-e280-454a-9c5d-ae9ad383ca14" />

### Step 25: Policy Verification  
### Verification Results:  
- Desktop wallpaper changed to corporate standard  
- Control Panel access blocked
<img width="550" height="129" alt="Screenshot 2025-09-27 120850" src="https://github.com/user-attachments/assets/0b21c3be-83cf-45e3-bd69-9441f5fb7171" />

- Minimum password lenght set to 8 characters  

## Phase 8: Print Services  
### Step 26: Print Server Configuration  
Network printing capabilities provide centralized print management and enable resource sharing across the organization.  
<img width="771" height="546" alt="Screenshot 2025-09-27 121145" src="https://github.com/user-attachments/assets/1eb5e4d8-71ac-4a4b-81df-4a08217a2cde" />  
<img width="570" height="431" alt="Screenshot 2025-09-27 121731" src="https://github.com/user-attachments/assets/bc435d6c-f36e-491e-b874-e06b2f567b00" />  


### Print Server Benefits:  
- Centralized management of printer drivers and settings  
- User access control through security groups  
- Print job monitoring and troubleshooting capabilities
<img width="719" height="267" alt="Screenshot 2025-09-27 122527" src="https://github.com/user-attachments/assets/849c1084-e680-4678-8968-d309bfa68760" />  

- Cost control through usage tracking

### Step 27: Network Printer Deployment  
Deployment Method: UNC path (\\TechStart-DC01\TechStart-Printer-01) provides direct network printer access.  
<img width="600" height="441" alt="Screenshot 2025-09-27 122244" src="https://github.com/user-attachments/assets/2a78caca-eda1-4148-ac9b-bef037d8349b" />  

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
### Technical Skills Demonstrated  
- Enterprise Infrastructure Design - Creating scalable, secure domain architecture  
- Active Directory Administration - User/group management, OU design, delegation  
- Network Services Management - DNS, DHCP configuration and troubleshooting  
- File Services Security - Share and NTFS permissions, access control principles  
- Group Policy Administration - Centralized management, policy design and application  
- Print Services Management - Network printing, centralized administration  
- Integration Testing - End-to-end validation, troubleshooting methodology  
- Documentation - Technical writing, process documentation

### Business Value Delivered  
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
Solution: Use TECHSTART\Administrator credentials for authorization  
Root Cause: Local user accounts lack domain privileges  

### Group Policy Not Applying  
Problem: Desktop policies not visible on clients  
Solution: Run gpupdate /force and verify GPO linking  
Check: Event Viewer → Windows Logs → System for policy errors  

### File Share Access Issues  
Symptom: Users cannot access department folders  
Solution: Verify both share and NTFS permissions are correctly configured  
Debug: Use net use command to test UNC path connectivity  

## Future Enhancements  
### Additional Services Integration  
- Certificate Services - PKI implementation for enhanced security  
- Windows Server Update Services - Centralized patch management  
- Remote Desktop Services - Centralized application delivery  
- Backup and Recovery - System State and file-level backup solutions

### Advanced Active Directory Features  
- Multiple Domain Controllers - High availability and load distribution  
- Site and Services - Multi-location replication management  
- Fine-Grained Password Policies - Role-based password requirements  
- Administrative Delegation - Distributed management responsibilities

### Monitoring and Management  
- System Center Operations Manager - Enterprise monitoring  
- PowerShell Desired State Configuration - Automated compliance  
- Event Log Monitoring - Proactive issue detection  
- Performance Monitoring - Capacity planning and optimization

## Lessons Learned  
- Planning Phase Critical - Proper network design prevents connectivity issues
- Security Layering - Multiple permission levels provide defense in depth  
- Testing Methodology - Systematic verification catches configuration errors early  
- Documentation Value - Clear processes enable consistent reproduction

### Professional Development  
- Problem-Solving Skills - Network troubleshooting requires systematic approach  
- Attention to Detail - Small configuration errors have large impacts  
- Process Documentation - Clear procedures essential for team environments  
- Security Mindset - Default configurations often insufficient for business use

### Best Practices Reinforced  
- Principle of Least Privilege - Grant minimum necessary access rights  
- Change Management - Document all configuration modifications  
- Backup Strategy - Regular system state backups prevent data loss  
- Monitoring Importance - Proactive monitoring prevents service disruptions

## Conclusion  
This comprehensive Windows domain lab demonstrates practical implementation of enterprise Active Directory infrastructure. The project showcases essential IT administration skills including domain services, network configuration, security implementation, and centralized management capabilities.  

The hands-on experience gained through building this environment directly translates to real-world IT support, system administration, and infrastructure management roles. The systematic approach to documentation, troubleshooting, and testing reflects professional IT practices valued by employers.  

Key Takeaway: Modern business environments require robust, secure, and manageable IT infrastructure. This lab provides the foundation for understanding and implementing these critical enterprise services.  

