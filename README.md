# Zulip Deploy

Deploys a Zulip server to a dedicated VM through GitHub Actions and SSH.

This repo is the Zulip equivalent of `projects/Lemmy-deploy`:

- GitHub Actions prepares a `.env` file from repository secrets
- the workflow uploads the compose files to the target VM
- the remote host runs `docker compose pull` and `docker compose up`
- the public reverse proxy stays in `infra`, so this repo does not manage Caddy or TLS for the public domain
- the Zulip VM should be dedicated, because the stack publishes the usual web/mail ports

## Layout

- `docker-compose.yml`: Zulip runtime stack, based on the official Docker stack layout
- `.github/workflows/deploy.yml`: remote deployment workflow
- `.env.example`: local template for the values used by the workflow and Compose
- `BACKUP.md`: what needs to be backed up from the VM

## Required GitHub secrets

- `HOST`: VM hostname or IP
- `SSH_PRIVATE_KEY`: private key used by Actions to reach the VM
- `POSTGRES_PASSWORD`
- `MEMCACHED_PASSWORD`
- `RABBITMQ_DEFAULT_PASS`
- `REDIS_PASSWORD`
- `SECRET_KEY`
- `EMAIL_PASSWORD`

## Required GitHub variables

- `ZULIP_HOSTNAME`: public Zulip hostname, for example `chat.example.com`
- `ZULIP_ADMINISTRATOR`: contact email shown by Zulip
- `LOADBALANCER_IPS`: IP or CIDR for the proxy in front of Zulip
- `EMAIL_HOST`
- `EMAIL_HOST_USER`
- `EMAIL_PORT`
- `EMAIL_USE_SSL`
- `EMAIL_USE_TLS`

## Remote host assumptions

- SSH port `2222`
- SSH user `root`
- deploy path `/opt/zulip`

If you want to change those, update the workflow env block.

## First boot

The remote stack is expected to be bootstrapped with the same workflow. If you want to validate the server manually, the Zulip container can generate an organization creation link after the stack is up:

```bash
docker compose exec -u zulip zulip /home/zulip/deployments/current/manage.py generate_realm_creation_link
```

## Reverse proxy

`infra` should proxy the public Zulip hostname to this VM. For a separate VM, configure Caddy to send the usual forwarded headers and trust the VM as a load balancer via `LOADBALANCER_IPS`.
