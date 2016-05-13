# ansible-baseline
Baseline webserver configuration via Ansible

This is an Ansible playbook that will take a bare, all-defaults RHEL/CentOS 7 box and set it up as a basic, somewhat more secure Apache webserver (not an original concept, but it's tailored to my current needs). Note that it uses features specific to RHEL 7, so at the moment it won't work on any other distros.

The playbook performs the following tasks on the remote host:

- **Adds an administrative account** with NOPASSWD sudo privleges (suitable for use as an Ansible deployment user), as defined in vault.yml. By default this play will use an SSH public key from the user running the playbook for the remote admin account. 
- Set a new root password per vault.yml.
- **Installs and enables NTP** using the timezone defined in vault.yml. Alternatively you could change it to use chronyd instead of ntpd.
- Updates all packages and installs packages set in the required_packages variable.
- Ensures **SELinux** package requirements are installed, enables it with the targeted policy, and forces a context relabel at next bootup (this is due to my VPS provider disabling SELinux by default).
- Disables **SSH** root login, disables SSH password login, and ensures SSH is enabled in the firewalld configuration.
- **Rate-limits SSH connections** to 3 attempts every 30 seconds per IP using firewalld direct rules. This is meant to limit intrusion attempts (rather than using something heavier like fail2ban).
- **Installs and enables Apache** (but does no further configuration). Enables http and https in the firewalld configuration.
- And ultimately reboots the system.

## Notes

The group_vars/all/vault.yml file is an Ansible vault -- the copy in this repo can be decrypted and edited with the password 'default' as described [here](https://docs.ansible.com/ansible/playbooks_vault.html "Vault Doc").

It's assumed that you have root SSH password access to the remote hosts, but *please note* you will no longer have root SSH access at all after running this playbook, and only key-based access for other users. The idea here is that afterward you'll be orchestrating everything through the Ansible admin user that's created.

The included ansible.cfg file will cause an automatic prompt for the SSH and vault passwords -- the presence of this file in the playbook directory will override any other config files on your system so you may want to remove it.

This playbook is meant to follow Ansible [best practices](https://docs.ansible.com/ansible/playbooks_best_practices.html "Best Practices Doc") as much as possible.