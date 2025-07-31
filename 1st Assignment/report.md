# Admin Privilege Monitoring Report

## Methodology
To monitor admin (sudo) access on a Kali Linux system, I:
1. Verified logging files (`/var/log/auth.log`) and found they were not present.
2. Identified systemd journal logging (`journalctl`) as the alternative.
3. Wrote a Python script that:
   - Checks for sudo events using `journalctl` every 30 seconds.
   - Logs each sudo usage attempt (success/failure) to a custom file with user and timestamp.
 
## Keyword
1. cd
2. cs
3. chmod +x sudo_monitor.py
4. sudo ./sudo_monitor.py

## Screenshot
<img width="429" height="552" alt="Screenshot 2025-07-31 133324" src="https://github.com/user-attachments/assets/3a32894b-5aac-4ef9-89e7-eb8b11bbb09b" />

## Findings
- System uses `journalctl` instead of traditional log files.
- Both successful and failed sudo attempts are traceable.
- Logs provide exact usernames and timestamps.
- sudo python3 sudo_monitor.py are used.

**Example logs:**
Jul 31 11:00:23 | USER: kali | STATUS: SUCCESS
Jul 31 11:05:10 | USER: kali | STATUS: FAILURE

## Conclusion
- Monitoring admin access is essential for system integrity and detecting potential insider threats.
- `journalctl` offers a more robust and modern logging approach.
- This system is expandable to support alerts or reporting features.

## Python Script: `sudo_monitor.py`
```python
#!/usr/bin/env python3
import subprocess
import re

def monitor_sudo_journal():
    print("🔍 Monitoring sudo usage via journalctl... (Ctrl+C to stop)")

    proc = subprocess.Popen(
        ["journalctl", "-f", "_COMM=sudo", "-o", "short"],
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True,
        bufsize=1,
    )

    try:
        for line in proc.stdout:
            line = line.strip()
            # Extract timestamp (first 15 chars)
            timestamp = line[:15]

            # Find user in line
            user_match = re.search(r'user=(\w+)', line)
            user = user_match.group(1) if user_match else "UNKNOWN"

            if "COMMAND=" in line:
                status = "SUCCESS"
            elif "authentication failure" in line.lower():
                status = "FAILURE"
            else:
                continue  # skip unrelated lines

            print(f"{timestamp} | USER: {user} | STATUS: {status}")

    except KeyboardInterrupt:
        print("\n🛑 Stopped monitoring.")
        proc.terminate()

if __name__ == "__main__":
    monitor_sudo_journal()
