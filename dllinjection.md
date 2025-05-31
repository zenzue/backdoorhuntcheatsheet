## DLL Injection Detection Cheatsheet

### Windows (PowerShell)

#### 1. List Suspicious DLLs Loaded by Processes

```powershell
Get-Process | ForEach-Object {
    try {
        $_.Modules | Select-Object @{Name="ProcessName";Expression={$_.ModuleName}}, FileName
    } catch {}
} | Where-Object { $_.FileName -like "*Temp*" -or $_.FileName -like "*AppData*" }
```

#### 2. Find Non-Microsoft DLLs

```powershell
Get-Process | ForEach-Object {
    try {
        $_.Modules | Where-Object { $_.FileName -and $_.FileName -notlike "C:\Windows\*" } |
        Select-Object ModuleName, FileName, @{Name="Process";Expression={$_.BaseAddress.ProcessName}}
    } catch {}
}
```

#### 3. Check for Injected Code Using Sysinternals Tools (Optional CLI)

```powershell
.\sigcheck64.exe -e -m <dll_path>
```

#### 4. Scan for Suspicious Memory Regions

```powershell
# Requires Sysmon or third-party EDR for deep inspection
```

---

### Linux

#### 1. Check for Suspicious Shared Libraries in Memory

```bash
for pid in $(ls /proc | grep -E '^[0-9]+$'); do
  if [ -f /proc/$pid/maps ]; then
    grep '\.so' /proc/$pid/maps | grep -v '/lib' | grep -v '/usr/lib' && echo "PID: $pid"
  fi
done
```

#### 2. Detect Preload Injection via LD\_PRELOAD

```bash
env | grep LD_PRELOAD
strings /proc/*/environ | grep LD_PRELOAD
```

#### 3. Check for Overwritten `libc` or other critical libs

```bash
find /lib /usr/lib -type f -name "*.so*" -exec md5sum {} \; > known-good.txt
md5sum -c known-good.txt | grep -v OK
```

#### 4. Scan for Anomalous Processes and Memory Mappings

```bash
lsof | grep deleted | grep .so
```
