# easy-ansible-n8n

Automated deployment of n8n on Ubuntu servers using Ansible, Docker, and Nginx with automatic SSL certificate management via Certbot.

## Features

- **Automated Installation**: Installs Docker, Docker Compose, Nginx, Certbot, and n8n with a single command.
- **SSL/TLS Security**: Automatically obtains and renews Let's Encrypt SSL certificates.
- **Reverse Proxy**: Configures Nginx as a secure reverse proxy for n8n.
- **Customization**: Easily configurable ports, domain names, and n8n settings via Ansible variables.
- **Storage Options**: Supports local storage or PostgreSQL (configurable).

## Prerequisites

Before running this playbook, ensure you have the following:

1.  **Ubuntu Server**: A fresh installation of Ubuntu 20.04 LTS or 22.04 LTS.
2.  **Domain Name**: A valid domain name (e.g., `n8n.example.com`) pointing to your server's public IP address.
3.  **SSH Access**: Root or sudo user access to the server via SSH.
4.  **Control Node**: A computer (your local machine or another server) with:
    -   **Python 3**: Installed.
    -   **Ansible**: Installed (`pip install ansible`).

## Network Requirements

Ensure the following ports are open on your server's firewall (and security groups if using a cloud provider like AWS, Azure, GCP):

-   **Port 22 (SSH)**: Required for Ansible connection.
-   **Port 80 (HTTP)**: Required for Let's Encrypt SSL certificate validation.
-   **Port 443 (HTTPS)**: Default secure port (or your custom HTTPS port defined in `group_vars/all.yml`).

## Installation & Usage

### 1. Install Ansible (on your control node)

If you haven't installed Ansible yet, you can do so using pip:

```bash
python3 -m pip install --user ansible
```

### 2. Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/velocitystar/n8n-ansible.git
cd n8n-ansible/easy-ansible-n8n
```

### 3. Configure Inventory

Edit the `inventory.ini` file to include your server's IP address and connection details.

1.  Open `inventory.ini`.
2.  Replace existing entries or add your server IP under `[n8n_servers]`.
3.  Set the `ansible_user` to the username you use to SSH into the server.

**Example `inventory.ini`:**

```ini
[n8n_servers]
# Replace 192.0.2.10 with your server's IP
192.0.2.10 ansible_user=ubuntu
```

### 4. Configure Variables

Unlock the power of customization by editing `group_vars/all.yml`.

**Key Variables:**

-   `domain_name`: **(Required)** Your n8n domain (e.g., `n8n.example.com`).
-   `letsencrypt_email`: **(Required)** Email for Let's Encrypt expiration notices.
-   `nginx_https_port`: The port you want to access n8n on (default: `443` or `4445` if customized).
-   `n8n_version`: The Docker tag for n8n (default: `latest`).
-   `n8n_storage_type`: `local` (SQLite) or `postgres` (PostgreSQL container).

**Example `group_vars/all.yml`:**

```yaml
# User configuration
domain_name: "n8n.example.com"
letsencrypt_email: "admin@example.com"
nginx_https_port: 4445

# n8n configuration
n8n_version: "latest"
```

### 5. Run the Playbook

Execute the playbook to start the deployment:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

_Note: If you need to enter a sudo password for the remote user, add the `--ask-become-pass` flag:_

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

## What Happens Next?

The playbook will performs the following steps:

1.  **Install Dependencies**: Docker, Docker Compose, Nginx, Certbot.
2.  **Configure Nginx (Stage 1)**: Sets up Nginx on Port 80 for Let's Encrypt validation.
3.  **Obtain Certificate**: Runs Certbot to get a genuine SSL certificate.
4.  **Configure Nginx (Stage 2)**: Reconfigures Nginx to proxy traffic to n8n over HTTPS (using your configured port).
5.  **Start n8n**: Launches the n8n Docker container.

Once the playbook completes, you can access your n8n instance at:

`https://<your-domain>:<nginx_https_port>/`

Example: `https://n8n.example.com:4445/`

## Troubleshooting

-   **Playbook fails at SSH**: Ensure you can SSH into the server manually (`ssh user@server_ip`) before running Ansible.
-   **Certbot fails**: Ensure your domain name DNS A record points to the server IP and Port 80 is open.
-   **Nginx fails to start**: Check if another service is using Port 80 or your custom HTTPS port.
