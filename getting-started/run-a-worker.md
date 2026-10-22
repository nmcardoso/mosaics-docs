# Run a worker

This page is for collaborators who lend a machine to a MOSAICS download. A *worker* is a small program that makes archive requests on behalf of the main node, so the download is spread over more than one IP address and stays inside each archive’s per-IP rate limit (see [Archive rate limits](../concepts/archives.md)). You receive one command from the person who runs the main node; this page says what it does, what it sends and how to stop and remove it.

## What the command does

The command you received has the form below, where `NAME` is the name you choose for your worker (1 to 64 letters, digits or hyphens):

```console
$ curl -sL <the address you were given> | sh -s -- NAME
```

It needs no root rights and installs nothing outside two folders in your home directory. In order:

1. A short shell script checks the name and that the machine is Linux on x86_64, downloads a small launcher program (a static binary of about 2 MB) and checks its SHA-256 against the value written in the script. A launcher that does not match is not run.
2. The launcher finds `uv` on your `PATH`, or downloads a pinned release of it into `~/.mosaics/bin` after checking its SHA-256. It never pipes a downloaded installer into a shell.
3. The launcher downloads the worker’s encrypted payload, checks its SHA-256, decrypts it into a private temporary folder (mode 0700) and starts the worker with `uv`, which fetches Python 3.12 if the machine has none and the worker’s few dependencies from PyPI.
4. The worker makes its own SSH host key on the first run, reads the announcement of where the main node is, logs in to the institute’s SSH host and tells the main node its name and its [machine data](). From then on it answers heartbeats and carries the requests the main node sends it. The worker logs each step as it goes, including the SSH host and port it connects to.
5. When the worker ends, the launcher deletes the temporary folder.

A second run with the same name finds everything cached and starts in a few seconds. A new release of the worker is picked up when you run the command again.

## Requirements

- Linux on x86_64 (the script refuses anything else).
- `curl` or `wget`, and `sha256sum` or `shasum`.
- Outbound HTTPS, to download the launcher, the payload and (if needed) `uv`, Python and the worker’s dependencies, and to look up your public address.
- Outbound SSH, to the institute’s login host. The login host uses a non-standard port, which the worker prints when it starts, so a firewall that only allows port 22 out will block it.
- Nothing inbound: the worker dials out and opens no port on your machine.

## What is sent, and where it is kept

When it registers, the worker sends the main node the data below. Each item is best effort and is left empty when it cannot be read.

| Item          | What is sent                                                                                                                                                                                                                                                       |
|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Name and user | The name you chose, your short and fully qualified host name, and the login name of the user running the worker.                                                                                                                                                   |
| System        | Operating system, kernel release, distribution name, CPU architecture.                                                                                                                                                                                             |
| Hardware      | Number of logical CPUs, CPU model, installed memory in GB.                                                                                                                                                                                                         |
| Process       | Python version, worker release and build, protocol version, process id, start time (UTC), local time zone.                                                                                                                                                         |
| Addresses     | The addresses of your network interfaces and of your default route (these can be private addresses), and your **public IP address**. The worker looks the public address up with one request to a public service (`api.ipify.org`, else `1.1.1.1`) when it starts. |
| Machine id    | The first 16 hexadecimal characters of a SHA-256 hash of the machine id, never the id itself. It lets the main node tell machines apart.                                                                                                                           |
| Environment   | Whether the worker runs inside a container or WSL.                                                                                                                                                                                                                 |

Nothing else is read: no file contents, no environment variables, no browsing data and no list of your processes.

The main node keeps this data in two places, both private to the people who run it. A **ledger** file on the main node records one entry when you connect and one when you leave, with the data above and what the session carried (requests, bytes and the archives they went to). If the operator reports to Weights & Biases, a private project holds the same entries. The ledger and the report are used to credit each worker for what it carried and to tell machines apart. They are not published. If you want your data removed, ask the operator.

The requests themselves pass through your machine. Your public IP address appears in the logs of the archives that serve them, and your connection carries the downloaded bytes. HTTPS traffic stays encrypted between the main node’s pipeline and the archive; the worker only opens the connection. Ask the operator how much data to expect, since it depends on the run, and run the worker on a connection where that is acceptable.

## What the main node can ask of your machine

The main node can ask the worker only to open TCP connections to a host and port, and to answer heartbeats. It cannot start a program, read a file or open a shell on your machine. By default the worker accepts destination ports 80 and 443 and refuses every destination that is not on the public internet: loopback, private (for example 10.x, 172.16 to 172.31 and 192.168.x), link-local, shared (carrier-grade NAT) and IPv6 unique-local addresses. A host name is looked up on your machine and refused when any of its addresses is one of those, so a web service on your own network, for example a router or a department server, cannot be reached through the worker. Inside the public internet the worker still opens a connection to any host on those ports. If you want to limit it to the archives it should reach, add options after the name; they are passed through:

```console
$ curl -sL <the address you were given> | sh -s -- NAME --allow-origin '*.cds.unistra.fr' --allow-origin 'irsa.ipac.caltech.edu' --allow-port 443
```

`--allow-origin` takes a hostname pattern and can be repeated; `--allow-port` can be repeated too. A request outside the list is refused and the refusal appears in the worker’s log. `--allow-private` lifts the refusal of non-public addresses; use it only for an archive mirror on a private network.

The worker logs in to the institute’s SSH host with a login that belongs to the pool and is part of the release. It is used for forwarding the tunnel and is not a login to your machine. Whether several machines may share that login is a question for the institute that hosts it; the operator checks it.

## Stop it

- **Ctrl-C** in the terminal closes the tunnel at once. A request that had not yet received its first byte is placed on another worker by the main node; a download in progress is retried by the pipeline.
- **`SIGTERM`** to the launcher (for example `kill -TERM <pid>`, or `kill -TERM -<pgid>` for the whole process group) **drains** the worker: it stops accepting new requests, waits for the live ones to finish (up to 300 seconds), then leaves. A second `SIGTERM` closes it at once.
- A worker that was told by the main node to drain also leaves on its own.

The main node notices a dead worker within a few heartbeats (about 30 seconds) and stops sending it requests.

## Where files go, and how to remove them

| Where                          | What                                                                                                                                         |
|--------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `~/.cache/mosaics-worker/bin/` | The launcher, named by its SHA-256.                                                                                                          |
| `~/.cache/mosaics-worker/`     | The encrypted payload, named by its SHA-256.                                                                                                 |
| `~/.mosaics/proxy/`            | The worker’s SSH host key, `NAME_worker_host_key` and its public half. A worker that restarts under the same name keeps its identity.        |
| `~/.mosaics/bin/`              | `uv`, only if the machine had none.                                                                                                          |
| `$TMPDIR/mosaics-worker-*`     | The decrypted payload while the worker runs. Deleted at exit; a folder left by a crash is removed by the next launch once it is an hour old. |
| `uv`’s own folders             | The Python 3.12 that `uv` installs when the machine has none. This is `uv`’s normal location and you may keep it.                            |

To remove everything the worker made:

```console
$ rm -rf ~/.cache/mosaics-worker ~/.mosaics/proxy ~/.mosaics/bin
```

If you also use MOSAICS on the machine, keep the rest of `~/.mosaics`.

## Keep it running

A worker ends when its terminal closes. Two ways to keep it running:

**`tmux` (or `screen`).** The simplest. Start the worker inside a session, detach with `Ctrl-b d`, and come back with `tmux attach`:

```console
$ tmux new -s worker
$ curl -sL <the address you were given> | sh -s -- NAME
```

**systemd, as a user service.** Save the script once, and let the service fetch it fresh at every start, because a saved script names one release of the launcher and the site keeps only the current and the previous release. Create `~/.config/systemd/user/mosaics-worker.service` with the lines below, replacing `NAME` and the address:

```ini
[Unit]
Description=MOSAICS proxy worker

[Service]
ExecStartPre=/bin/sh -c 'curl -fsSL "<the address you were given>" -o "%h/.cache/mosaics-worker.sh"'
ExecStart=/bin/sh %h/.cache/mosaics-worker.sh NAME
KillMode=mixed
TimeoutStopSec=330
Restart=on-failure
RestartSec=60

[Install]
WantedBy=default.target
```

Then run `systemctl --user daemon-reload`, `systemctl --user enable --now mosaics-worker` and, so that it survives your logout, `loginctl enable-linger $USER`.

`KillMode=mixed` matters. The launcher puts the worker in its own process group and relays the signals it gets. systemd’s default, `control-group`, sends `SIGTERM` to every process of the service at once, so the worker would receive it twice, once directly and once from the launcher, and the second `SIGTERM` closes it without draining. With `KillMode=mixed`, systemd signals only the launcher, which drains the worker; whatever is left after `TimeoutStopSec` (longer than the 300-second drain) is killed. `systemctl --user stop mosaics-worker` therefore drains.

## For the operator: see who is connected

The main node serves a live list of its workers only when it is started with `--stats-port PORT`; the endpoint is off by default and bound to the proxy’s own address. `GET /stats` then shows each worker’s name, host name, public address, current requests and totals per archive, and `POST /drain?worker=ID` drains one. After the fact, `mosaics-proxy workers` prints the contribution of each name from the ledger. The commands and options are in the [`mosaics-proxy` reference](../cli/proxy.md).
