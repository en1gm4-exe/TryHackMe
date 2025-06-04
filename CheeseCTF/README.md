# CHEESE CTF:  Summary & Recommendations


# Walkthrough: 
[https://github.com/en1gm4-exe/CheeseCTF-TryHackMe/Walkthrough.md](https://github.com/en1gm4-exe/TryHackMe/blob/main/CheeseCTF/WriteUp.md)




## 🚩 Attack Flow Summary
1. **Initial Access**: SQLi bypass on login page → Gained web app access  
   `comte' AND 1=1 -- -`  
2. **LFI Exploitation**:  
   ```bash
   ?file=php://filter/convert.base64-encode/resource=secret.php
3. **RCE Achieved**: PHP filter chain → Reverse shell
4. **Persistence**: SSH key injection to .ssh/authorized_keys
5. **Privilege Escalation**: Abused writable systemd timer → Root access
   

## 🛡️ Remediation Checklist

# In PHP configuration (php.ini):
+ disable_functions = "system,exec,passthru,shell_exec"
+ allow_url_include = Off
+ open_basedir = /var/www/html:/tmp

# For SQLi:
- $query = "SELECT * FROM users WHERE user='$_POST[user]'";
+ $stmt = $pdo->prepare("SELECT * FROM users WHERE user=?");

# Fix SSH permissions:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Audit SUID binaries:
find / -perm -4000 -type f -exec ls -la {} \; 2>/dev/null

# Secure systemd:
sudo chmod 750 /etc/systemd/system/*
