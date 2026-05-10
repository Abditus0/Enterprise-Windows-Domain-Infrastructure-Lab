# Enterprise Windows Domain Infrastructure Lab

A full Windows Server domain built for a consulting company. Active Directory runs the show. I designed the OU structure around real departments, created security groups for each one, and assigned users so permissions follow the role, not the person. Sales people see Sales files. Management sees Management files. Nobody sees what they shouldn't.

DHCP hands out IPs in a controlled range so static assignments stay clean. DNS resolves the internal domain. File shares are locked down with both share-level and NTFS permissions, so even if someone bypasses one layer, the other still holds. Print services are centralized so adding a printer means one configuration in one place. Group Policy ties it all together. Password length, desktop standards, security settings, all pushed from the domain controller to every machine in the domain. Two Windows 11 clients are joined to the domain to prove the whole thing works end to end.

## The setup

| Role | Hostname | OS | IP |
|------|----------|----|-----|
| Domain Controller | TechStart-DC01 | Windows Server 2022 | 192.168.10.10 (static) |
| Client 1 | TechStart-01 | Windows 11 Pro | DHCP (192.168.10.100-200) |
| Client 2 | TechStart-02 | Windows 11 Pro | DHCP (192.168.10.100-200) |

Domain: `techstart.local`
Network: `192.168.10.0/24`
DNS: `192.168.10.10` (the DC)

The DC is on a static IP because a domain controller that gets a different IP after every reboot is going to ruin your day. Clients pull DHCP from the same DC. The 1-99 range is reserved for static assignments (servers, printers, network gear), 100-200 is the DHCP scope, and 201-254 is held in reserve.

<img width="397" height="451" alt="DC static IP configuration" src="https://github.com/user-attachments/assets/8ce005d5-b5bf-4e33-bc47-7cfd0a8c109c" />

## Active Directory

After installing the AD DS role and promoting the server, I had a working forest root domain. First job was building the OU structure to match the company's departments instead of dumping everything in the default Users container.

<img width="749" height="517" alt="OU structure in Active Directory" src="https://github.com/user-attachments/assets/c23e3bd5-ba34-4913-8d8f-5ba8c937fa2b" />

Department-based OUs because Group Policy gets applied at the OU level. If you want one wallpaper for Sales and a different one for Management, you need them in different OUs. Same logic applies to password policy, software deployment, login scripts, all of it. Flat structure means no flexibility.

Computer OUs are separate from user OUs because workstations and servers usually need different policies. A workstation should auto-lock after 5 minutes. A server should not.

### Users and groups

Created user accounts for each department and dropped them into the right OUs.

<img width="710" height="526" alt="Domain user accounts" src="https://github.com/user-attachments/assets/93c503e3-c8f0-4e24-bd4b-cd42fa54326c" />

Then security groups, one per department plus a baseline group for everyone:

| Group | Purpose |
|-------|---------|
| Management_Users | Management department access |
| Sales_Users | Sales department resources |
| Technical_Users | Technical team resources |
| Admin_Users | Administrative staff access |
| File_Server_Access | Baseline file server connectivity |
| Printer_Users | Network printer access |

<img width="695" height="523" alt="Security groups" src="https://github.com/user-attachments/assets/68673913-dd0a-490b-a06c-13032edde5e2" />

Permissions get assigned to groups, never directly to users. When someone moves from Sales to Management, you change their group membership and that's it. No hunting through every share and folder to update permissions one by one. Everyone is also in `File_Server_Access` so they can reach the file server in the first place. Department groups control what they can see once they get there.

## DHCP

DHCP role installed on the DC. Created a scope for the 192.168.10.100-200 range with the gateway, subnet, and DNS server pushed out automatically.

<img width="513" height="421" alt="DHCP scope configuration" src="https://github.com/user-attachments/assets/a5c7a9aa-28d7-4e7e-9304-4d1c3006580b" />

Then authorized the DHCP server in Active Directory. This is a quiet but important step. Authorization is what tells AD that this DHCP server is allowed to hand out leases on the domain. Without it, Windows clients running on the domain will straight up ignore the DHCP server. It's also what stops a rogue DHCP server (someone plugging in a home router, for example) from handing out bad IPs to your domain machines.

<img width="293" height="262" alt="DHCP server authorized" src="https://github.com/user-attachments/assets/48463f91-6764-4043-a510-92982f117b5c" />

8-day lease duration, which is the default and fine for an office environment. Short enough that abandoned leases get cleaned up regularly, long enough that machines don't renew constantly.

## Joining clients to the domain

Two Windows 11 Pro VMs (Home doesn't support domain join, has to be Pro). Set them on NAT during install so Windows could pull updates, then switched them to the internal network and joined them to `techstart.local`.

<img width="727" height="383" alt="Domain join screen" src="https://github.com/user-attachments/assets/602b0480-4263-4a73-85fd-dba175a322e1" />

After the join and reboot, the login screen shows the domain and any domain user can sign in.

<img width="1015" height="764" alt="Domain login from Windows 11 client" src="https://github.com/user-attachments/assets/166813b3-e6de-4bc4-a017-643b99b81f43" />

Verified that DHCP was assigning IPs from the right range and that DNS was set to the DC automatically.

<img width="693" height="323" alt="DHCP lease verification" src="https://github.com/user-attachments/assets/4d17fbbd-e75b-4c67-90f6-07d25f208e56" />

## File shares

Department folders sitting on the DC, each one with its own share and its own permission set.

<img width="421" height="267" alt="File share structure" src="https://github.com/user-attachments/assets/0077f22d-25b6-4d49-866f-6c93fb75748a" />

The hidden share trick (the `$` at the end) doesn't make a share more secure on its own, but it does keep it out of network browse lists. Casual users browsing `\\TechStart-DC01` in Explorer won't see it. Anyone who knows the path can still hit it directly. It's not a security control, it's a tidiness one.

### Two layers of permissions

This is the part most people get wrong. Windows has both share permissions and NTFS permissions, and they both apply. The effective permission is the more restrictive of the two. Skip one and you've left a gap.

**Share permissions** control access over the network. I removed `Everyone` (default and bad) and gave `File_Server_Access` Full Control at the share level.

<img width="353" height="441" alt="Share-level permissions" src="https://github.com/user-attachments/assets/6e146c9c-722e-408c-8cac-6401d7d7ef6c" />

**NTFS permissions** control access at the file system level. This is where the actual department-by-department lockdown happens. Inheritance gets disabled on each department folder so the permissions don't leak down from the parent.

<img width="762" height="521" alt="NTFS inheritance disabled" src="https://github.com/user-attachments/assets/d71eedc4-37aa-425f-8c44-b192888a419e" />

Then the department group gets Full Control to its own folder, Domain Admins keep Full Control as a backstop, and SYSTEM keeps Full Control because Windows needs it for the file system to work properly.

<img width="763" height="513" alt="NTFS department group permissions" src="https://github.com/user-attachments/assets/021708d9-65ab-4c12-b2af-ce64c5212ef2" />

Result: a Sales user can open the Sales folder but gets denied on the Management folder. Both layers have to grant access, so even if I made a mistake on one of them, the other catches it.

<img width="487" height="245" alt="Access granted to own department" src="https://github.com/user-attachments/assets/1f129417-64b8-4b7f-8db9-e15ee23ef001" />  

Access denied.  
<img width="514" height="187" alt="Access denied to other department" src="https://github.com/user-attachments/assets/e76d8356-c9cd-44e2-a623-4c1b5090bc27" />  

## Group Policy

A single GPO called `TechStart Workstation Policy` linked at the domain level so it applies to every domain machine.

<img width="748" height="360" alt="GPO created in Group Policy Management" src="https://github.com/user-attachments/assets/881fdf39-8589-4072-a8c2-1404ad0719df" />

Two policies inside it for now:

- **Desktop wallpaper** set to a corporate image at `C:\Windows\Web\Wallpaper\Windows\img0.jpg`. Visual proof the policy is being applied, plus it stops users from changing it to whatever they want.
- **Minimum password length** set to 12 characters. Bumped from the default 7. Anything under 12 in 2026 is a joke.

<img width="756" height="457" alt="Password policy in GPO editor" src="https://github.com/user-attachments/assets/95a899f7-ce2d-4fc3-828e-e3a7b344c0e6" />

GPO linked to the domain root. `gpupdate /force` on the clients pulls the policy down without waiting for the 90-minute background refresh.

<img width="752" height="515" alt="GPO linked at domain level" src="https://github.com/user-attachments/assets/78fca136-2f0f-4c24-b33f-b8c0d8200974" />

Wallpaper changed on the next login.

<img width="1018" height="761" alt="Wallpaper applied via GPO" src="https://github.com/user-attachments/assets/3ea48d32-5924-49d0-a49a-fb1e073c6aa2" />

The whole point of GPO is that I configured this once on the DC and every domain-joined machine gets it. Add a 50th computer next year and it gets the same policy without anyone touching it.

## Print services

Print server role added so printer drivers and queues live on the DC instead of being installed on every workstation.

<img width="767" height="520" alt="Print Management console" src="https://github.com/user-attachments/assets/a512dbdc-a0e5-4543-856c-e8f8b1d5c47f" />

Clients connect to a printer with a UNC path (`\\TechStart-DC01\TechStart-Printer-01`). Drivers download from the server on first connect, so end users never see a driver install dialog.

<img width="560" height="339" alt="Network printer connection" src="https://github.com/user-attachments/assets/81b46d1b-3522-4f5d-a03c-0a29c7d69c6a" />

Adding a new printer is one job: install it on the DC, share it, point users at the path. Compare that to walking around to every workstation with a driver disk.

## Tech stack

- Windows Server 2022 Standard (Desktop Experience) for the DC
- Windows 11 Pro for clients (Home doesn't support domain join)
- Active Directory Domain Services
- DNS, DHCP, and Print Services roles on the DC
- Group Policy Management for centralized config
- VirtualBox for the lab, all VMs on an internal network

## Problems I had to solve

**Clients couldn't reach the DC at first.** Spent longer than I should have on this. Both VMs were on what I thought was the same network but VirtualBox had them on different internal network names. They were in completely separate broadcast domains. Renamed both adapters to the same internal network and they could see each other right away. If your client can't ping the DC, check VirtualBox network settings before anything else.

**DHCP wouldn't authorize.** Got "Cannot contact Active Directory" the first time I tried to authorize the DHCP server. I was logged in as a local user with admin rights, but DHCP authorization needs domain credentials. Switched to `TECHSTART\Administrator` and it went through in seconds.

**GPO wasn't applying.** Wallpaper wouldn't change on the client even after the policy was linked. Ran `gpupdate /force`, still nothing. Turned out I had configured the policy under User Configuration but the GPO wasn't linked to an OU containing the user, only the computer. Moved the link to the domain root so it covers both, problem solved. The trap is that User policies need to apply where the user lives, not where the computer lives.

**Forgot to disable NTFS inheritance.** First time around I set department permissions on the folders but inheritance was still on, so the parent folder's permissions were leaking down. Sales users could see Management files because of an inherited permission three levels up. Disabled inheritance on each department folder, converted the inherited permissions to explicit ones, then removed the ones that shouldn't be there. Now the only permissions on a department folder are the ones I put there on purpose.

## What I learned

The thing that really hit home is that AD permissions live in two places at once. Share permissions and NTFS permissions both apply, and the more restrictive one wins. Skip one and you've left a hole. The right move is to lock down both layers so a mistake on one is caught by the other.

OU design matters more than it looks. Build it right at the start and Group Policy is easy. Build it flat or wrong and every policy you add later becomes a workaround. Departments first, computer/user separation second, that's the layout that holds up.

Other things that came out of this:

- Always set the DC's DNS to itself (127.0.0.1 or the DC's own static IP). Pointing it at an external DNS first will break domain resolution in subtle ways.
- Domain join requires Windows Pro or Enterprise. Home edition will not work, no matter what registry hacks you find online.
- `gpupdate /force` is your friend during testing. Group Policy refreshes on its own every 90 minutes and you don't want to wait 90 minutes to see if your change worked.
- Default password policy on a Windows domain is weak. Always change it.

## Why I built it

I wanted to know what running an actual Windows domain involves end to end, not just the parts that come up in an interview. Setting up the DC, designing the OUs, getting users into groups, locking down permissions on two layers, pushing policy out from one place, joining real clients and watching it all click together.

Now I've done the whole loop. If a small office handed me their Windows environment tomorrow and asked me to clean it up or rebuild it, I'd know exactly where everything lives and how the pieces fit.
