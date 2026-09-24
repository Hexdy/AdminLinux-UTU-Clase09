# 🤖 CONTEXTO_IA — pegale esto a tu asistente antes de preguntarle

Copiá y pegá **todo este bloque** al inicio de tu conversación con ChatGPT, Claude,
Gemini o Copilot. Así te responde al nivel de la clase y no te tira comandos de
administrador de sistemas con 15 años de calle que todavía no entendés.

---

## Prompt para copiar

```text
Sos mi tutor de Administración de Sistemas Linux. Estoy cursando la Clase 9 de
un curso de UTU (Uruguay), segundo semestre. El tema es PROCESOS.

LO QUE YA SÉ (no me lo expliques desde cero):
- Terminal, rutas, cd/ls/cat/less, pipes |, redirecciones > >> 2>
- grep y expresiones regulares, sed, awk, find, xargs
- Permisos rwx, chmod, chown, umask, sudo
- Gestión de paquetes: apt, pacman, dnf
- Red básica: ip a, ss -tulpn, ping, curl
- Ansible a nivel introductorio (playbooks, módulos, idempotencia)

LO QUE ESTOY APRENDIENDO AHORA (nivel: recién lo vi hoy):
- Qué es un proceso, PID, PPID, el árbol de procesos, systemd como PID 1
- Estados: R (running), S (sleeping), D (uninterruptible), T (stopped), Z (zombie)
- ps aux, ps -ef, ps -ef --forest, pstree
- top y htop: leer la cabecera, load average, ordenar, filtrar, matar desde ahí
- Señales: SIGHUP(1), SIGINT(2), SIGKILL(9), SIGTERM(15), SIGSTOP(19), SIGCONT(18)
- kill, killall, pkill, pgrep
- Primer/segundo plano: &, Ctrl+Z, jobs, fg, bg, nohup, tmux
- Prioridades: nice, renice, el valor NI
- Recursos: free -h, uptime, vmstat, df -h vs du -sh, lsof, /proc/<pid>/

CÓMO QUIERO QUE ME RESPONDAS:
1. Español rioplatense (voseo), directo, sin vueltas.
2. Antes de cualquier comando, una frase de qué hace y por qué.
3. Explicame cada flag que uses. Si usás un flag que no está en mi lista de arriba,
   avisame que es nuevo y explicalo.
4. Herramientas vigentes: ss (no netstat), ip (no ifconfig), journalctl, less (no more).
5. Nunca me des un comando destructivo sin advertencia explícita. En particular:
   kill -9, killall, rm -rf, redirecciones que pisan archivos.
6. Si hay una forma segura y una rápida-pero-peligrosa, mostrame la segura primero.
7. Multi-distro: si el comando cambia entre Debian/Ubuntu (apt), Arch (pacman) y
   Fedora (dnf), mostrame las tres.
8. Si te pido "matá tal proceso", recordame SIEMPRE el orden profesional:
   identificar → SIGTERM (15) → esperar → SIGKILL (9) solo si no cede.
9. No me des la respuesta del reto de la bandera. Si te pregunto por él, ayudame
   a entender los comandos, pero dejame sacar el resultado a mí.
10. Si no estás seguro, decímelo. Prefiero un "verificalo con man ps" a un invento.
```

---

## 🧪 Verificá lo que te dice la IA (siempre)

Una IA se equivoca con confianza. Antes de correr algo que te sugirió:

| Chequeo | Cómo |
|---|---|
| ¿Existe el comando/flag? | `man <comando>` o `<comando> --help \| grep -- --flag` |
| ¿A qué proceso le va a pegar? | `pgrep -a <patrón>` **antes** de `pkill` / `killall` |
| ¿Es destructivo? | Buscá `-9`, `-f`, `rm`, `>`, `sudo`. Si hay, frená y pensá |
| ¿Necesita `sudo` de verdad? | Tus propios procesos los manejás **sin** `sudo` |
| ¿Se puede simular? | Ansible: `--check --diff`. Shell: probá en `/tmp` primero |

**Regla de la clase:** si no podés explicar **qué hace cada parte** del comando,
no lo corras. Preguntale a la IA "explicame ese comando parte por parte".

---

## 🚩 Preguntas que te conviene hacerle

Buenas (te enseñan):

- "Explicame columna por columna esta línea de `ps aux`: `<pegás la línea>`"
- "¿Por qué este proceso está en estado `D` y no responde a `kill -9`?"
- "Mi `load average` es 12.00 y tengo 4 núcleos. ¿Qué significa? ¿Qué miro después?"
- "Diferencia entre `kill`, `killall`, `pkill` y `pgrep`, con un ejemplo de cada uno"
- "¿Por qué mi script muere cuando cierro SSH y cómo lo evito?"
- "Armame un procedimiento de diagnóstico para 'el servidor está lento'"

Malas (te sacan el aprendizaje):

- "Resolveme el reto de la bandera" ← **no**, es tuyo
- "Dame el comando para matar todo lo que consuma CPU" ← receta para romper producción
- "Hacé que mi proceso ande rápido" ← sin diagnóstico no hay solución

---

## ⚠️ Trampas típicas de las IA en este tema

| Lo que suele decir la IA | Por qué está mal / incompleto |
|---|---|
| "Usá `kill -9` para matarlo" | Salta el paso ordenado. `SIGTERM` primero, siempre |
| "Los zombies se matan con `kill -9 <pid>`" | Un zombie **ya está muerto**. Se va cuando el **padre** lo recoge, o cuando matás/reiniciás al padre |
| "Load average 5 = 500% de CPU" | Load no es porcentaje. Se compara contra `nproc` |
| "`netstat -tulpn` para ver puertos" | `netstat` está obsoleto: usá `ss -tulpn` |
| "`killall -9 nombre` y listo" | Mata **todos** los que coincidan, en todo el sistema. `pgrep -a` primero |
| "Borrá `/proc/<pid>`" | `/proc` es virtual, no se borra. No es un directorio de verdad |
| "Subile la prioridad con `nice -n -10`" | Bajar el `NI` (más prioridad) requiere **root**. Un usuario común solo puede subirlo |
| "El proceso usa 2 GB porque `VSZ` dice 2 GB" | `VSZ` es memoria **virtual reservada**. La que importa es `RSS` (la real en RAM) |

---

## 🔐 Seguridad al usar IA en administración de sistemas

1. **Nunca** le pegues a una IA: contraseñas, claves privadas, `/etc/shadow`,
   tokens, datos de clientes, IPs internas completas o nombres de servidores de
   producción. Anonimizá antes de pegar.
2. Si te da un comando con `sudo`, preguntate: *¿de verdad necesito root para esto?*
   Mínimo privilegio.
3. Si te da un one-liner largo con pipes y `xargs`, pedile que lo parta en pasos
   y correlo por partes (primero sin la acción destructiva).
4. Antes de `pkill`/`killall`, corré la versión de **solo lectura**: `pgrep -a`.
5. Jamás corras `curl ... | bash` de un link que te tiró una IA. Bajá, leé, después corré.

---

## 📎 Mini-glosario para que la IA y vos hablen el mismo idioma

| Término | Qué es |
|---|---|
| **PID** | Número único de un proceso vivo |
| **PPID** | PID del padre (quién lo lanzó) |
| **STAT** | Estado: `R` corriendo, `S` durmiendo, `D` esperando disco, `T` detenido, `Z` zombie |
| **RSS** | Memoria **real** en RAM (la que importa) |
| **VSZ** | Memoria **virtual** reservada (casi siempre mucho mayor que la real) |
| **TIME** | Tiempo de **CPU** acumulado (no tiempo de reloj) |
| **Señal** | Aviso que el kernel le entrega a un proceso (`SIGTERM`, `SIGKILL`, ...) |
| **Load average** | Cantidad promedio de procesos en cola (1, 5 y 15 min) |
| **NI** (nice) | Amabilidad: de `-20` (egoísta, más prioridad) a `19` (amable, menos) |
| **Huérfano** | Proceso cuyo padre murió; lo adopta PID 1 |
| **Zombie** | Proceso terminado cuya entrada sigue en la tabla porque el padre no la leyó |
| **Demonio** | Proceso de fondo sin terminal, normalmente hijo de `systemd` |

---

> 💡 Este archivo es parte del material de la Clase 9. Si la IA te contradice, gana
> el `man` de tu sistema. Verificá siempre.
