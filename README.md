# Proxy Scanner

A powerful Bash-based proxy scanner for Linux that can scan IP ranges, detect open TCP ports, and validate HTTP and SOCKS5 proxy servers.

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

---

## Requirements

### Required

* Bash 4+
* Netcat (`nc`)

### Optional

* Curl (required for proxy validation)

### Install Dependencies

#### Debian / Ubuntu

```bash
sudo apt update
sudo apt install netcat-openbsd curl
```

#### Arch Linux

```bash
sudo pacman -S openbsd-netcat curl
```

#### Fedora

```bash
sudo dnf install nc curl
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

Example:

```text
192.168.1.10:8080
```

### HTTP Proxy Validation

Verifies that an open port actually works as an HTTP proxy.

### SOCKS5 Proxy Validation

Verifies SOCKS5 proxy functionality.

### Combined Mode

Tests HTTP first and then SOCKS5 if HTTP validation fails.

---

## Example Usage

### Scan Common Proxy Ports

```bash
Ports:
8080 3128 1080 8000
```

### Scan CIDR Range

```text
192.168.1.0/24
```

### Scan a Single Host

```text
192.168.1.100
```

---

## Output Files

### Open Ports

```text
tcp_open_YYYYMMDD_HHMMSS.txt
```

Example:

```text
192.168.1.20:8080
192.168.1.30:3128
```

### Working Proxies

```text
working_proxies_YYYYMMDD_HHMMSS.txt
```

Example:

```text
192.168.1.20:8080 (HTTP proxy)
192.168.1.50:1080 (SOCKS5)
```

---

## Performance

The scanner supports configurable parallel jobs:

```text
TCP Scan Threads: Default 50
Proxy Validation Threads: Default 20
```

Increasing these values can significantly reduce scan time on large networks.

---

## Security Notice

This tool is intended for:

* Network administration
* Security testing
* Infrastructure auditing
* Authorized proxy discovery

Only scan systems and networks that you own or have permission to test.

---

## Project Structure

```text
proxy-scanner.sh
README.md
```

---

## Author

Lionprogram

Linux Bash & Zsh Network Tools

---

## License

MIT License

Feel free to use, modify, and distribute this project.
