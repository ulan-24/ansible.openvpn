# Overview
Ansible Playbook for deploying OpenVPN on Ubuntu and Arch Hosts with LDAP authentication support.

### Requirements
- Ansible >= 2.4

### Roles (in order of Execution)
- **openvpn**
  - Installs, configures and hardens OpenVPN security configuration
  - Enables `tls-crypt` (supersedes tls-auth)
  - Enables forward secrecy TLS ciphers
  - Reduces OpenVPN daemon privileges after initialization
  - Configures LDAP auth over TLS
  - Pushes OpenDNS for client configuration

- **easy-rsa**
  - Certificate Authority and passphrase (optionally transfers CA cert to alt/backup VPN hosts)
  - OpenVPN server certificate and private key
  - Diffie-Hellman (DH) parameters key
  - Hash-based Message Authentication Code (HMAC) key

- **network** (default: UFW or iptables)
  - Enables NAT forwarding
  - Port forward OpenVPN (default proto: tcp)
  - Starts OpenVPN network/service

- **client**
  - Generates OpenVPN client `.ovpn` profile using a template
  - Sends an email to the OpenVPN user with the client profile for desktop/mobile import

### Tags (in order of Execution)
  - `openvpn`, `vpnconfig`, `ldap`
  - `rsa`
  - `network`
  - `client`, `email`

### Execution
- Update the global variables in `group_vars/all` for your own VPN server environment.
- Update the host_vars variable file for each OpenVPN host with your host-specific environment.
- **Recommended**: Use vpn virtualenv `mkvirtualenv mailstack; pip install -r requirements.txt`
- Execute the playbook

  `ansible-playbook [options] vpn.yaml`

- Execute specifc role(s)
  
  `ansible-playbook [options] --tags "vpn_config" vpn.yaml`

- Exclude a role

  `ansible-playbook [options] --skip-tags "client"`

### LDAP
LDAP auth is enabled by default. The OpenVPN client file `openvpn-client.ovpn` can be imported into a mobile/desktop app which will prompt for the VPN user's LDAP credentials. An OpenVPN LDAP schema can be found in `roles/openvpn/files/ldap/openvpn-ldap.schema`. 

### Client
By default this Playbook uses LDAP auth over certificate based authentication. While using LDAP auth the client `openvpn-client.ovpn` file only requires the CA cert and tls-crypt key. One nice advantage of using LDAP auth with OpenVPN is that creating unique client configs is not necessary. The same client config can be distributed to VPN users.

###### Notes
- To have the EasyRSA role generate a new PKI, Certificate Authority (CA) and Server certs/keys, the `pki` directory within `/etc/easyrsa/` must not be present. The role will not overwrite an existing PKI and related files.

- Currently works on remote hosts running os-families:
  - ArchLinux: Arch Linux
  - Debian: Ubuntu

### TODO
- MFA/YubiKey support
- Email client config to VPN user
- Handle more OS families
- Handle `update-systemd-resolved` for unix clients
