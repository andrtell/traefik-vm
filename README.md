# Traefik VM

Following the instructions in this repository you will configure a VM running Ubuntu 22.04 LTS

Such that:

- Traefik will be running on the VM.
- Certificates are automatically created and renewed by Treafik.
- Network traffic on port 80 (http), 443 (https), 8080 (Treafik) and 8443 (Treafik) are allowed.
- Traffic on port 80 is routed to port 8080, and traffic on port 443 is routed to port 8443.

Before you start, you must first complete the setup in [Basic VM](https://github.com/andrtell/basic-vm). 

**Warning**

If you are updating existing DNS records, please wait a while (make note of TTL) before running Traefik. It is very easy to hit the rate limit over at Lets Encrypt if cached DNS records are not up to date.

## Local setup

### Ansible

Ansible needs to know about your remote machine.

*Before you continue*

Create the file `inventory.yaml` in the root folder of this repo.

```
ungrouped:
  hosts:
    vm:
      ansible_host: <DOMAIN>
```

## Remote setup

This step will be run as `agent` on the VM.

Ansible will prompt you for the password you provided in [Basic VM](https://github.com/andrtell/basic-vm) step 1.

The email you provide will be used by Lets encrypt.

*Before you continue*

Run all the playbooks.

```
ansible-playbook -i inventory.yaml --ask-become-pass --extra-vars "lets_encrypt_email=<EMAIL>" playbooks/*.yaml
```

*Then confirm Traefik works on your VM*

Deploy a container on your VM.

See [Basic VM](https://github.com/andrtell/basic-vm) step 2 for how to setup a remote Podman connection.

```
podman -r -c vm run \
  -d --rm --name httpd --network traefik \
  --label 'traefik.enable=true' \
  --label 'traefik.http.routers.httpd.rule=Host(`<DOMAIN>`)' \
  --label 'traefik.http.routers.httpd.entrypoints=websecure' \
  --label 'traefik.http.routers.httpd.tls=true' \
  --label 'traefik.http.routers.httpd.tls.certresolver=letsencrypt' \
  docker.io/httpd
```

Visit `https://<DOMAIN>`.

To stop the server.

```
podman -r -c vm stop httpd
```

**OK, all done!**
