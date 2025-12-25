# 🚀 Linux Caching System

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-Bionic-orange)](https://ubuntu.com/)
[![Vagrant](https://img.shields.io/badge/Vagrant-2.0+-blue)](https://www.vagrantup.com/)
[![WireGuard](https://img.shields.io/badge/WireGuard-VPN-green)](https://www.wireguard.com/)

> **A high-performance, secure caching infrastructure for APT packages in Debian-based environments**

Developed as part of my thesis project at the **University of Bologna** (2023/2024), this system optimizes package management in corporate environments through intelligent caching and secure VPN connectivity.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture](#-architecture)
- [Technologies](#-technologies)
- [Getting Started](#-getting-started)
- [Configuration](#-configuration)
- [Results](#-results)
- [Contributing](#-contributing)
- [License](#-license)
- [Author](#-author)

---

## 🎯 Overview

In modern corporate environments, managing software packages across multiple machines can be challenging. Repeatedly downloading the same packages from remote repositories leads to:

- ⚠️ Excessive bandwidth consumption
- 🐌 Slow download speeds
- 💸 Increased operational costs
- 🔄 Network congestion

This project addresses these challenges by implementing a **centralized caching proxy** that stores downloaded packages locally and serves them to multiple clients through a **secure VPN tunnel**.

---

## ✨ Key Features

### 🎯 Performance

- **Smart Caching**: First download caches the package; subsequent requests served instantly
- **Bandwidth Optimization**: Reduces redundant downloads by up to 80%
- **Speed Improvement**: Significantly faster package installation across the network

### 🔒 Security

- **WireGuard VPN**: Modern, lightweight VPN with state-of-the-art cryptography
- **Encrypted Communications**: All traffic between clients and server is protected
- **Network Isolation**: Secure tunnel prevents unauthorized access

### 🛠️ Infrastructure

- **Automated Deployment**: Complete setup via Vagrant provisioning scripts
- **Reproducible Environments**: Consistent configuration across all instances
- **Scalable Architecture**: Easy to add new clients to the network

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Corporate Network                    │
│                                                         │
│  ┌──────────────┐         ┌──────────────┐            │
│  │   Client 1   │         │   Client 2   │            │
│  │  (Ubuntu)    │         │  (Ubuntu)    │            │
│  └──────┬───────┘         └──────┬───────┘            │
│         │                        │                     │
│         │   WireGuard VPN        │                     │
│         └────────┬───────────────┘                     │
│                  │                                      │
│         ┌────────▼─────────┐                           │
│         │  Cache Server    │                           │
│         │  apt-cacher-ng   │                           │
│         │  + WireGuard     │                           │
│         └────────┬─────────┘                           │
│                  │                                      │
└──────────────────┼─────────────────────────────────────┘
                   │
                   ▼
          Internet / APT Repositories
```

### Component Overview

| Component          | Technology            | Purpose                            |
| ------------------ | --------------------- | ---------------------------------- |
| **Virtualization** | Vagrant + VirtualBox  | Automated VM management            |
| **Cache Server**   | apt-cacher-ng         | Package caching proxy              |
| **VPN**            | WireGuard             | Secure client-server communication |
| **OS**             | Ubuntu 18.04 (Bionic) | Base system for all machines       |

---

## 💻 Technologies

### Core Stack

- **[Vagrant](https://www.vagrantup.com/)** - Infrastructure automation and VM management
- **[apt-cacher-ng](https://www.unix-ag.uni-kl.de/~bloch/acng/)** - Specialized APT package caching proxy
- **[WireGuard](https://www.wireguard.com/)** - Fast, modern VPN implementation
- **[VirtualBox](https://www.virtualbox.org/)** - Virtualization provider

### Why These Technologies?

**Vagrant over Docker/Multipass:**

- Complete OS-level isolation
- Better network control for VPN configuration
- Easier filesystem management for caching
- Reproducible environments with simple configuration

**apt-cacher-ng over Squid/Approx:**

- Optimized specifically for APT packages
- Native support for package signatures and dependencies
- Automatic cache management
- Simpler configuration for Debian-based systems

**WireGuard over OpenVPN/IPsec:**

- Superior performance (lower latency, faster speeds)
- Modern cryptography
- Minimal configuration complexity
- Smaller codebase = reduced attack surface

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed:

- [Vagrant](https://www.vagrantup.com/downloads) (2.0+)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (6.0+)
- Unix-based host system (Linux/macOS)

### Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/yourusername/linux-package-caching.git
   cd linux-package-caching
   ```

2. **Configure the environment**

   Edit the `Vagrantfile` if you need to customize:
   - Number of clients
   - Network configuration
   - Resource allocation (RAM, CPU)

3. **Deploy the infrastructure**

   ```bash
   vagrant up
   ```

   This command will:
   - Create and configure the cache server
   - Set up WireGuard VPN
   - Deploy client VMs
   - Run all provisioning scripts automatically

4. **Verify the setup**
   ```bash
   vagrant status
   ```

---

## ⚙️ Configuration

### Server Configuration

The server is automatically configured via `setup_apt_cacher.sh`:

- **apt-cacher-ng**: Listens on VPN interface (10.0.0.1:3142)
- **WireGuard**: Creates secure tunnel on UDP port 51820
- **Firewall**: Configured with iptables for proper routing

### Client Configuration

Clients are provisioned using `setup_client.sh`:

```bash
# APT proxy configuration is automatically set to:
Acquire::http::Proxy "http://10.0.0.1:3142";
```

### Adding New Clients

To connect external Debian-based machines:

1. **Copy the client setup script**

   ```bash
   scp setup_client.sh user@external-machine:~/
   ```

2. **Update the server endpoint** in the script:

   ```bash
   SERVER_ENDPOINT="<server-ip>:51820"
   ```

3. **Run the setup**

   ```bash
   chmod +x setup_client.sh
   ./setup_client.sh
   ```

4. **Add peer to server** (`/etc/wireguard/wg0.conf`):

   ```ini
   [Peer]
   PublicKey = <client-public-key>
   AllowedIPs = 10.0.0.X/32
   ```

5. **Restart WireGuard** on server:
   ```bash
   sudo wg-quick down wg0
   sudo wg-quick up wg0
   ```

---

## 📊 Results

The implementation demonstrated significant improvements:

### Performance Metrics

- ✅ **Bandwidth Reduction**: ~80% decrease in external bandwidth usage
- ✅ **Speed Improvement**: 3-5x faster package installation for cached packages
- ✅ **Network Efficiency**: Reduced load on upstream repositories

### Security

- 🔒 All client-server communications encrypted via WireGuard
- 🔒 Zero-trust network architecture
- 🔒 Protected against man-in-the-middle attacks

### Operational Benefits

- 🔄 Fully automated deployment and configuration
- 📦 Reproducible infrastructure as code
- 🎯 Easy scalability for additional clients

---

## 🧪 Testing

### Test with Vagrant Clients

1. **SSH into a client**

   ```bash
   vagrant ssh client-1
   ```

2. **Update package lists**

   ```bash
   sudo apt update
   ```

3. **Install a package**

   ```bash
   sudo apt install -y htop
   ```

4. **Verify caching** (install same package on client-2 - should be much faster)
   ```bash
   vagrant ssh client-2
   sudo apt install -y htop
   ```

### Monitor Cache Performance

Access the apt-cacher-ng web interface:

```
http://10.0.0.1:3142/acng-report.html
```

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Ideas for Contributions

- Support for additional Linux distributions (Fedora, CentOS)
- Docker-based implementation
- Monitoring and metrics dashboard
- Automated testing suite
- Cloud deployment guides (AWS, Azure, GCP)

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👨‍💻 Author

**Samite Mounir**

_Bachelor's Thesis in Computer Science and Engineering_  
Alma Mater Studiorum - Università di Bologna  
Academic Year 2023/2024

**Advisor:** Prof. Vittorio Ghini

---

## 🙏 Acknowledgments

Special thanks to:

- Prof. Vittorio Ghini for supervision and guidance
- The University of Bologna, School of Engineering
- The open-source community behind Vagrant, WireGuard, and apt-cacher-ng

---

## 📚 Additional Resources

- [Official Thesis Document (PDF)](https://amslaurea.unibo.it/id/eprint/32723/)
- [apt-cacher-ng Documentation](https://www.unix-ag.uni-kl.de/~bloch/acng/)
- [WireGuard Documentation](https://www.wireguard.com/quickstart/)
- [Vagrant Documentation](https://www.vagrantup.com/docs)

---

<div align="center">

**⭐ If you find this project useful, please consider giving it a star! ⭐**

Made with ❤️ for the DevOps and Linux

</div>
