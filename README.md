# Proxy Scanner

A powerful Bash-based proxy scanner for Linux and Android (Termux) that can scan IP ranges, detect open TCP ports, and validate HTTP and SOCKS5 proxy servers.

## Supported Platforms

✅ Linux (Ubuntu, Debian, Arch, Fedora)

✅ Android (Termux)

---

## Features

* Scan single IPs, IP ranges, or CIDR blocks
* Load IPs from text files
* Manual IP input support
* Fast parallel TCP port scanning
* HTTP proxy validation
* SOCKS5 proxy validation
* Automatic duplicate removal
* IPv4 address validation
* Configurable timeout settings
* Multi-threaded scanning for high performance
* Colored terminal output
* Result export to text files
* Linux and Android (Termux) support

---

## Requirements

### Required

* Bash 4+
* Netcat (`nc`)

### Optional

* Curl (required for proxy validation)

---

## Install Dependencies

### Debian / Ubuntu

```bash
sudo apt update
sudo apt install netcat-openbsd curl
```

### Arch Linux

```bash
sudo pacman -S openbsd-netcat curl
```

### Fedora

```bash
sudo dnf install nc curl
```

### Android (Termux)

```bash
pkg update && pkg upgrade -y
pkg install bash
pkg install netcat-openbsd
pkg install curl
```

Verify installation:

```bash
bash --version
nc -h
curl --version
```

---

## Installation

Clone the repository:

```bash
git clone https://github.com/yourusername/proxy-scanner.git
cd proxy-scanner
chmod +x proxy-scanner.sh
```

Run:

```bash
./proxy-scanner.sh
```

---

## Running on Android (Termux)

This script is fully compatible with Termux and does not require root access.

### Installation

```bash
pkg update && pkg upgrade -y
pkg install git bash netcat-openbsd curl

git clone https://github.com/yourusername/proxy-scanner.git
cd proxy-scanner

chmod +x proxy-scanner.sh
./proxy-scanner.sh
```

### Recommended Mobile Settings

For better battery life and performance on Android devices:

```text
TCP Scan Threads: 20-30
Proxy Validation Threads: 10-20
```

Avoid scanning very large ranges such as:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Large scans may consume significant CPU, RAM, and battery resources.

### Notes

* No root access required
* Supports HTTP and SOCKS5 proxy validation
* Supports CIDR and dash-notation ranges
* Results are saved locally in the Termux environment
* Works on Android phones and tablets

### Troubleshooting

If `nc` is not found:

```bash
pkg install netcat-openbsd
```

If `curl` is not found:

```bash
pkg install curl
```

If you are using the outdated Google Play version of Termux, install the latest version from F-Droid or GitHub.

---

## Supported Input Methods

### 1. CIDR Range

```text
192.168.1.0/24
10.0.0.0/16
```

### 2. Dash Notation

Short format:

```text
192.168.1.1-254
```

Full range:

```text
10.0.0.1-10.0.2.255
```

### 3. File Input

Example file:

```text
192.168.1.1
192.168.1.2
192.168.1.3
```

### 4. Manual Input

Paste IP addresses directly into the terminal.

---

## Scan Modes

### TCP Scan Only

Checks whether a TCP port is open.

### HTTP Proxy Validation

Verifies that an open port actually works as an HTTP proxy.

### SOCKS5 Proxy Validation

Verifies SOCKS5 proxy functionality.

### Combined Mode

Tests HTTP first and then SOCKS5 if HTTP validation fails.

---

## Output Files

### Open Ports

```text
tcp_open_YYYYMMDD_HHMMSS.txt
```

### Working Proxies

```text
working_proxies_YYYYMMDD_HHMMSS.txt
```

---

## Performance

Default values:

```text
TCP Scan Threads: 50
Proxy Validation Threads: 20
```

Increasing these values can significantly reduce scan time on powerful systems.

---

## Security Notice

This tool is intended for:

* Network administration
* Security testing
* Infrastructure auditing
* Authorized proxy discovery

Only scan systems and networks that you own or have explicit permission to test.

---

## Author

Lionprogram

Linux Bash & Zsh Network Tools

---

## License

MIT License

Feel free to use, modify, and distribute this project.
