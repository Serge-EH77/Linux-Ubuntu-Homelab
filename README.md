---
# 📦 Homelab Ubuntu Server

[Ubuntu Server Documentation](https://ubuntu.com/server/docs/)

**Infrastructure Stack:**
Dell OptiPlex → Ubuntu Server → Docker → MySQL → Nginx → Tailscale

This repository documents the complete chronological journey of transforming a Dell OptiPlex into a fully functional homelab server. It serves as a comprehensive technical log, detailing every step of the configuration process, the specific challenges encountered during deployment, and the methodologies used to resolve them.

### Skills Demonstrated:
* **Linux Server Administration:** Advanced terminal operations, package management, and system optimization.
* **Docker Containerization:** Orchestrating multi-container environments and managing persistence.
* **Database Management:** Configuring and securing MySQL instances within Docker.
* **Network & Security:** Implementing secure remote access via Tailscale and managing SSH user permissions.
* **Web Services:** Configuring Nginx as a reverse proxy for high-availability web hosting.
* **Architecture Design:** Scaling hardware resources for homelab reliability and performance.

This documentation is intended for developers and enthusiasts looking to transition from local development to a structured, server-side homelab environment. Each section is designed to be modular, allowing users to replicate specific parts of the architecture or follow the entire setup guide from bare metal to a production-ready state.
---
🧭 Table of Contents

Hardware Overview

Ubuntu Server Installation

Core Package Installation

Docker & Portainer Setup

MySQL Docker Container

Problems Encountered & Fixes

Backend Connectivity

SSH User Management

Tailscale Remote Access

Portfolio Hosting

Final Architecture Diagram

Final State of the Homelab

Skills Demonstrated

---
🖥️ 1. Hardware Overview

Dell OptiPlex 7060 SFF

Component	Specification

CPU	Intel Core i5/i7

RAM	16 GB

Storage	250 GB SSD

Network	Gigabit Ethernet

OS	Ubuntu Server 22.04 LTS

This machine became the foundation of the homelab.

---
🧑‍💻 2. Installing Ubuntu Server

BIOS Setup

Enabled Virtualization (VT‑x)

Disabled Secure Boot

Set UEFI boot mode

Booted from USB

**Ubuntu Installation**

Installed Ubuntu Server 22.04 LTS 

Enabled OpenSSH Server

Connected via Ethernet

Created main user

Initial Updates

sudo apt update && sudo apt upgrade -y

---
🧰 3. Installing Core Server Packages

_sudo apt install -y docker.io docker-compose nginx mysql-client openssh-server ufw_

Enabled firewall:

_sudo ufw allow OpenSSH_

_sudo ufw enable_

---
🐳 4. Installing Docker & Portainer

Enabled Docker:

_sudo systemctl enable --now docker_

Installed Portainer:

_sudo docker run -d -p 9000:9000 --name portainer \-v /var/run/docker.sock:/var/run/docker.sock \-v portainer_data:/data \portainer/portainer-ce_

Portainer accessible at:
_http://<server-ip>:9000_

To find Ip: 
_sudo ip a_

---
🗄️ 5. Creating the MySQL Docker Container

---
🗄️ 5. Creating the MySQL Docker Container

Create a `docker-compose.yml` file:

```yaml
version: "3.8"
services:
  mysql:
    image: mysql:5.7
    container_name: choose_a_name
    environment:
      MYSQL_ROOT_PASSWORD: your_secret_password
      MYSQL_DATABASE: db_name
    volumes:
      - /home/username/container_name:/var/lib/mysql
    networks:
      - aware_net
    ports:
      - "3306:3306"
    restart: unless-stopped

networks:
  aware_net:
    driver: bridge
```

Start MySQL:

_docker compose up -d_

---

console.error('Connection failed:', err.message);
  }
}

testConnection();

---

### Implementation Details:
* **`mysql2/promise`**: This module is used to leverage `async/await` syntax, which simplifies handling asynchronous database operations compared to traditional callback-based methods.
* **IP Address Configuration**: Ensure `ip_adresss` is replaced with the actual static IP of your Ubuntu server, not the internal container address.
* **Security Consideration**: While `'%'` allows access from any host, it is best practice to restrict this to a specific subnet (e.g., `'[###########]/24'`) if your environment allows for stricter firewall rules.
* **Dependencies**: Before running this script, ensure you have initialized your Node.js project and installed the driver by running `npm install mysql2`.

### Troubleshooting Checklist:
* **Firewall (UFW)**: Verify that the Ubuntu host's firewall allows traffic on port 3306 using `sudo ufw allow 3306/tcp`.
* **Credentials**: Confirm that the database user provided in the script has explicitly been granted privileges for the specific database being queried.
* **Driver Status**: If you encounter a "Module not found" error, verify the `node_modules` folder exists in your project directory.
---
🟦 7. Testing Backend Connectivity 

```
test.js
const mysql = require('mysql2/promise');

async function testConnection() {
  try {
    const connection = await mysql.createConnection({
      host: 'ip_adresss',
      user: 'user_name',
      password: 'password',
      database: 'db_name'
    });

    console.log('Connected to MySQL successfully!');

    const [rows] = await connection.execute('SELECT 1 + 1 AS result');
    console.log('Query test (1 + 1):', rows[0].result);

    await connection.end();
  } catch (err) {
    console.error('Connection failed:', err.message);
  }
}

testConnection();
```
Run:
node test.js

Output:
Connected to MySQL successfully!
Query test (1 + 1): 2

Backend connectivity solved.
---
🔒 8. Adding SSH Users for Teammates

To facilitate collaboration on your homelab, it is essential to manage access securely. Follow these steps to provision Linux and MySQL users for your team members:

**Linux User Management:**
Create dedicated system accounts for each teammate to provide them with individual environments:
```bash
sudo adduser teammate1
sudo adduser teammate2
```

**MySQL Permission Configuration:**
Restrict database access by creating specific MySQL users limited to local connections, ensuring they only have the necessary permissions to perform their tasks:
```sql
CREATE USER 'teammate1'@'localhost' IDENTIFIED BY 'pass1';
GRANT SELECT, INSERT, UPDATE ON db_name.* TO 'teammate1'@'localhost';
FLUSH PRIVILEGES;
```

**Accessing the Environment:**
Once the accounts are provisioned, teammates can access the server and database using the following commands:
*   **SSH Access:** `ssh teammate1@hostip_address`
*   **Database Access:** `mysql -u teammate1 -p db_name`

**Security Best Practices:**
*   **Principle of Least Privilege:** Always grant the minimum permissions required for a teammate to perform their work (e.g., use `SELECT` only if they do not need to modify data).
*   **SSH Key Authentication:** For better security than passwords, consider disabling password authentication and having teammates add their public SSH keys to `/home/teammate1/.ssh/authorized_keys`.
*   **Auditing:** Periodically run `last` or check `/var/log/auth.log` to monitor who has accessed the server and when.
  
🔒 9. Installing Tailscale for Remote Access

Tailscale simplifies secure remote connectivity by creating a peer-to-peer mesh VPN based on the WireGuard protocol. This allows you to access your home server from anywhere without exposing ports to the public internet.

**Installation Steps:**
1. Install Tailscale:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```
2. Authenticate and start the service:
   ```bash
   sudo tailscale up
   ```

**Remote Access Details:**
Once connected, your server will be assigned a unique private Tailscale IP address (e.g., `100.x.x.x`). You can then access your services securely using this IP:

*   **SSH Access:** `ssh username@100.x.x.x`
*   **Portainer Interface:** `http://100.x.x.x:9000`
*   **MySQL Database:** `mysql -h 100.x.x.x -u team3 -p smartsplit`
*   **Portfolio Website:** `http://100.x.x.x:8080`

**Key Benefits:**
*   **No Port Forwarding:** Eliminates the need to open ports on your router, significantly reducing the attack surface.
*   **End-to-End Encryption:** Traffic is encrypted between devices using WireGuard.
*   **Ease of Use:** Works seamlessly across different networks and NAT configurations.

For further configuration and advanced settings, refer to the [official Tailscale Documentation](https://tailscale.com/docs).
---

🌐 10. Hosting My Portfolio
To publish the portfolio website, I organized the static assets and deployed them using an Nginx container managed by Docker Compose.

**Deployment Steps:**
1. **Directory Setup:** Place your portfolio HTML, CSS, and JS files in: `/home/username/portfolio`
2. **Docker Configuration:** Create a `docker-compose.yml` file with the following stack:
   ```yaml
   version: "3.8"
   services:
     portfolio:
       image: nginx:latest
       container_name: portfolio
       volumes:
         - /home/username/portfolio:/usr/share/nginx/html:ro
       ports:
         - "8080:80"
       restart: unless-stopped
   ```
3. **Execution:** Launch the service using:
   ```bash
   docker compose up -d
   ```

**Accessing the Site:**
Once deployed, the portfolio is available locally on port 8080 or remotely through your secure Tailscale address: `http://100.x.x.x:8080`

🧭 11. Final Architecture Diagram
```Mermaid
                ┌──────────────────────────┐
                │      Dell OptiPlex       │
                │   Ubuntu Server 22.04    │
                └─────────────┬────────────┘
                              │
                     ┌────────▼────────┐
                     │     Docker       │
                     └────────┬────────┘
                              │
        ┌─────────────────────┼──────────────────────┐
        │                     │                      │
┌───────▼───────┐   ┌────────▼────────┐   ┌─────────▼─────────┐
│   Portainer   │   │   MySQL 5.7     │   │   Nginx Portfolio   │
│   (9000)      │   │   (3306)        │   │   (8080)            │
└───────────────┘   └─────────────────┘   └─────────────────────┘
                              │
                              │
                     ┌────────▼────────┐
                     │   Tailscale      │
                     │   (100.x.x.x)    │
                     └────────┬────────┘
                              │
        ┌─────────────────────┼──────────────────────┐
        │                     │                      │
┌───────▼────────┐   ┌────────▼────────┐   ┌─────────▼─────────┐
│  Laptop (LAN)  │   │  Phone (TS)     │   │  Teammates (SSH)   │
└────────────────┘   └─────────────────┘   └─────────────────────┘
```

📸12.  Some images
Portainer Dashboard with container names
<img width="1468" height="832" alt="Screenshot 2026-10-02 at 9 34 43 PM" src="https://github.com/user-attachments/assets/05792b72-4231-4623-beb4-29ca631f65e3" />
<img width="672" height="477" alt="Screenshot 2026-10-02 at 9 35 37 PM" src="https://github.com/user-attachments/assets/1a6a5e1d-2f0d-4aaf-a097-c9d9e03eb054" />


---
🔮 13. Future Roadmap
As the Homelab continues to evolve, the following improvements and services are planned to increase functionality, security, and data resiliency:

*   **Reverse Proxy Migration:** Transition from direct port access to Nginx Proxy Manager or Traefik for SSL/TLS management (HTTPS).
*   **Automated Backups:** Implement Restic or BorgBackup to sync container volumes and databases to off-site cloud storage.
*   **Monitoring Stack:** Deploy Grafana and Prometheus to monitor server resource usage and container health in real-time.
*   **Media Management:** Integrate Jellyfin or Plex to host personal media libraries.
*   **Enhanced Security:** Set up Fail2Ban to protect SSH and implement UFW firewall hardening.
*   **CI/CD Pipeline:** Integrate GitHub Actions to automate the deployment of new application updates directly to the server.
---
