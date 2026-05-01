# Backup

Back up the persistent Docker volumes from the Zulip VM.

At minimum, keep:

- `database`
- `zulip`

Depending on how you use the instance, also keep:

- `redis`
- `rabbitmq`

The Compose stack stores the Zulip application data, uploads, and database state in named Docker volumes, so a VM snapshot alone is not enough unless it includes those volumes.

