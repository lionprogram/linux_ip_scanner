#!/usr/bin/env bash

# ========================================================== #
#                     Proxy Scanner                          #
#       Lionprogram - Linux bash & zsh - Ip scanner          #
# ========================================================== #
#
# Description:
#   Scan IP ranges and validate HTTP/SOCKS5 proxies.
#
# Requirements:
#   nc (netcat-openbsd)
#   curl (for proxy validation)
#
# Usage:
#   ./proxy-scanner.sh
#
# Author:
#   Lionprogram
#
# License:
#   MIT

# ------------------------- Color definitions -------------------------
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
NC='\033[0m'               # No Color

# ------------------------- Prerequisite check -------------------------
if ! command -v nc &> /dev/null; then
    echo -e "${RED}Error: 'nc' (netcat) is required. Please install netcat-openbsd.${NC}"
    exit 1
fi

# ----------------------------------------------------------------------
# Function: normalize_ip_list
# Filters and deduplicates a list of IPv4 addresses.
# Arguments: list of raw IP strings
# Output: unique, valid IPv4 addresses (one per line)
# ----------------------------------------------------------------------
normalize_ip_list() {
    local -a raw=("$@")
    local -a valid=()
    local -A seen
    local ip clean_ip

    for ip in "${raw[@]}"; do
        clean_ip=$(echo "$ip" | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//')
        [[ -z "$clean_ip" ]] && continue

        if [[ "$clean_ip" =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
            if [[ -z "${seen[$clean_ip]}" ]]; then
                seen[$clean_ip]=1
                valid+=("$clean_ip")
            fi
        else
            echo -e "${RED}⚠️  Invalid IP skipped: $clean_ip${NC}" >&2
        fi
    done

    printf '%s\n' "${valid[@]}"
}

# ================== NEW: IP RANGE EXPANSION FUNCTIONS ==================
ip_to_int() {
    local ip="$1"
    local a b c d
    IFS=. read -r a b c d <<< "$ip"
    echo $(( (a << 24) + (b << 16) + (c << 8) + d ))
}

int_to_ip() {
    local int="$1"
    echo "$(( (int >> 24) & 0xFF )).$(( (int >> 16) & 0xFF )).$(( (int >> 8) & 0xFF )).$(( int & 0xFF ))"
}

# ----------------------------------------------------------------------
# Function: generate_cidr_ips
#
# Description:
#   Expands a CIDR subnet into individual IPv4 addresses.
#
# Arguments:
#   $1 - CIDR notation
#        Example: 192.168.1.0/24
#
# Output:
#   IPv4 addresses (one per line)
#
# Returns:
#   0 on success
# ----------------------------------------------------------------------
generate_cidr_ips() {
    local cidr="$1"
    local network="${cidr%/*}"
    local mask="${cidr#*/}"
    local start_int end_int i

    start_int=$(ip_to_int "$network")
    # Calculate broadcast address: start_int | (0xFFFFFFFF >> mask)
    end_int=$(( start_int | (0xFFFFFFFF >> mask) ))
    # Exclude network and broadcast for /31 and /32? For simplicity, include all.
    # But for /31 and /32, just return the single IP if needed.
    if [[ $mask -eq 32 ]]; then
        echo "$network"
        return
    fi
    # For normal ranges, iterate from start_int+1 to end_int-1 (exclude network/broadcast)
    # However many scanners include network/broadcast - we'll include all for flexibility.
    for (( i=start_int; i<=end_int; i++ )); do
        int_to_ip "$i"
    done
}

generate_dash_range_ips() {
    local range="$1"
    local start_ip end_ip
    start_ip="${range%-*}"
    end_ip="${range#*-}"
    # If end_ip does not contain dots, treat as last octet only (legacy style)
    if [[ "$end_ip" != *.* ]]; then
        # Legacy: same first three octets as start_ip
        local prefix="${start_ip%.*}"
        local start_last="${start_ip##*.}"
        for (( i=start_last; i<=end_ip; i++ )); do
            echo "$prefix.$i"
        done
        return
    fi
    # Full IP range (e.g., 192.168.1.1-192.168.2.254)
    local start_int end_int i
    start_int=$(ip_to_int "$start_ip")
    end_int=$(ip_to_int "$end_ip")
    if [[ $start_int -gt $end_int ]]; then
        echo -e "${RED}Start IP is greater than end IP.${NC}" >&2
        return
    fi
    for (( i=start_int; i<=end_int; i++ )); do
        int_to_ip "$i"
    done
}
# =========================================================================

# ----------------------------------------------------------------------
# Step 1: Choose IP input method
# ----------------------------------------------------------------------
echo -e "${CYAN}========== INPUT METHOD ==========${NC}"
echo "1) Scan an IP range (e.g., 192.168.1.1-254 or 192.168.1.0/24)"
echo "2) Load IP list from a text file (one IP per line)"
echo "3) Enter IPs manually (paste – for small lists only)"

read -p "Choose [1, 2 or 3]: " input_method

ip_list=()

if [[ "$input_method" == "2" ]]; then
    read -p "Enter path to text file (e.g., /home/user/ips.txt): " file_path
    if [[ ! -f "$file_path" ]]; then
        echo -e "${RED}File not found: $file_path${NC}"
        exit 1
    fi
    raw_ips=()
    while IFS= read -r line; do
        raw_ips+=("$line")
    done < "$file_path"
    mapfile -t ip_list < <(normalize_ip_list "${raw_ips[@]}")
    if [[ ${#ip_list[@]} -eq 0 ]]; then
        echo -e "${RED}No valid IPs found in file. Exiting.${NC}"
        exit 1
    fi
    echo -e "${GREEN}✓ Loaded ${#ip_list[@]} unique, valid IP(s) from file.${NC}"

elif [[ "$input_method" == "3" ]]; then
    echo -e "${YELLOW}Enter IP addresses (one per line). When done, press ENTER, then Ctrl+D (or ENTER twice):${NC}"
    raw_ips=()
    while IFS= read -r line; do
        [[ -z "$line" ]] && break
        raw_ips+=("$line")
    done
    mapfile -t ip_list < <(normalize_ip_list "${raw_ips[@]}")
    if [[ ${#ip_list[@]} -eq 0 ]]; then
        echo -e "${RED}No valid IPs entered. Exiting.${NC}"
        exit 1
    fi
    echo -e "${GREEN}✓ ${#ip_list[@]} unique, valid IP(s) loaded.${NC}"

else
    # Method 1: IP range (CIDR or dash notation) - ENHANCED
    read -p "Enter IP range (e.g., 192.168.1.1-254 or 192.168.1.0/24 or 10.0.0.1-10.0.2.255): " ip_range
    ip_range=$(echo "$ip_range" | xargs)

    # Store generated addresses before validation so
    # duplicates and malformed entries can be removed
    # by normalize_ip_list().
    # local -a raw_range_ips=()
    local -a raw_range_ips=()

    if [[ $ip_range == *"/"* ]]; then
        # CIDR notation (any valid mask)
        mapfile -t raw_range_ips < <(generate_cidr_ips "$ip_range")
        if [[ ${#raw_range_ips[@]} -eq 0 ]]; then
            echo -e "${RED}Invalid CIDR notation.${NC}"
            exit 1
        fi
    elif [[ $ip_range == *"-"* ]]; then
        # Dash range (supports both old and new style)
        mapfile -t raw_range_ips < <(generate_dash_range_ips "$ip_range")
        if [[ ${#raw_range_ips[@]} -eq 0 ]]; then
            echo -e "${RED}Invalid dash range.${NC}"
            exit 1
        fi
    else
        # Single IP
        raw_range_ips=("$ip_range")
    fi

    # Normalize and deduplicate (in case of overlaps)
    mapfile -t ip_list < <(normalize_ip_list "${raw_range_ips[@]}")
    if [[ ${#ip_list[@]} -eq 0 ]]; then
        echo -e "${RED}No valid IPs generated from the range. Exiting.${NC}"
        exit 1
    fi
    echo -e "${GREEN}✓ Generated ${#ip_list[@]} unique IP(s) from range.${NC}"
fi

# ----------------------------------------------------------------------
# Common scan parameters
# ----------------------------------------------------------------------
read -p "Enter port(s) separated by space (e.g., 8080 3128 1080 8000): " -a ports
read -p "TCP timeout (seconds, default 3): " tcp_timeout
tcp_timeout=${tcp_timeout:-3}
if [[ ! "$tcp_timeout" =~ ^[0-9]+$ ]]; then
    echo -e "${RED}Invalid TCP timeout. Using default 3.${NC}"
    tcp_timeout=3
fi

read -p "Max parallel TCP scans (default 50): " max_jobs
max_jobs=${max_jobs:-50}
if [[ ! "$max_jobs" =~ ^[0-9]+$ ]]; then
    echo -e "${RED}Invalid number for parallel jobs. Using default 50.${NC}"
    max_jobs=50
fi

# ----------------------------------------------------------------------
# Step 2: Scan mode selection (TCP only / HTTP / SOCKS5 / both)
# ----------------------------------------------------------------------
echo ""
echo -e "${CYAN}========== SCAN MODE ==========${NC}"
echo "1) Only TCP scan (find open ports, no proxy test)"
echo "2) Test HTTP proxy (requires curl)"
echo "3) Test SOCKS5 proxy (requires curl)"
echo "4) Test both HTTP and SOCKS5 proxies"
read -p "Choose [1-4]: " scan_mode

if [[ $scan_mode -ge 2 && $scan_mode -le 4 ]]; then
    if ! command -v curl &> /dev/null; then
        echo -e "${RED}Error: 'curl' is required for proxy testing. Install it.${NC}"
        exit 1
    fi
    read -p "Test URL (default: http://httpbin.org/ip): " test_url
    test_url=${test_url:-http://httpbin.org/ip}
    test_url_https=${test_url_https:-https://httpbin.org/ip}

    echo ""
    echo -e "${CYAN}========== PROXY TIMEOUT SETTINGS ==========${NC}"
    echo "Select network condition:"
    echo "1) Fast    (conn 2s, max 5s)"
    echo "2) Medium  (conn 4s, max 8s)"
    echo "3) Slow    (conn 6s, max 12s)"
    echo "4) Custom"
    read -p "Choose [1-4] (default 2): " net_opt
    net_opt=${net_opt:-2}
    case $net_opt in
        1) conn_t=2; max_t=5 ;;
        2) conn_t=4; max_t=8 ;;
        3) conn_t=6; max_t=12 ;;
        4) read -p "Connect timeout (s): " conn_t
           read -p "Max time (s): " max_t
           ;;
        *) conn_t=4; max_t=8 ;;
    esac
    if [[ ! "$conn_t" =~ ^[0-9]+$ ]]; then conn_t=4; fi
    if [[ ! "$max_t" =~ ^[0-9]+$ ]]; then max_t=8; fi

    read -p "Follow redirects? (y/n, default y): " follow
    follow=${follow:-y}
    if [[ "$follow" == "y" ]]; then
        FOLLOW="-L"
    else
        FOLLOW=""
    fi
fi

total_ips=${#ip_list[@]}
echo -e "${CYAN}🔎 Scanning $total_ips IPs on ${#ports[@]} ports${NC}"

# ----------------------------------------------------------------------
# Phase 1: TCP connect scan (parallel)
# ----------------------------------------------------------------------
tmp_open=$(mktemp)
running=0

test_tcp() {
    local ip="$1"
    local port="$2"
    local out="$3"
    if nc -zv -w "$tcp_timeout" "$ip" "$port" 2>&1 | grep -qi "succeeded\|connected"; then
        echo "$ip:$port" >> "$out"
        echo -e "${GREEN}✅ TCP OPEN: $ip:$port${NC}"
    fi
}
export -f test_tcp
export tcp_timeout

for ip in "${ip_list[@]}"; do
    for port in "${ports[@]}"; do
        ( test_tcp "$ip" "$port" "$tmp_open" ) &
        ((running++))
        if [[ $running -ge $max_jobs ]]; then
            wait -n
            ((running--))
        fi
    done
done
wait

mapfile -t open_list < "$tmp_open"
open_count=${#open_list[@]}

echo ""
echo -e "${CYAN}========== TCP SCAN RESULT ==========${NC}"
if [[ $open_count -eq 0 ]]; then
    echo -e "${RED}No open ports found. Exiting.${NC}"
    rm -f "$tmp_open"
    exit 0
else
    echo -e "${GREEN}Found $open_count open TCP port(s):${NC}"
    printf '%s\n' "${open_list[@]}"
fi

# If only TCP scan was requested, save results and exit
if [[ $scan_mode -eq 1 ]]; then
    final_file="tcp_open_$(date +%Y%m%d_%H%M%S).txt"
    cp "$tmp_open" "$final_file"
    echo -e "${GREEN}✓ Open ports saved to: $final_file${NC}"
    rm -f "$tmp_open"
    exit 0
fi

# ----------------------------------------------------------------------
# Phase 2: Proxy validation (HTTP / SOCKS5)
# ----------------------------------------------------------------------
echo ""
echo -e "${CYAN}========== PROXY VALIDATION PHASE ==========${NC}"
read -p "Max parallel proxy tests (default 20): " proxy_jobs
proxy_jobs=${proxy_jobs:-20}
if [[ ! "$proxy_jobs" =~ ^[0-9]+$ ]]; then
    proxy_jobs=20
fi

final_file="working_proxies_$(date +%Y%m%d_%H%M%S).txt"
> "$final_file"
tmp_result=$(mktemp)
running=0

test_http_proxy() {
    local ip="$1"
    local port="$2"
    local url="$3"
    local url_https="$4"
    local out="$5"

    code=$(curl -s -o /dev/null -w "%{http_code}" -x "http://$ip:$port" $FOLLOW "$url" --connect-timeout "$conn_t" --max-time "$max_t" 2>/dev/null)
    if [[ $? -eq 0 && "$code" == "200" ]]; then
        echo "$ip:$port (HTTP proxy)" >> "$out"
        echo -e "${GREEN}✅ HTTP PROXY WORKING (HTTP 200)${NC}"
        return 0
    fi

    code2=$(curl -s -o /dev/null -w "%{http_code}" -x "http://$ip:$port" $FOLLOW "$url_https" --connect-timeout "$conn_t" --max-time "$max_t" 2>/dev/null)
    if [[ $? -eq 0 && "$code2" == "200" ]]; then
        echo "$ip:$port (HTTP proxy via HTTPS)" >> "$out"
        echo -e "${GREEN}✅ HTTP PROXY WORKING (via HTTPS, code $code2)${NC}"
        return 0
    fi

    echo -e "${YELLOW}⚠️  $ip:$port not HTTP proxy (codes: $code / $code2)${NC}"
    return 1
}

test_socks5_proxy() {
    local ip="$1"
    local port="$2"
    local url="$3"
    local out="$4"

    if curl --socks5 "$ip:$port" "$url" -s -o /dev/null --connect-timeout "$conn_t" --max-time "$max_t" 2>/dev/null; then
        echo "$ip:$port (SOCKS5)" >> "$out"
        echo -e "${GREEN}✅ SOCKS5 PROXY WORKING${NC}"
        return 0
    else
        echo -e "${YELLOW}⚠️  $ip:$port not SOCKS5 proxy${NC}"
        return 1
    fi
}

export -f test_http_proxy test_socks5_proxy
export conn_t max_t FOLLOW test_url test_url_https

for target in "${open_list[@]}"; do
    ip="${target%:*}"
    port="${target#*:}"
    (
        if [[ $scan_mode -eq 2 ]]; then
            test_http_proxy "$ip" "$port" "$test_url" "$test_url_https" "$tmp_result"
        elif [[ $scan_mode -eq 3 ]]; then
            test_socks5_proxy "$ip" "$port" "$test_url" "$tmp_result"
        elif [[ $scan_mode -eq 4 ]]; then
            test_http_proxy "$ip" "$port" "$test_url" "$test_url_https" "$tmp_result"
            if [[ $? -ne 0 ]]; then
                test_socks5_proxy "$ip" "$port" "$test_url" "$tmp_result"
            fi
        fi
    ) &
    ((running++))
    if [[ $running -ge $proxy_jobs ]]; then
        wait -n
        ((running--))
    fi
done
wait

cat "$tmp_result" >> "$final_file"
working_count=$(wc -l < "$final_file" 2>/dev/null || echo 0)

# ----------------------------------------------------------------------
# Final summary
# ----------------------------------------------------------------------
echo ""
echo -e "${CYAN}========== FINAL SUMMARY ==========${NC}"
echo -e "TCP open ports found: ${GREEN}$open_count${NC}"
echo -e "Working proxies detected: ${GREEN}$working_count${NC}"
if [[ $working_count -gt 0 ]]; then
    echo -e "Saved to: ${CYAN}$final_file${NC}"
    cat "$final_file"
else
    echo -e "${RED}No working proxies found.${NC}"
fi

# Cleanup temporary files
rm -f "$tmp_open" "$tmp_result"
