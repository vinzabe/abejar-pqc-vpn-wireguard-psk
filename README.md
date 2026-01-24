# Abejar Post-Quantum Cryptography VPN Router

<p align="center">
  <strong>WireGuard + Pre-Shared Key Edition</strong>
</p>

<p align="center">
  <a href="https://github.com/vinzabe/abejar-pqc-vpn-wireguard-psk/actions/workflows/ci.yml"><img src="https://github.com/vinzabe/abejar-pqc-vpn-wireguard-psk/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
  <a href="https://github.com/vinzabe/abejar-pqc-vpn-wireguard-psk/pkgs/container/abejar-pqc-wireguard-psk"><img src="https://img.shields.io/badge/Docker-ghcr.io-blue?logo=docker" alt="Docker"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Commercial-red" alt="License"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/PQC-PSK%20%2B%20Kyber--1024-purple?style=for-the-badge" alt="PQC PSK"/>
  <img src="https://img.shields.io/badge/Security-Production--Ready-green?style=for-the-badge" alt="Production Ready"/>
</p>

---

## Overview

**Abejar PQC VPN Router (WireGuard + PSK Edition)** implements post-quantum security using standard WireGuard with enhanced pre-shared key (PSK) protection combined with Kyber key encapsulation.

This is the **most production-ready** variant, using battle-tested WireGuard with an additional layer of quantum-resistant security.

---

## Post-Quantum Cryptography Security

### Why PQC Matters

Current encryption (RSA, ECDH) is vulnerable to quantum computers using Shor's algorithm. Adversaries may be conducting **"harvest now, decrypt later"** attacks, storing encrypted traffic to decrypt once quantum computers become available.

### How This Solution Protects You

| Threat | Classical Protection | PQC Enhancement |
|--------|---------------------|-----------------|
| Quantum Key Recovery | X25519 (vulnerable) | Kyber-1024 encapsulated PSK |
| Harvest Now, Decrypt Later | Vulnerable | Protected |
| Man-in-the-Middle | Protected | Protected |
| Traffic Analysis | Protected | Protected |

### Security Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Security Layer Stack                         │
├─────────────────────────────────────────────────────────────────┤
│  Layer 4: ChaCha20-Poly1305 Encryption (Symmetric - QC Safe)   │
├─────────────────────────────────────────────────────────────────┤
│  Layer 3: Kyber-1024 Encapsulated PSK (Post-Quantum)           │
├─────────────────────────────────────────────────────────────────┤
│  Layer 2: Pre-Shared Key (256-bit)                             │
├─────────────────────────────────────────────────────────────────┤
│  Layer 1: X25519 Key Exchange (Classical)                       │
├─────────────────────────────────────────────────────────────────┤
│  Transport: WireGuard Protocol                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Quick Start

### Prerequisites

- Docker 24.0+
- Docker Compose 2.20+
- WireGuard VPN credentials from your provider

### Installation

```bash
# Clone the repository
git clone https://github.com/vinzabe/abejar-pqc-vpn-wireguard-psk.git
cd abejar-pqc-vpn-wireguard-psk

# Run setup script
./scripts/setup.sh

# Configure your environment
nano .env

# Add your WireGuard credentials
nano vpn-router/config/wireguard/wg0.conf

# Start the VPN router
docker compose up -d

# Verify status
./scripts/vpn-status.sh
```

### Using Pre-built Images

```bash
# Pull the pre-built image
docker pull ghcr.io/vinzabe/abejar-pqc-wireguard-psk:latest
```

---

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PQC_ENABLED` | `true` | Enable post-quantum cryptography |
| `PQC_ALGORITHM` | `kyber1024-psk` | PQC algorithm for key encapsulation |
| `PQC_HYBRID_MODE` | `true` | Use hybrid X25519 + Kyber mode |
| `PQC_KEY_ROTATION_HOURS` | `24` | PSK rotation interval |
| `VPN_INTERFACE` | `wg0` | WireGuard interface name |
| `VPN_NETWORK_CIDR` | `172.25.0.0/16` | Docker network CIDR |
| `VPN_ROUTER_IP` | `172.25.0.2` | VPN router container IP |
| `APP_PORT_START` | `10091` | Starting port for app mappings |
| `APP_PORT_END` | `10100` | Ending port for app mappings |
| `CF_TUNNEL_TOKEN` | - | Cloudflare Tunnel token |
| `CF_TUNNEL_PQC` | `true` | Enable PQC for Cloudflare Tunnel |
| `LOG_LEVEL` | `info` | Logging level |
| `LOG_FORMAT` | `json` | Log format (json/text) |

### WireGuard Configuration

Place your VPN provider's WireGuard configuration in `vpn-router/config/wireguard/wg0.conf`:

```ini
[Interface]
PrivateKey = <your-private-key>
Address = 10.x.x.x/32
DNS = 1.1.1.1

[Peer]
PublicKey = <vpn-provider-public-key>
PresharedKey = <psk-key>  # Enhanced with Kyber encapsulation
AllowedIPs = 0.0.0.0/0
Endpoint = <vpn-server>:51820
PersistentKeepalive = 25
```

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        Docker Host                                │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    VPN Router Container                     │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │  │
│  │  │  WireGuard   │  │   Kyber-1024 │  │  Traffic Router  │  │  │
│  │  │  Protocol    │──│   PSK Encaps │──│                  │  │  │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘  │  │
│  └─────────────────────────────┬──────────────────────────────┘  │
│                                │                                  │
│  ┌─────────────────────────────┼──────────────────────────────┐  │
│  │              Cloudflared Container                          │  │
│  │  ┌──────────────┐          │          ┌──────────────────┐ │  │
│  │  │ PQC TLS 1.3  │──────────┴──────────│ Tunnel Protocol  │ │  │
│  │  └──────────────┘                     └──────────────────┘ │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                │                                  │
│  ┌─────────────────────────────┼──────────────────────────────┐  │
│  │              Application Containers                         │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │  │
│  │  │  Web App │  │   API    │  │ Database │  │  Cache   │   │  │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
                    ┌──────────────────────┐
                    │  VPN Provider Server │
                    │  (Quantum-Resistant) │
                    └──────────────────────┘
                                │
                                ▼
                    ┌──────────────────────┐
                    │      Internet        │
                    └──────────────────────┘
```

---

## Security Model

| Layer | Technology | Protection Level |
|-------|------------|------------------|
| Transport | WireGuard | Modern VPN protocol |
| Key Exchange | X25519 | Classical security |
| PQC Layer | PSK + Kyber-1024 | Quantum resistance |
| Encryption | ChaCha20-Poly1305 | Symmetric (QC safe) |
| Ingress | Cloudflare PQC TLS | Edge quantum protection |

---

## Verification

```bash
# Check VPN connection status
./scripts/vpn-status.sh

# Test PQC handshake
./scripts/test-pqc-handshake.sh

# Verify external IP shows VPN
docker exec vpn-router curl -s https://ipinfo.io/ip

# Test end-to-end connectivity
./scripts/test-connection.sh
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [SECURITY.md](SECURITY.md) | Detailed security analysis and threat model |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System architecture deep dive |
| [docs/INSTALLATION.md](docs/INSTALLATION.md) | Detailed installation guide |
| [docs/CLOUDFLARE_SETUP.md](docs/CLOUDFLARE_SETUP.md) | Cloudflare Tunnel configuration |

---

## Examples

See the [examples/](examples/) directory for complete deployment examples:

- **SaaS Application**: Full-stack app with database and Redis
- **Simple Web App**: Basic nginx deployment

---

## License

This software is commercially licensed. Pre-built Docker images are provided for evaluation.

For **full source code**, **enterprise licensing**, and **support**:

| | |
|---|---|
| **Email** | grant@abejar.net |
| **Subject** | Abejar PQC VPN - License Inquiry |

---

<p align="center">
  <sub>Copyright 2024 Abejar. All rights reserved.</sub>
</p>
