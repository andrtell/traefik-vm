# Traefik VM
VM with Traefik and Podman.

## SSH Setup

Add the IP of your VM to `/etc/hosts`.

```
0.0.0.0 vm01
```

Create SSH-keys for the `agent` user.

```
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519-agent -C agent
```

Update your `~/.ssh/config`.

```
Match user root host vm01
    IdentityFile ~/.ssh/id_ed25519-root

Match user agent host vm01
    IdentityFile ~/.ssh/id_ed25519-agent
```

Add your keys to the SSH-agent.

```
ssh-add ~/.ssh/id_ed22519-root
```
```
ssh-add ~/.ssh/id_ed22519-agent
```

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

Run the playbook `playbooks/01_user.yaml`. This will create the `agent` user on the VM.

```
ansible-playbook -i inventory.yaml --extra-vars "agent_password=$(mkpasswd --method=sha-512)" playbooks/01_user.yaml
```

Run all other playbooks.

```
ansible-playbook -i inventory.yaml --ask-become-pass --extra-vars "@vars.yaml" playbooks/0[2-5]*.yaml
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
