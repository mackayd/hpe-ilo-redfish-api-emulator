# HPE iLO Redfish API Emulator

A community runtime package of the [HPE iLO Redfish emulator](https://github.com/HewlettPackard/ilo-redfish-emulator), with two server profiles and a Docker Compose setup for API learning, integration testing and VMware Aria Operations management pack development.

The emulator serves sample hardware inventory, health, firmware, storage, networking, power, thermal and log resources through Redfish. It includes the upstream session service and supported mock actions. Available resources differ between profiles.

This package builds on HPE's emulator; it is not an independently written implementation or an official HPE release. See [UPSTREAM.md](UPSTREAM.md) for its exact source revision and packaging changes.

## Requirements

- Docker Engine with Docker Compose v2, or Docker Desktop using Linux containers.
- Internet access for the first build to download the Python base image and dependencies.
- A free local TCP port: 8443 by default, plus 8444 for the optional second instance.

The runtime source and both mockup datasets are included. No Git submodules or external profile downloads are needed.

## Start one emulator

Open a terminal in this repository and run:

```sh
docker compose up --build -d
docker compose ps
```

The default profile is `DL380a_Gen12`. Its API service root is:

```text
https://localhost:8443/redfish/v1/
```

There is no iLO web management interface; this is a Redfish API emulator. A self-signed certificate is generated inside the container at startup.

Check the service root and server inventory:

```sh
curl -k https://localhost:8443/redfish/v1/
curl -k -u root:root_password https://localhost:8443/redfish/v1/Systems/
curl -k -u root:root_password https://localhost:8443/redfish/v1/Systems/1/
```

On Windows PowerShell, use `curl.exe` for these examples. The `-k` option accepts this emulator's self-signed lab certificate.

For session authentication, the upstream implementation expects a POST to `/redfish/v1/SessionService/Sessions` (without a trailing slash), with Basic authentication as well as the matching `UserName` and `Password` JSON fields. Use the returned `X-Auth-Token` header on subsequent requests. This login behavior differs from physical iLO.

## Included profiles

| Profile | Description | Default endpoint |
| --- | --- | --- |
| `DL380a_Gen12` | HPE ProLiant DL380a Gen12 sample dataset | `https://localhost:8443/redfish/v1/` |
| `DL360` | HPE ProLiant DL360 sample dataset, for testing a second response shape | `https://localhost:8444/redfish/v1/` when enabled |

To run both profiles together:

```sh
docker compose --profile second up --build -d
```

To change the first instance's profile or host port, copy `.env.example` to `.env`, edit it, and run the same start command again. For example:

```dotenv
MOCKUP_FOLDER=DL360
EXTERNAL_PORT=9443
```

Only the two profile names listed above are bundled. `DL360_Gen12` is omitted because it failed to load in the upstream revision used for this package.

## Accounts and network access

The upstream demonstration accounts are:

| Username | Password | Role |
| --- | --- | --- |
| `root` | `root_password` | Administrator |
| `operator` | `operator_password` | Operator |
| `guest` | `guest_password` | ReadOnly |

To replace all default accounts, set `AUTH_CONFIG` in your local `.env` file:

```dotenv
AUTH_CONFIG='demo:choose-a-lab-password:Administrator'
```

Separate multiple accounts with semicolons: `username:password:role;username:password:role`. Colons and semicolons cannot appear in usernames or passwords. Supported roles are `Administrator`, `Operator` and `ReadOnly`.

By default Docker publishes the ports on `127.0.0.1`, for access from the Docker host. To let a management pack builder or another lab machine connect, set `BIND_ADDRESS` in `.env` to the host's LAN IP, or `0.0.0.0` for all host interfaces. Use that host address in the API URL and allow the selected port through the host firewall if needed.

This emulator uses demonstration accounts and a development web server. Keep it on a trusted lab network. `.env` is excluded from Git and from the Docker build context.

## Logs, stop and reset

```sh
docker compose logs --tail 100 ilo
docker compose --profile second down
```

Changes made through the API are held in memory. Restarting the container reloads the bundled profile and generates a fresh self-signed certificate:

```sh
docker compose restart ilo
```

## Repository contents

```text
src/           Emulator runtime and pinned Python dependencies
mockups/       DL380a_Gen12 and DL360 sample Redfish datasets
Dockerfile     Builds the runtime image
compose.yaml   Starts one or two emulator instances
.env.example  Optional local configuration
LICENSE        Upstream HPE BSD 3-Clause licence
licenses/      DMTF licence for inherited emulator code
UPSTREAM.md    Source revision and local changes
```

The mock datasets originate from the public upstream project. They contain sample identifiers and recorded values; they do not track real hardware or continuously generate new performance measurements. Emulator results should be followed by testing against physical iLO when validating an integration.

## Licence and related projects

Upstream code is distributed under the [BSD 3-Clause licence](LICENSE). Original HPE and DMTF notices are retained; see [UPSTREAM.md](UPSTREAM.md).

- [HPE iLO management pack](https://github.com/mackayd/aria-operations-HPE-ILO-management-pack)
- [HPE OneView REST API emulator](https://github.com/mackayd/hpe-oneview-rest-api-emulator)
- [Dell Unity API emulator](https://github.com/mackayd/Unity-API-Emulator)
