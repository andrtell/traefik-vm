# Traefik VM
VM with Traefik and podman.

## Before you start

Create the file `inventory.yaml`.

```
ungrouped:
  hosts:
    vm:
      ansible_host: <HOST>
```

Create the file `vars.yaml`.

```
lets_encrypt_email: myemail@example.com
force_https: false
dashboard_domain: ''
```

Create SSH key for the `agent` user.

```
$ ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519-agent -C agent
```

## Setup VM

Run the playbook `playbooks/01_user.yaml`.

```
$ ssh-add ~/.ssh/id_ed25519-root
```

```
$ ansible-playbook \
    -i inventory.yaml \
    --private-key ~/.ssh/id_ed25519-root \
    --extra-vars "agent_password=$(mkpasswd --method=sha-512)"
    playbooks/01_user.yaml
```

Run all other playbooks.

```
$ ssh-add ~/.ssh/id_ed25519-agent
```

```
$ ansible-playbook \
    -i inventory.yaml \
    --private-key ~/.ssh/id_ed25519-agent \
    --ask-become-pass \
    --extra-vars "@vars.yaml" \
    playbooks/0[2-5]*.yaml
```

## Podman

Add a new podman connection.

```
$ podman system connection add --identity=$HOME/.ssh/id_ed25519-agent vm ssh://agent@<HOST>/run/user/1000/podman/podman.sock
```

Test the connection.

```
$ podman -r -c vm version
```

Podman does not seem to play well with the ssh-agent.

Update your `~/.ssh/config` file.

```
Match host <HOST> user agent
    IdentityFile ~/.ssh/id_ed25519-agent
```

Add a new podman connection.

```
$ podman system connection add vm ssh://agent@0.0.0.0/run/user/1000/podman/podman.sock
```

## Containers

Start a demo container.

```
podman -r -c vm run -d --rm --name httpd --network traefik \
    --label 'traefik.enable=true' \
    --label 'traefik.http.routers.myrouter.rule=PathPrefix(`/`)' \
    docker.io/httpd
```

Test it by visiting `http://<HOST>:80`
