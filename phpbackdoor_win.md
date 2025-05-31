## PHP Backdoor Hunting Cheat Sheet (Windows PowerShell)

### 1. Find Suspicious PHP Files

```powershell
# Search for PHP files with dangerous functions
Get-ChildItem -Recurse -Filter *.php -Path "C:\xampp\htdocs" | ForEach-Object {
    $content = Get-Content $_.FullName -Raw
    if ($content -match "eval|base64_decode|system|shell_exec|passthru|popen|proc_open") {
        Write-Output $_.FullName
    }
}

# List hidden PHP files
Get-ChildItem -Recurse -Filter ".*.php" -Path "C:\xampp\htdocs" -Force
```

### 2. Detect Obfuscated or Encoded Payloads

```powershell
# Search for base64_decode
Select-String -Path "C:\xampp\htdocs\*.php" -Pattern "base64_decode" -Recurse

# Detect multiple obfuscation patterns
Select-String -Path "C:\xampp\htdocs\*.php" -Pattern "eval\(base64_decode|gzinflate|str_rot13|create_function|assert\(" -Recurse
```

### 3. Check for Web Shell Indicators

```powershell
# Scan for command execution strings
Select-String -Path "C:\xampp\htdocs\*.php" -Pattern "cmd|shell|uname|id|whoami|wget|curl|python|perl|nc|netcat|tcp|udp" -Recurse
```

### 4. Check File Creation and Modification

```powershell
# Recently modified PHP files (last 3 days)
Get-ChildItem -Recurse -Filter *.php -Path "C:\xampp\htdocs" | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-3) }

# Recently created files
Get-ChildItem -Recurse -Filter *.php -Path "C:\xampp\htdocs" | Where-Object { $_.CreationTime -gt (Get-Date).AddDays(-3) }
```

### 5. Analyze Web Server Logs (Apache Example)

```powershell
# Search access logs for suspicious parameters
Select-String -Path "C:\xampp\apache\logs\access.log" -Pattern "cmd|shell|base64|eval|POST /|\x" 
```

### 6. Look for PHP in Non-standard Locations

```powershell
# PHP files outside web root
Get-ChildItem -Recurse -Include *.php -Path "C:\Users", "C:\Program Files", "C:\Windows" -ErrorAction SilentlyContinue

# Check if image or text files contain PHP
Get-ChildItem -Recurse -Include *.jpg, *.png, *.txt -Path "C:\xampp\htdocs" | ForEach-Object {
    if (Select-String -Path $_.FullName -Pattern "<\?php" -Quiet) {
        Write-Output "PHP code inside file: $($_.FullName)"
    }
}
```

### 7. Use Windows Security Tools

* Enable **Windows Defender** or third-party AV with web shell signatures.
* Use **Sysmon** for monitoring file creation and PowerShell execution.

---

### Additional Recommendations

* Set `disable_functions` in `php.ini`:

  ```ini
  disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,eval,assert
  ```
* Use access control to prevent PHP uploads to executable directories.
* Monitor suspicious PowerShell or PHP execution with Windows Event Logs.
