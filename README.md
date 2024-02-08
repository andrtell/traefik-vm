# Traefik VM

Use [Ansible](https://docs.ansible.com/ansible/latest/index.html) to configure a remote machine running Ubuntu 22.04 LTS.

Such that:

- Traefik is installed and started as a SystemD user service.
- Certificates are automatically created and renewed by Treafik.
- Network traffic on port 80 (http), 443 (https), 8080 (Treafik) and 8443 (Treafik) are allowed.
- Traffic on port 80 is routed to port 8080, and traffic on port 443 is routed to port 8443.

This repo requires you to first setup you VM using [Basic VM](https://github.com/andrtell/basic-vm). 

## Domain

You need to register a domain name (e.g. `example.com`) and point it to your VM if you have not already. 

**Optional (but recommended)**

You could also add wildcard domain for your sub-domains (e.g `*.example.com`).

**Warning**

After updating your DNS records please wait a while (an hour or more) before running Traefik. It is very easy to hit the rate limit for Lets Encrypt if your DNS records are not up to date.

## Local setup

### Ansible

Ansible needs to know about your remote machine.

*Before you continue*

Create the file `inventory.yaml` in the root folder of this repo.

Make sure to replace `<YOUR-DOMAIN>` with your domain.

```
ungrouped:
  hosts:
    vm01:
      ansible_host: <YOUR-DOMAIN>
```

## Remote setup

This step will be run as `agent`.

You will be prompted for the `sudo` password you provided in the `Basic VM` setup.

This step will install Traefik and configure the firewall on the remote machine.

Make sure to replace `<YOUR-EMAIL>` with your email.

*Before you continue*

Run all playbooks.

```
ansible-playbook -i inventory.yaml --ask-become-pass --extra-vars "lets_encrypt_email=<YOUR-EMAIL>" playbooks/*.yaml
```
