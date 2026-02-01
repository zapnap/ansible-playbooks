# Nap's Ansible Playbooks

Ansible playbooks (Linode etc).

## Requirements

- Ansible 2.14+
- Python 3.8+
- Linode API token ([create one here](https://cloud.linode.com/profile/tokens))
- SSH key pair

## Quick Start

1. **Install dependencies:**
   ```bash
   # Find Ansible's Python path
   ansible --version | grep python

   # Install Python deps into that Python (replace path as needed)
   /path/to/ansible/python -m pip install linode_api4 ansible_specdoc

   # Install Ansible collections
   ansible-galaxy collection install -r requirements.yml
   ```

2. **Configure variables:**
   ```bash
   cp group_vars/vars.example.yml group_vars/vars.yml
   # Edit vars.yml with your values
   ```

3. **Create and configure a server:**
   ```bash
   ansible-playbook create_linode.yml
   ```

4. **Destroy when done:**
   ```bash
   ansible-playbook destroy_linode.yml
   ```

## Configuration

Edit `group_vars/vars.yml`:

| Variable | Description |
|----------|-------------|
| `linode_token` | Your Linode API token |
| `linode_label` | Name for the server |
| `linode_type` | Instance size (e.g., `g6-standard-2`) |
| `linode_region` | Datacenter (e.g., `us-east`) |
| `linode_image` | OS image (e.g., `linode/ubuntu22.04`) |
| `password` | Root password (used during creation) |
| `ssh_key` | Your public SSH key (contents of `~/.ssh/id_rsa.pub`) |

### Encrypting Secrets

Use ansible-vault to encrypt sensitive values (`password`, `linode_token`):

```bash
# Encrypt a value
ansible-vault encrypt_string 'your-secret-value' --name 'linode_token'

# Paste the output into vars.yml, then run playbooks with:
ansible-playbook create_linode.yml --ask-vault-pass

# Or use a password file:
echo 'your-vault-password' > ~/.vault_pass && chmod 600 ~/.vault_pass
ansible-playbook create_linode.yml --vault-password-file ~/.vault_pass
```

## What Gets Configured

The `node_config` role sets up:

### Base Setup
- `ubuntu` user with passwordless sudo
- SSH key authentication
- Essential packages (git, vim, curl, build-essential)
- Go 1.20

### Security Hardening
- **SSH hardening** - Disables password auth, root login, limits auth attempts
- **fail2ban** - Bans IPs after 3 failed SSH attempts (24h ban)
- **unattended-upgrades** - Automatic daily security patches
- **Swap file** - 2GB swap to prevent OOM crashes

## Post-Setup Access

After the playbook runs, root SSH login is disabled. Connect as:

```bash
ssh ubuntu@<your-server-ip>
```

The `ubuntu` user has passwordless sudo:
```bash
sudo apt update  # works without password
```

## Customization

### Change swap size

```bash
ansible-playbook create_linode.yml -e swap_size=4G
```

## File Structure

```
├── ansible.cfg
├── create_linode.yml      # Provision + configure
├── destroy_linode.yml     # Tear down
├── hosts.ini              # Inventory (auto-populated)
├── requirements.yml       # Ansible collections
├── group_vars/
│   └── vars.yml           # Your configuration
└── roles/
    ├── node_create/       # Linode provisioning
    ├── node_config/       # Server configuration + hardening
    └── node_destroy/      # Linode teardown
```

## References

- [Linode Ansible Collection Guide](https://www.linode.com/docs/guides/deploy-linodes-using-linode-ansible-collection/)
- [Linode Instance Types](https://www.linode.com/docs/products/compute/compute-instances/plans/choosing-a-plan/)
- [Linode Regions](https://www.linode.com/docs/products/platform/get-started/guides/choose-a-data-center/)
