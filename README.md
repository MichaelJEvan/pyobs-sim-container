# pyobs 2.0 simulator

A containerised pyobs observatory — one simulated telescope, one simulated
camera, and the XMPP server they talk through. It exists so software that
drives pyobs can be developed and tested without an observatory, and it starts
in one command.

Built for the [Stellarium ↔ pyobs bridge](https://github.com/MichaelJEvan/pyobs-stellarium-bridge),
but there is nothing bridge-specific in it. Anything that speaks pyobs can
point at it.

## Running it

```bash
docker compose up -d
```

That is the whole setup. Two containers come up, five XMPP accounts get
registered, and a `DummyRaDecTelescope` sits at Polaris waiting to be told
where to go.

```bash
docker compose logs -f sim     # watch it
docker compose down            # stop and remove both containers
```

XMPP is on **localhost:5222**. Point your client at it.

| account | for |
|---|---|
| `telescope` | the simulated mount |
| `camera` | the simulated camera |
| `stellarium` | free, for a client |
| `scratch` | free, for a client |
| `console` | free, for a client |

They all share one password: `pyobs`. These are localhost-only accounts in a
throwaway container, and the password is the one pyobs uses in its own
`tests/xmpp/` config. It is not a secret and is not meant to be one.

**One XMPP account per program.** Two clients sharing a JID kick each other off
in a loop, which looks like a hang rather than an auth error. That is why there
are three spare accounts rather than one.

## Your own site

`work/_environment.yaml` sets where the observatory is, which is what decides
what is above the horizon. On first start the container copies
`work/_environment.example.yaml` into place, so you get McDonald Observatory in
Texas — home of MONET/North, one of the telescopes pyobs really runs.

Edit `work/_environment.yaml` for your own location. It is **gitignored**, so
your coordinates stay on your machine.

## Changing the pyobs version

One line in `sim/Dockerfile`:

```
ARG PYOBS_VERSION=2.0.1
```

Then `docker compose build sim && docker compose up -d sim`. Pin a version
rather than tracking latest — pyobs 2.0 is still an actively developing line,
and a client has to be on a version that talks to the module.

## What's here

| file | what it does |
|---|---|
| `docker-compose.yml` | the two services and how they wait for each other |
| `sim/Dockerfile` | Python 3.11 plus a pinned pyobs-core |
| `ejabberd/ejabberd.yml` | the XMPP server config, from pyobs's own `tests/xmpp/` |
| `work/telcam.yaml` | the telescope and camera modules |
| `work/_environment.example.yaml` | the default observing site |

## Things that cost an afternoon

Written down because none of them produce a useful error message.

- **`CTL_ON_START`, not `CTL_ON_CREATE`.** The ejabberd image ships its
  database directory, so the first-run check is always false and
  `CTL_ON_CREATE` never fires — silently, with no accounts created. Commands
  are separated by ` ; `, **not** newlines; as a multi-line YAML block it does
  nothing at all. The `!` prefix makes a failure logged and ignored, which
  matters because `register` conflicts on every restart.
- **`ghcr.io/processone/ejabberd`, not `ejabberd/ecs`.** The latter is
  amd64-only. Under emulation on Apple Silicon `ejabberdctl` segfaults, which
  is what makes account creation fail without saying so.
- **The shared roster group is required, not cosmetic.** `XmppComm` never sends
  subscription requests, so without `srg_user_add @all@` the modules connect
  fine and cannot see each other.
- **`start_period: 10s` on the healthcheck.** Run `ejabberdctl` sooner and it
  collides with the control socket ejabberd is still creating, and the node
  dies on startup.
- **`depends_on: condition: service_healthy`.** A plain dependency lets pyobs
  start against a server that is not listening yet, and its reconnect attempts
  pile up from there.
- **A restart resets the mount to `telcam.yaml`'s `position:`** (Polaris). A
  slew that "arrives instantly" after a restart is that, not a bug.

## Related repos

- [pyobs-stellarium-bridge](https://github.com/MichaelJEvan/pyobs-stellarium-bridge):
  puts any pyobs telescope on a Stellarium sky chart.
- [pyobs-indi](https://github.com/MichaelJEvan/pyobs-indi):
  drives INDI mounts from pyobs. This container is the easiest way to try
  both without hardware.

## License

MIT.
