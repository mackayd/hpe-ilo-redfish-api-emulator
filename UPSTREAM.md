# Upstream source and local changes

## Source

- Project: [HewlettPackard/ilo-redfish-emulator](https://github.com/HewlettPackard/ilo-redfish-emulator)
- Revision: [`0bf00287731ca598e223372aa4a9627a101888c8`](https://github.com/HewlettPackard/ilo-redfish-emulator/tree/0bf00287731ca598e223372aa4a9627a101888c8)
- Included source: `src/`
- Included datasets: `mockups/DL380a_Gen12/` and `mockups/DL360/`
- Licence: original upstream [BSD 3-Clause LICENSE](LICENSE), retained unchanged.

The runtime includes code derived from the [DMTF Redfish Interface Emulator](https://github.com/DMTF/Redfish-Interface-Emulator). Original notices remain in the source files. Its BSD 3-Clause licence is also supplied in [licenses/DMTF-LICENSE.md](licenses/DMTF-LICENSE.md).

This repository packages an upstream emulator for convenient use; it does not claim authorship of HPE's implementation or datasets. It is a community package and is not endorsed by HPE or DMTF.

## Changes in this package

- Bundle only the two selected server profiles and the runtime source.
- Provide Docker Compose defaults, an optional second instance and an example environment file.
- Use Python 3.12 in the Docker image, a non-root runtime user and internal HTTPS port 8443.
- Pin Python dependencies to the versions used for release validation.
- Generate development certificates using SHA-256 instead of SHA-1.
- Stop logging the authentication configuration and stop startup when custom account configuration is invalid.
- Make the in-memory update worker a daemon so startup errors can terminate the process cleanly.
- Normalize the `HTTPS` setting and select a bundled profile by default.
- Allow `BIND_HOST` to select the Python listener address; the container uses `0.0.0.0`, with host exposure controlled by Compose.

Upstream mockup JSON content is retained unchanged. Deployment tools, virtual appliances and management-pack build artifacts are outside this runtime package.

## Validation on 29 October 2026

Both bundled profiles were started from the packaged source using Python 3.12.14. HTTPS service-root, system, manager, chassis, firmware and account queries passed. Basic authentication, invalid credentials, session creation/token queries, custom accounts, SHA-256 certificates and clean failure for invalid account configuration were checked.

All bundled JSON/XML profiles parsed and matched upstream byte for byte. Docker Compose passed the official specification schema; pinned dependency wheels resolved for Linux x86-64/Python 3.12. The package was checked for private lab identifiers and generated credentials/artifacts.

A Docker image build and container startup were not executed during this packaging pass because a Docker engine was unavailable. The live API checks exercised the packaged Python runtime and datasets directly.
