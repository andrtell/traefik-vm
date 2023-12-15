# Traefik VM

VM with Traefik and Podman.

## Basic VM

Setup you VM using [basic-vm](https://github.com/andrtell/basic-vm).

## Ansible

Create the file `inventory.yaml`.

```
ungrouped:
  hosts:
    vm:
      ansible_host: vm01
```

Create the file `vars.yaml`.

```
force_https: false

lets_encrypt_email: acme@example.com

cert_from_file: true
cert_fullchain: /etc/letsencrypt/live/example.com/fullchain.pem
cert_privkey: /etc/letsencrypt/live/example.com/privkey.pem

dashboard_domain: 'traefik.example.com'
```

## Traefik VM

Run all playbooks.

```
ansible-playbook -i inventory.yaml --ask-become-pass --extra-vars "@vars.yaml" playbooks/*.yaml
```

## Podman

Add a new podman connection.

```
podman system connection add vm01 ssh://agent@vm01/run/user/1000/podman/podman.sock
```

Test the connection.

```
podman -r -c vm01 version
```

## Containers

Start a demo container.

```
podman -r -c vm01 run -d --rm --name httpd --network traefik \
    --label 'traefik.enable=true' \
    --label 'traefik.http.routers.myrouter.rule=PathPrefix(`/`)' \
    docker.io/httpd
```

Test it by visiting `http://vm01`
