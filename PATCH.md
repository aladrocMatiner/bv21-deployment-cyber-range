# PATCH - Problemas Encontrados y Soluciones

Este documento resume unicamente los problemas que impidieron que CRL funcionara y las acciones
que fueron necesarias para solucionarlos.

## 1) Problema: la imagen oficial no arranca el CLI

Sintoma:
- Al ejecutar `./crlcli init --use-crld`, el contenedor intentaba arrancar OpenRC y generaba errores
  repetidos de TTY (`/dev/tty1` ... `/dev/tty6`) y no avanzaba.

Causa:
- La imagen `docker.cs.kau.se/csma/cyber-range/crl` no define un `ENTRYPOINT` valido para el CLI.
  Al no existir entrypoint, se usa el `CMD` de la imagen (python3) sin modulo/argumentos, lo que
  dispara OpenRC en Alpine y queda bloqueado por falta de TTY.

Solucion aplicada:
- Se forzo la ejecucion del CLI con `python3 -m crl` desde `crl/crlcli`:
  `docker compose run --remove-orphans --rm --entrypoint python3 crl -m crl "$@"`

Resultado:
- El CLI pudo ejecutar `init`, `create` y `start` sin quedarse bloqueado por TTY/OpenRC.

## 2) Problema: faltaban dependencias en la imagen oficial

Sintoma:
- Al ejecutar `python3 -m crl`, aparecia `ModuleNotFoundError: No module named 'python_on_whales'`.

Causa:
- La imagen oficial no tenia instalado el paquete `python-on-whales` (ni el resto de dependencias del CLI).

Solucion aplicada:
- Se construyo una imagen local `crl:latest` usando el `Dockerfile` del repo para instalar dependencias.
- Se ejecuto el CLI usando `CRL_IMAGE=crl` para forzar el uso de la imagen local.

Resultado:
- El CLI tuvo acceso a todas las dependencias y los comandos funcionaron.

## Nota operativa

Para repetir el despliegue sin volver a los problemas anteriores:
- Usar siempre `CRL_IMAGE=crl` al ejecutar `crlcli`.
