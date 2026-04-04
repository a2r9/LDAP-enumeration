# LDAP-Pulse

Author : Abhijit Rekhi

LDAP-Pulse is an automated Active Directory enumeration and exploitation tool. It leverages LDAP to query the Directory Information Tree (DIT) for misconfigurations—specifically, cleartext credentials stored in user description and info attributes. Upon identifying exposed credentials, it dynamically generates a resource script and hands execution off to the Metasploit Framework to establish a Meterpreter session via SMB (PsExec).
## Operational Flow
 1. **Enumeration:** Authenticates to the Domain Controller via LDAP (Authenticated Bind).
 2. **Harvesting:** Dynamically maps the domain search base and queries all user objects for populated description and info fields.
 3. **Parsing:** Utilizes regular expressions to extract potential passwords from identified fields.
 4. **Weaponization:** Generates an .rc file pre-configured with the target's IP, extracted credentials, and designated payload (windows/meterpreter/reverse_tcp).
 5. **Execution:** Triggers msfconsole in quiet mode to execute the attack chain and catch the shell.
## Prerequisites
 * Python 3.x
 * ldap3 library (pip install ldap3)
 * Metasploit Framework (msfconsole must be in your system PATH)
## Configuration
Before execution, update the TARGET_CONFIG dictionary within ldap_pulse.py:
```python
TARGET_CONFIG = {
    'dc_ip': '192.168.1.10',       # Target Domain Controller IP
    'domain': 'corp.local',        # Target Domain Name
    'user': 'service_account',     # Compromised AD Username
    'password': 'Password123!',    # Compromised AD Password
    'lhost': '192.168.1.50'        # Attacker IP (for the reverse shell)
}

```
## Usage
Execute the script from a terminal with Metasploit access:
```bash
python3 ldap_pulse.py

```
If vulnerable accounts are found, the tool will prompt for authorization before launching the PsExec module against the target.

## Disclaimer
This tool is designed for authorized Red Team operations and security auditing. Do not execute against infrastructure you do not own or have explicit, documented permission to test.
