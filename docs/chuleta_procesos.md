# 🐧 Chuleta de procesos — AdminLinux Clase 9

Imprimila. Es la hoja que vas a mirar en el parcial y en el laburo.

---

## 1. Ver procesos: `ps`

| Comando | Qué hace |
|---|---|
| `ps` | Solo los procesos de **tu terminal** (casi nunca alcanza) |
| `ps aux` | **Todos** los del sistema, estilo BSD. El más usado |
| `ps -ef` | Todos, estilo UNIX. Muestra **PPID** (el padre) |
| `ps -ef --forest` | Todos, dibujados como **árbol** |
| `pstree -p` | El árbol, más lindo, con PIDs |
| `ps -u <usuario>` | Solo los de un usuario |
| `ps -p <pid> -o pid,ppid,stat,%cpu,%mem,cmd` | Un proceso, columnas a medida |
| `ps aux --sort=-%cpu \| head` | Los 10 que más CPU usan |
| `ps aux --sort=-%mem \| head` | Los 10 que más RAM usan |

### Columnas de `ps aux`

```
USER   PID   %CPU  %MEM   VSZ    RSS   TTY   STAT  START  TIME   COMMAND
 1      2     3     4      5      6     7     8      9     10     11
```

| Columna | Qué significa |
|---|---|
| `USER` | Dueño del proceso |
| `PID` | Su número único |
| `%CPU` | Porcentaje de CPU. **Puede pasar de 100** si usa varios núcleos |
| `%MEM` | Porcentaje de la RAM física |
| `VSZ` | Memoria **virtual** reservada (KB). Suele ser enorme y no asusta |
| `RSS` | Memoria **real** en RAM (KB). **Esta es la que importa** |
| `TTY` | Terminal asociada. `?` = no tiene (demonio) |
| `STAT` | Estado (ver abajo) |
| `START` | Cuándo arrancó |
| `TIME` | Tiempo de **CPU** consumido (no de reloj) |
| `COMMAND` | El comando con sus argumentos |

---

## 2. Estados (`STAT`)

| Letra | Estado | Qué está pasando |
|---|---|---|
| `R` | Running | Corriendo o listo para correr |
| `S` | Sleeping | Durmiendo, esperando algo. **Es lo normal** |
| `D` | Uninterruptible | Esperando disco/red. **No responde ni a `kill -9`** |
| `T` | Stopped | Detenido (`Ctrl+Z` o `SIGSTOP`) |
| `Z` | Zombie | Terminó, pero el padre no leyó su salida |
| `I` | Idle | Hilo del kernel sin trabajo |

Modificadores que se pegan al lado: `s` líder de sesión · `l` multihilo ·
`+` está en primer plano · `<` prioridad alta · `N` prioridad baja.

Ejemplo: `Ssl+` = durmiendo, líder de sesión, multihilo, en primer plano.

> ⚠️ **`D` no es `Z`.** Un `D` está trabajando (esperando al kernel): mirá el disco,
> el NFS o el dispositivo. Un `Z` ya está muerto: mirá al **padre**.

---

## 3. Monitores en vivo: `top` y `htop`

```bash
top          # viene en todos lados
htop         # más lindo, colores, mouse, scroll
```

Instalar `htop`:

| Distro | Comando |
|---|---|
| Debian / Ubuntu | `sudo apt install htop` |
| Arch | `sudo pacman -S htop` |
| Fedora / RHEL | `sudo dnf install htop` |

### Teclas dentro de `top`

| Tecla | Qué hace |
|---|---|
| `P` | Ordenar por CPU |
| `M` | Ordenar por memoria |
| `k` | Matar un proceso (te pide PID y señal) |
| `u` | Filtrar por usuario |
| `1` | Mostrar cada núcleo por separado |
| `c` | Ver la línea de comando completa |
| `h` | Ayuda |
| `q` | Salir |

### Teclas dentro de `htop`

| Tecla | Qué hace |
|---|---|
| `F3` | Buscar |
| `F4` | Filtrar |
| `F5` | Vista de árbol |
| `F6` | Elegir columna de orden |
| `F9` | Mandar señal (elegís cuál) |
| `F10` | Salir |

### Load average

```
load average: 2.31, 1.85, 1.42
                1min  5min  15min
```

**No es porcentaje.** Es cuántos procesos hay corriendo + esperando turno.
Se compara contra la cantidad de núcleos:

```bash
nproc        # cuántos núcleos tenés
```

| Load vs núcleos | Diagnóstico |
|---|---|
| Load < núcleos | Tranquilo |
| Load = núcleos | Al límite, sin cola |
| Load > núcleos | Hay cola: algo se está demorando |

Y mirá la **tendencia**: `8.0 / 4.0 / 2.0` está **empeorando**;
`2.0 / 4.0 / 8.0` ya **se está recuperando**.

---

## 4. Señales y `kill`

`kill` **no mata**: **manda una señal**. El nombre es histórico y confunde.

| Nº | Nombre | Qué es | ¿Se puede ignorar? |
|---|---|---|---|
| 1 | `SIGHUP` | "Se cerró tu terminal" / releé la config | Sí |
| 2 | `SIGINT` | Lo que manda `Ctrl+C` | Sí |
| 9 | `SIGKILL` | Ejecución inmediata por el kernel | **No** |
| 15 | `SIGTERM` | "Cerrá ordenado, por favor" (**por defecto**) | Sí |
| 18 | `SIGCONT` | Seguí | Sí |
| 19 | `SIGSTOP` | Congelate | **No** |

```bash
kill -l              # lista todas las señales de tu sistema
```

### 🔴 Regla profesional: TERM primero, KILL último

```bash
kill <pid>                  # = kill -15, pedido ordenado
sleep 5                     # dale tiempo a cerrar
ps -p <pid>                 # ¿sigue vivo?
kill -9 <pid>               # solo si NO cedió
```

Con `-9` el proceso **no se entera**: no guarda, no cierra archivos, no libera locks.
Base de datos con `-9` = datos corruptos.

### Familia completa

| Comando | Qué hace |
|---|---|
| `kill <pid>` | `SIGTERM` a ese PID |
| `kill -9 <pid>` | `SIGKILL` a ese PID (último recurso) |
| `kill -HUP <pid>` | Recargar config sin reiniciar |
| `pgrep <nombre>` | Te da los PIDs que coinciden |
| `pgrep -a <nombre>` | Ídem, **con la línea de comando** ← usalo antes de matar |
| `pgrep -u <user> <nombre>` | Filtrado por usuario |
| `pkill <nombre>` | `SIGTERM` a todos los que coincidan |
| `pkill -u <user> <nombre>` | Ídem, acotado a un usuario |
| `killall <nombre-exacto>` | `SIGTERM` por nombre **exacto** de ejecutable |

> ✅ **Siempre** `pgrep -a` antes de `pkill`/`killall`. Mirá qué vas a matar.

---

## 5. Primer y segundo plano

| Comando | Qué hace |
|---|---|
| `comando &` | Lanzarlo en **segundo plano** |
| `Ctrl+Z` | **Pausar** lo que está en primer plano (queda en `T`) |
| `jobs` | Ver los trabajos de **esta** terminal |
| `fg %1` | Traer el trabajo 1 al **primer** plano |
| `bg %1` | Reanudarlo en **segundo** plano |
| `Ctrl+C` | `SIGINT`: cortar lo que está en primer plano |
| `nohup comando &` | Que **sobreviva** al cierre de la terminal |
| `disown %1` | Sacarlo de la lista de la shell (ya lanzado) |

**¿Por qué muere al cerrar la terminal?** Porque al cerrarse te manda `SIGHUP` a
todo lo que colgaba de ella. `nohup` lo hace **ignorar** esa señal.

```bash
nohup ./respaldo.sh > respaldo.log 2>&1 &
```

Mejor todavía: **`tmux`** (sesión que sigue viva del otro lado del SSH).

```bash
tmux new -s trabajo     # crear
# Ctrl+b  d             # salir dejándolo vivo (detach)
tmux ls                 # listar
tmux attach -t trabajo  # volver
```

---

## 6. Prioridades: `nice` y `renice`

El valor `NI` va de **-20** (egoísta, más prioridad) a **19** (amable, menos prioridad).
Por defecto: **0**.

```bash
nice -n 10 ./tarea_pesada.sh     # arrancar amable
renice -n 5 -p <pid>             # cambiarle la amabilidad a uno vivo
renice -n 5 -u <usuario>         # a todos los de un usuario
```

> 🔒 **Subir** el `NI` (ser más amable) lo puede hacer cualquiera.
> **Bajarlo** (pedir más prioridad) lo puede hacer **solo root**. Si no, cualquier
> usuario se quedaría con la máquina.

---

## 7. Recursos del sistema

| Comando | Para qué |
|---|---|
| `free -h` | RAM y swap en formato humano |
| `uptime` | Hace cuánto está prendida + load average |
| `vmstat 1 5` | 5 muestras de CPU/memoria/IO cada 1 s |
| `df -h` | Espacio **libre por sistema de archivos** |
| `df -i` | **Inodos** libres (se llenan aunque sobre espacio) |
| `du -sh <dir>` | Cuánto **ocupa** ese directorio |
| `du -sh * \| sort -rh \| head` | Los 10 directorios más pesados de acá |
| `lsof -p <pid>` | Qué archivos tiene abiertos ese proceso |
| `lsof \| grep deleted` | Archivos **borrados pero todavía abiertos** |
| `lsof -i :<puerto>` | Quién tiene ese puerto |
| `ss -tulpn` | Puertos escuchando (**`ss`**, no `netstat`) |

En `free -h` mirá la columna **`available`**, no `free`: Linux usa la RAM libre
como caché de disco y la devuelve cuando hace falta.

> 💡 `df` dice "el disco está lleno" pero `du` no encuentra los archivos →
> hay un archivo **borrado y todavía abierto**. `lsof | grep deleted`.

---

## 8. `/proc` — el sistema de archivos que no existe

Todo lo de arriba sale de acá. `/proc` es **virtual**: lo genera el kernel al vuelo.

| Ruta | Qué tiene |
|---|---|
| `/proc/<pid>/cmdline` | El comando completo con el que arrancó |
| `/proc/<pid>/status` | Estado, memoria, PPID, usuario |
| `/proc/<pid>/cwd` | Enlace a su directorio de trabajo |
| `/proc/<pid>/fd/` | Los archivos que tiene abiertos |
| `/proc/cpuinfo` | Tus CPUs |
| `/proc/meminfo` | Memoria al detalle |
| `/proc/loadavg` | El load average crudo |

```bash
cat /proc/<pid>/cmdline | tr '\0' ' '; echo
```

---

## 9. 🚨 "El servidor está lento" — procedimiento de guardia

```bash
uptime              # 1. ¿hay cola? load vs nproc
nproc
top                 # 2. ¿quién come CPU? (P) ¿quién come RAM? (M)
free -h             # 3. ¿queda memoria? ¿está swapeando?
df -h               # 4. ¿disco lleno? (¡y df -i por los inodos!)
vmstat 1 5          # 5. ¿está esperando disco? (columna wa)
ps aux | awk '$8 ~ /^Z/'   # 6. ¿hay zombies?
journalctl -p err -n 50 --no-pager   # 7. ¿qué dice el sistema?
```

**Después** de entender qué pasa, actuás: `renice`, `kill 15`, limpiar disco,
avisar al dueño del servicio. Nunca al revés.

---

## 10. ⚠️ Lo que NO se hace

| ❌ Nunca | ✅ En su lugar |
|---|---|
| `kill -9` de entrada | `kill 15` → esperar → `-9` si no cede |
| `sudo kill -9 1` | Jamás: PID 1 es `systemd`, se cae todo |
| `kill -9` a un proceso en `D` | No sirve. Revisá disco/NFS |
| `killall` sin mirar | `pgrep -a` primero |
| `sudo` para todo | Mínimo privilegio |
| `netstat` | `ss` |
| `ifconfig` | `ip` |
| `chmod 777` | Permisos mínimos que funcionen |
| Comando de IA sin leer | Entender cada flag antes de Enter |
