# Fixes From Original

This document summarizes only the issues that prevented CRL from running and the fixes applied.

## 1) Issue: official image does not start the CLI

Symptom:
- Running `./crlcli init --use-crld` triggered OpenRC and repeated TTY errors
  (`/dev/tty1` ... `/dev/tty6`), and the process never advanced.

Cause:
- The image `docker.cs.kau.se/csma/cyber-range/crl` does not define a valid CLI `ENTRYPOINT`.
  With no entrypoint, Docker falls back to the image `CMD` (python3) without module/args, which
  triggers OpenRC on Alpine and blocks due to missing TTYs.

Fix applied:
- Forced the CLI invocation via `python3 -m crl` from `crl/crlcli`:
  `docker compose run --remove-orphans --rm --entrypoint python3 crl -m crl "$@"`

Result:
- The CLI could run `init`, `create`, and `start` without being blocked by TTY/OpenRC.

## 2) Issue: missing dependencies in the official image

Symptom:
- Running `python3 -m crl` raised `ModuleNotFoundError: No module named 'python_on_whales'`.

Cause:
- The official image did not have `python-on-whales` (and other CLI dependencies) installed.

Fix applied:
- Built a local image `crl:latest` using the repo `Dockerfile` to install dependencies.
- Ran the CLI with `CRL_IMAGE=crl` to force the local image.

Result:
- The CLI had all required dependencies and the commands worked.

## Operational note

To avoid the same issues on future runs:
- Always use `CRL_IMAGE=crl` when running `crlcli`.
