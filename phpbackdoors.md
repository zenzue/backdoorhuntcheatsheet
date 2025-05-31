## PHP Backdoor Hunting Cheat Sheet (Linux)

### 1. Find Suspicious PHP Files

```bash
# Search for dangerous functions in PHP files
find /var/www/ -type f -name "*.php" -exec grep -iE "(eval|base64_decode|system|shell_exec|passthru|popen|proc_open)" {} \; -print

# Hidden PHP files
find /var/www/ -type f -name ".*.php"

# Recently modified PHP files
find /var/www/ -type f -name "*.php" -mtime -3
```

### 2. Detect Obfuscated or Encoded Payloads

```bash
# Look for base64 encoded payloads
grep -r --include="*.php" "base64_decode" /var/www/

# Look for nested functions used in obfuscation
grep -r --include="*.php" -E "(eval\(base64_decode|gzinflate|str_rot13|create_function|assert\()" /var/www/
```

### 3. Monitor for Web Shell Behavior

```bash
# Search for command execution strings
grep -r --include="*.php" -E "(cmd|shell|uname|id|whoami|wget|curl|python|perl|nc|netcat|tcp|udp)" /var/www/
```

### 4. Check for File Upload Misuse

```bash
# Identify world-writable directories
find /var/www/ -type d -perm -0002 -exec ls -ld {} \;

# Find recently uploaded or created PHP files
find /var/www/html/uploads/ -type f -iname "*.php" -ctime -1
```

### 5. Analyze Web Server Logs

```bash
# Apache log inspection for unusual patterns
grep -Ei "(cmd|shell|base64|eval|POST \/|\\x[a-f0-9]{2})" /var/log/apache2/access.log*

# Nginx suspicious access patterns
grep "\.php?" /var/log/nginx/access.log | grep -E "(=id|=whoami|=uname|=ls)"
```

### 6. Identify Rogue or Misplaced PHP Files

```bash
# PHP files outside web roots
find /etc /usr /home -type f -name "*.php"

# PHP inside image or document files
find /var/www/ -type f \( -iname "*.jpg" -o -iname "*.png" -o -iname "*.txt" \) -exec grep -l "<?php" {} \;
```

### 7. Scan With Security Tools

```bash
# ClamAV scan with PHP rules
clamscan -r /var/www/

# Linux Malware Detect (LMD)
maldet -a /var/www/
```

---

### Additional Recommendations

* Harden `php.ini`:

  ```ini
  disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,eval,assert
  ```
* Monitor file integrity with tools like AIDE or Tripwire
* Sanitize and restrict upload directories using MIME and extension checks
