# arcane-remote

Arcane Edge agent for a Docker host managed from the central Arcane manager
([tkightly/arcane](https://github.com/tkightly/arcane) on docker-mgmt). The
agent dials out to the manager on port 9553, so the host needs no inbound
port.

## Deploying

1. In Arcane: **Environments → Add → Edge**, named after the host. Copy the
   agent token.
2. On the host:

   ```sh
   git clone https://github.com/tkightly/arcane-remote.git /opt/arcane-agent
   cd /opt/arcane-agent
   cp .env.example .env    # set AGENT_TOKEN
   chmod 600 .env
   docker compose up -d
   ```

The environment shows as Online once the agent connects.

## Projects directory

Git-synced projects are checked out on the host at `PROJECTS_PATH` (default
`/opt/stacks`), mounted at `/app/data/projects` in the agent. Arcane maps
projects' relative bind mounts through this mount, so the container path
must stay `/app/data/projects`. The agent runs as uid 1000 (`PUID`), which
must own the directory.

## No socket proxy

The agent mounts the Docker socket directly. It has to be able to create
containers, and that alone is full control of the host (a privileged
container with `/` mounted), so a socket proxy only blocks extras such as
`exec`. Earlier versions of this repo used one; it broke image builds with
HAProxy-based proxies, needed an unreleased build of
`wollomatic/socket-proxy`, and broke deploys whenever the host's `docker`
group id changed.

## Updating

Keep the agent on the same version as the manager. Arcane v3 renames the
environment variables to `ARCANE_*`, so that upgrade needs the rename here
too.
