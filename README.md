# Overview
Ansible Playbook for deploying OpenVPN on Ubuntu and Arch Hosts with LDAP authentication support.

### Requirements
- Ansible <= 2.6

### Roles (in order of Execution)
- **openvpn**
  - Installs, configures and hardens OpenVPN security configuration
  - Enables `tls-crypt` (supersedes tls-auth)
  - Enables forward secrecy TLS ciphers
  - Reduces OpenVPN daemon privileges after initialization
  - Configures LDAP auth over TLS
  - Pushes OpenDNS for client configuration

- **easy-rsa** (default expire: 10 years)
  - Certificate Authority and random generated passphrase
  - OpenVPN server certificate and private key
  - Diffie-Hellman (DH) parameters key
  - Hash-based Message Authentication Code (HMAC) key
  - **Optional**: transfer CA cert and HMAC key to other secondary/backup OpenVPN host(s)

- **network** (default: `ufw` or `iptables`)
  - Enables NAT forwarding
  - Port forward OpenVPN (default proto: `tcp`)
  - Starts OpenVPN network/service

- **client**
  - Generates OpenVPN client `.ovpn` profile from template
  - Sends an email to the OpenVPN user with the client profile for desktop/mobile import
  - Retrieves OpenDNS config pushed from OpenVPN server

### Tags (in order of Execution)
  - `openvpn`, `vpnconfig`, `ldap`
  - `rsa`
  - `network`
  - `client`, `email`

### Execution
- Update the global variables in `group_vars/all` for your own VPN server environment.
- Update the `host_vars` variable file for each OpenVPN server with your host-specific environment.
- **Recommended**: Set up vpn virtualenv `mkvirtualenv vpn; pip install -r requirements.txt`
- Execute the playbook

  `ansible-playbook [options] vpn.yaml`

- Execute specifc role(s)
  
  `ansible-playbook [options] --tags "openvpn,rsa" vpn.yaml`

- Execute sending client profile

  `ansible-playbook [options] --tags "client,email" vpn.yaml`

- Exclude a role

  `ansible-playbook [options] --skip-tags "client"` vpn.yaml

### LDAP
LDAP auth is enabled by default. The OpenVPN client file `openvpn-client.ovpn` can be imported into a mobile/desktop app which will prompt for the VPN user's LDAP credentials. An OpenVPN LDAP schema can be found in `roles/openvpn/files/ldap/openvpn-ldap.schema`. 

### Client
By default this Playbook uses LDAP auth rather than certificate based authentication. While using LDAP auth the client `openvpn-client.ovpn` file only requires the CA cert and tls-crypt key. One nice advantage of using LDAP auth with OpenVPN is that creating unique client configs is not necessary. The same client config can be distributed to VPN users.

### Email
The OpenVPN client config profile can be emailed to the VPN user. The client profile is generated using a template so the appropriate configuration and any number of VPN servers can be imported by desktop and mobile apps.

###### Notes
- To have the EasyRSA role generate a new PKI, Certificate Authority (CA) and Server certs/keys, the `pki` directory within `/etc/easyrsa/` must not be present. The role will not overwrite an existing PKI and related files.

- Currently works on remote hosts running os-families:
  - ArchLinux: Arch Linux
  - Debian: Ubuntu

### TODO
- MFA/YubiKey support
- Handle more OS families
