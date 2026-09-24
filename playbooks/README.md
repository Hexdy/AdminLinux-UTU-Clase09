# Playbooks de Ansible — AdminLinux Clase 9 (procesos + el 🏁 reto de la bandera)

Un **escenario real de SRE**: un servidor se cayó de madrugada y ya lo reiniciaron.
El sistema vivo no existe más — te queda la **foto forense** que alcanzó a guardar la
guardia nocturna. Desplegás un playbook de Ansible que te arma la estación de trabajo
(instala `htop`, deja el volcado, prepara la misión) y después, leyendo procesos,
sacás **3 datos** para el post-mortem. Con esos 3 armás una **bandera** y la verificás
con otro playbook.

> 🐧 Con una sola VM alcanza: todo corre contra `localhost` (grupo `[local]`).
> No es hacking: es el laburo normal de diagnosticar un incidente de producción.

## Paso 0 — ¿Tenés todo? (te dice qué falta y cómo instalarlo)

```bash
cd playbooks
bash comprobar_requisitos.sh
```

Detecta tu distro y, si falta algo (Ansible, Python, `htop`), te da el comando exacto
para **apt**, **pacman** o **dnf**.

## Paso 1 — Desplegá el reto (armá la estación de guardia)

```bash
# Simular primero (no toca nada, buena costumbre):
ansible-playbook armar_reto.yml -l local --check --diff

# Aplicar de verdad (deja datos/reto/ listo):
ansible-playbook armar_reto.yml -l local
#   ↳ si al instalar htop te pide contraseña de sudo, agregá  -K
```

Atajo que hace el preflight + el armado:

```bash
bash correr.sh
```

Esto deja tu misión en `../datos/reto/MISION.txt`. Leela:

```bash
cat ../datos/reto/MISION.txt
```

## Paso 2 — Sacá los 3 datos del diagnóstico

Sobre `datos/reto/ps_aux_reto.txt` (y `top_cabecera.txt` para el contexto):

1. **R1** — ¿Cuál es el **PID** del proceso que se comió la CPU?
2. **R2** — ¿Cuántos procesos quedaron en estado **ZOMBIE**?
3. **R3** — ¿Qué **número de señal** mandás **primero** para terminarlo ordenado?

Armá la bandera con el formato exacto:
`BANDERA{<pid_culpable>-<cantidad_zombies>-<numero_senal>}`

> 💡 Los comandos exactos NO están acá a propósito: la idea es que los armes vos
> (el apunte de la clase te da todas las piezas: `sort -k3 -rn`, `awk` sobre `$8`,
> la tabla de señales). Si le pedís ayuda a una IA, **entendé cada columna** antes
> de correr nada.

> ⚠️ Dos trampas puestas a propósito: hay procesos en estado `D` (esperando disco —
> **no** son zombies y **no** se arreglan con `kill -9`), y la nota de la guardia
> trae un dato "de memoria" que está **mal**. Verificá vos.

## Paso 3 — Verificá tu bandera con Ansible

```bash
ansible-playbook comprobar_bandera.yml -e "bandera=BANDERA{...}"
```

- ✅ correcta → 🏁 trofeo.
- ❌ incorrecta → volvé a mirar cada dato.

El repo **no guarda la bandera en texto plano**: solo su huella **SHA-256** en
`secretos/bandera.sha256`. Es la misma idea con la que se guardan las contraseñas
(nunca el secreto, solo su hash). **Security by design.**

## 🏁 Cómo se entrega

**No hay Pull Request para este reto.** Cuando el playbook te dé
`✅ BANDERA CORRECTA`, **decile la bandera al profe** (en clase o por el canal del
curso). No subas la bandera al repo ni la pegues en un issue.

## Probá la idempotencia (lo más importante de Ansible)

Corré `armar_reto.yml` **dos veces seguidas**: la 1ª hace cambios (`changed`), la 2ª
tiene que dar **`changed=0`** (ya estaba como debía). Ese es el corazón de IaC:
describís el **estado deseado**, no pasos sueltos.

## 🔀 Multi-distro: el mismo playbook en 3 familias

| | Debian / Ubuntu | Arch | Fedora / RHEL |
|---|---|---|---|
| Gestor | `apt` | `pacman` | `dnf` |
| Paquete del monitor | `htop` | `htop` | `htop` |
| Paquete de `ps`/`top` | `procps` | `procps-ng` | `procps-ng` |
| Detección | `ansible_os_family == 'Debian'` | `'Archlinux'` | `'RedHat'` |

El playbook detecta la familia con `ansible_os_family` y el módulo genérico
`package` elige el gestor por vos. La instalación va dentro de un `block`/`rescue`:
si el gestor falla (sin red, sin sudo, repo caído) te avisa y **el reto sigue
funcionando igual**, porque `ps` ya viene en todos lados.

## Qué hay acá

| Archivo | Qué es |
|---|---|
| `armar_reto.yml` | Despliega la estación: htop + volcado + misión (tri-distro, idempotente). |
| `comprobar_bandera.yml` | Verifica tu bandera por hash (sin exponer el secreto). |
| `comprobar_requisitos.sh` | Preflight: detecta tu distro y te dice qué instalar. |
| `correr.sh` | Atajo: preflight + arma el reto. |
| `templates/mision.txt.j2` | Plantilla Jinja de la misión (se personaliza con tus facts). |
| `reto/ps_aux_reto.txt` | El volcado de `ps aux` del incidente (datos del reto). |
| `reto/top_cabecera.txt` | La cabecera de `top` del incidente (datos del reto). |
| `reto/nota_guardia.txt` | Nota de la guardia nocturna (se deja en 0600). |
| `secretos/bandera.sha256` | Solo la **huella** de la bandera (no el secreto). |
| `inventory.ini` / `ansible.cfg` | Qué máquinas administra Ansible + config local. |

## ⚠️ Seguridad

- Ningún playbook usa `chmod 777`, contraseñas en texto plano ni secretos versionados.
- La nota de la guardia queda en **0600** (mínimo privilegio) por diseño.
- `become: true` se usa **solo** en la tarea que instala el paquete, no en todo el play.
- La bandera se guarda como **hash**, no en claro.
- Simulá (`--check --diff`) antes de aplicar algo que no conozcas.
