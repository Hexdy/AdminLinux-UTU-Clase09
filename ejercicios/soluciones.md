# ✅ Soluciones comentadas — Clase 9

> ⚠️ Mirá esto **después** de intentar. La solución sin el intento previo no te
> deja nada. Y ojo: casi siempre hay **más de una forma correcta**; si la tuya da
> el mismo resultado y la entendés, vale.

Todos los ejercicios de dataset se corren desde `datos/`.

---

## Ejercicio 1 — Los 5 que más CPU comen

```bash
awk 'NR>1 {print $3, $2, $11}' ps_aux_practica.txt | sort -rn | head -5
```

**Qué hace cada parte:**

- `NR>1` → saltea el encabezado (`NR` es el número de línea).
- `{print $3, $2, $11}` → `%CPU`, `PID` y el primer token del comando.
- `sort -rn` → orden **n**umérico (no alfabético) e inve**r**so (de mayor a menor).
- `head -5` → las 5 primeras.

**Resultado:**

```
63.5 1877 python3
9.8 1944 ffmpeg
7.9 2140 /usr/sbin/mariadbd
4.2 2011 rsync
2.4 1610 /usr/lib/firefox/firefox
```

> 💡 Si querés el comando **completo** (no solo `$11`), usá `cut -c66-` sobre el
> original o `awk '{ $1=$1; for(i=11;i<=NF;i++) printf "%s ", $i; print "" }'`.
> En procesos **reales** el equivalente es `ps aux --sort=-%cpu | head -6`.

**Error clásico:** `sort -r` sin `-n` ordena como texto y te pone `9.8` arriba de
`63.5` (porque "9" > "6" alfabéticamente).

---

## Ejercicio 2 — Los 3 que más RAM ocupan

```bash
awk 'NR>1 {print $4, $2, $1, $11}' ps_aux_practica.txt | sort -rn | head -3
```

**Resultado:**

```
6.8 2140 mysql /usr/sbin/mariadbd
4.2 1240 postgres /usr/lib/postgresql/15/bin/postgres
3.1 1610 ulyses /usr/lib/firefox/firefox
```

**¿Es el mismo que más CPU come?** **No.** El que más CPU come es el PID `1877`
(`python3 entrenar_modelo.py`, 63.5 %) y el que más RAM ocupa es el `2140`
(`mariadbd`, 6.8 %). Es lo normal: CPU y memoria son recursos distintos y el
cuello de botella puede estar en cualquiera de los dos.

> 🔎 En un sistema real, `%MEM` sale de `RSS` (memoria física real). **No** mires
> `VSZ`: es memoria virtual reservada y siempre parece gigante.

---

## Ejercicio 3 — Contar por estado

```bash
awk 'NR>1 {print substr($8,1,1)}' ps_aux_practica.txt | sort | uniq -c | sort -rn
```

- `substr($8,1,1)` → la **primera letra** de `STAT` (así `Ssl+` cuenta como `S`).
- `sort | uniq -c` → el clásico: `uniq -c` necesita la entrada **ordenada**.

**Resultado:**

```
     24 S
      3 R
      2 Z
      1 T
      1 D
```

Lectura: la enorme mayoría **durmiendo** (`S`) — eso es un sistema sano. Hay 3
corriendo, 2 zombies (algo hay que mirar), 1 detenido y 1 esperando disco.

---

## Ejercicio 4 — Cazar zombies

```bash
awk 'NR>1 && $8 ~ /^Z/ {print $2, $11, $12}' ps_aux_practica.txt
awk 'NR>1 && $8 ~ /^Z/' ps_aux_practica.txt | wc -l
```

**Resultado:**

```
1901 [convertir_csv] <defunct>
1902 [convertir_csv] <defunct>
2
```

**Claves:**

- `$8 ~ /^Z/` → la columna `STAT` **empieza** con `Z`. El `^` es lo que evita
  falsos positivos.
- **Por qué `[corchetes]` y `<defunct>`:** el proceso ya terminó y liberó su
  memoria, así que el kernel **ya no tiene** su línea de comando original. `ps`
  la muestra entre corchetes y marca `<defunct>` ("difunto").

**El error del ejercicio:** usar `grep Z` o `$8 == "Z"`.

- `grep Z` matchea cualquier `Z` en cualquier lado de la línea.
- `$8 == "Z"` falla si el estado trae modificadores (`Z+`, `Zl`).
- Y ojo con el `D` (PID `2011`, `rsync`): **no** es zombie, está esperando disco.

> 🧟 Recordá: a un zombie **no** se lo mata con `kill -9` — ya está muerto. Se va
> cuando el **padre** lee su código de salida (`wait()`). Si el padre está colgado,
> se le manda `SIGTERM` **al padre**. Si el padre muere, `systemd` adopta al zombie
> y lo limpia solo.

---

## Ejercicio 5 — ¿Quién tiene más procesos?

```bash
awk 'NR>1 {print $1}' ps_aux_practica.txt | sort | uniq -c | sort -rn
```

**Resultado:**

```
     12 ulyses
      8 root
      3 postgres
      2 www-data
      2 backup
      1 systemd+
      1 redis
      1 mysql
      1 message+
```

> 🔎 El orden **entre empates** puede variar según tu versión de `sort`. Lo que
> importa son los números, no quién queda arriba entre los que tienen 1.

Suma de `%CPU` de `ulyses`:

```bash
awk 'NR>1 && $1=="ulyses" {s+=$3} END {print s}' ps_aux_practica.txt
```

**Resultado:** `78.4`

> 💡 `s+=$3` acumula; `END {...}` se ejecuta **después** de la última línea. Es el
> patrón de acumulador de awk que vimos en la Clase 4.

**Lectura de sysadmin:** un solo usuario se está llevando el 78 % de la CPU. No es
necesariamente malo (puede ser una tarea legítima), pero si el servidor es
compartido, ahí tenés el candidato a `renice`.

---

## Ejercicio 6 — Árbol de procesos (en vivo)

```bash
ps -ef --forest | head -40      # 1
pstree -p | head -30            # 2
ps -o ppid= -p $$               # 3  (PPID de tu shell)
ps -p 1 -o pid,comm             # 4  (quién es el PID 1)
```

- `$$` es una variable de la shell con **su propio PID**.
- El `PPID` de tu shell suele ser tu emulador de terminal (`gnome-terminal-`,
  `konsole`, `tmux: server`) o `sshd` si entraste por SSH.
- El PID 1 es casi siempre **`systemd`** (en algunos contenedores puede ser otra cosa).

> 🔴 **Al PID 1 no se lo mata.** `sudo kill -9 1` deja el sistema sin el proceso que
> gestiona todos los servicios. En una VM se arregla reiniciando; en producción es
> un incidente.

---

## Ejercicio 7 — El ciclo completo de una señal

```bash
sleep 600 &                     # 1
PID=$(pgrep -u "$USER" -x sleep | tail -1)   # 2  sin mirar la salida de la shell
echo "$PID"

kill -STOP "$PID"               # 3
ps -p "$PID" -o pid,stat,cmd    #    STAT -> T

kill -CONT "$PID"               # 4
ps -p "$PID" -o pid,stat,cmd    #    STAT -> S

kill "$PID"                     # 5  = kill -15 = SIGTERM (ordenado)
sleep 1
ps -p "$PID" -o pid,stat,cmd    #    "no such process" / sin filas
```

- `pgrep -x sleep` → `-x` exige coincidencia **exacta** del nombre (no `sleepy`).
- `-u "$USER"` → solo los tuyos. **Mínimo privilegio.**
- `kill "$PID"` **sin número** ya es `SIGTERM`. No hace falta `-9`.

> 🔒 `SIGSTOP` (19) y `SIGKILL` (9) son las **dos únicas** señales que un proceso no
> puede atrapar ni ignorar: las aplica el kernel directamente.

---

## Ejercicio 8 — `pgrep` antes de `pkill`

```bash
sleep 500 & sleep 500 & sleep 500 &     # 1

pgrep -a -u "$USER" -x sleep            # 2  MIRÁ qué vas a matar
# 3120 sleep 500
# 3121 sleep 500
# 3122 sleep 500

pkill -u "$USER" -x sleep               # 3  SIGTERM a los tuyos, solo a los tuyos
pgrep -a -u "$USER" -x sleep            # 4  sin salida = listo
```

**Por qué el paso 2 no es opcional:** `pkill`/`killall` matan **todo** lo que
coincida con el patrón, y un patrón amplio (`pkill python`) puede llevarse puesto
un servicio de producción que no tenía nada que ver. `pgrep -a` es la versión de
**solo lectura** del mismo filtro: primero mirás, después actuás.

> ✅ Regla: **nunca** escribas un `pkill` que no hayas corrido antes como `pgrep -a`
> con exactamente los mismos filtros.

---

## Ejercicio 9 — Primer y segundo plano

```bash
ping -c 100 127.0.0.1 > /dev/null   # 1  primer plano
# Ctrl+Z                            # 2  queda detenido -> STAT = T
jobs -l                             # 3  [1]+ 3456 Detenido  ping ...
bg %1                               # 4  sigue, pero en segundo plano
jobs                                #    [1]+ Ejecutando
fg %1                               # 5  vuelve al primer plano
# Ctrl+C                            #    cortado
```

**Las señales:**

| Atajo | Señal | Nº | Qué hace |
|---|---|---|---|
| `Ctrl+C` | `SIGINT` | 2 | Interrumpir: le pide al proceso que corte |
| `Ctrl+Z` | `SIGTSTP` | 20 | Suspender: lo deja detenido (`T`) |

> 🔎 `Ctrl+Z` manda `SIGTSTP` (20), **no** `SIGSTOP` (19). La diferencia: `SIGTSTP`
> **sí** se puede atrapar (por eso `vim` o `less` pueden reaccionar distinto),
> `SIGSTOP` no.

**Trampa:** un proceso detenido con `Ctrl+Z` **no avanza**. Si pausás una descarga
y te vas, no descargó nada. Para que siga: `bg`.

---

## Ejercicio 10 — Que sobreviva al SSH

```bash
nohup sleep 300 > sleep.log 2>&1 &   # 1
# cerrás la terminal, abrís otra:
pgrep -a -u "$USER" -x sleep         # 2  sigue vivo
ps -o pid,ppid,stat,cmd -p <PID>     # 3  PPID = 1
kill <PID>                           # 4  ordenado
```

- **Paso 3:** el `PPID` pasó a **1** (`systemd`). Cuando muere el padre, el proceso
  queda **huérfano** y `systemd` lo **adopta**. Eso no es un error: es el mecanismo
  que garantiza que todo proceso tenga padre para recoger su código de salida (y que
  no queden zombies eternos).
- **Paso 5:** la señal que lo habría matado es **`SIGHUP` (1)** — *hang up*, herencia
  de los módems: cuando se cortaba la línea, se avisaba a todo lo que colgaba de esa
  terminal. `nohup` = "no hangup": hace que el proceso la **ignore**.

> 💡 En la vida real la solución profesional es **`tmux`** (o `systemd-run --user`):
> con `tmux` la sesión entera sigue viva del otro lado y podés **volver a entrar** a
> ver la salida, cosa que con `nohup` no podés.
>
> ```bash
> tmux new -s respaldo
> # Ctrl+b  d     (detach, sigue corriendo)
> tmux attach -t respaldo
> ```

---

## Ejercicio 11 — Prioridades

```bash
nice -n 19 sleep 400 &              # 1  el más amable posible
ps -o pid,ni,cmd -p $!              # 2  NI = 19   ($! = PID del último en 2do plano)

renice -n -5 -p $!                  # 3  FALLA
# renice: failed to set priority for 3456 (process ID): Permission denied

sudo renice -n -5 -p $!             # 4  ahora sí
ps -o pid,ni,cmd -p $!              #    NI = -5

kill $!                             # 5
```

**Por qué falla el paso 3:** `NI` va de **-20** (más prioridad) a **19** (menos).
Un usuario común **solo puede subir** el valor (ser más amable), **nunca bajarlo**.
Si pudiera, cualquiera pondría todos sus procesos en `-20` y se quedaría con la
máquina, dejando sin CPU al resto (y al propio sistema). **Solo root** puede pedir
más prioridad. Es control de recursos, no un capricho.

> ⚠️ `nice` no es magia: si el cuello de botella es el **disco** o la **red**, bajar
> el `NI` no mejora nada. Para disco existe `ionice`.

---

## Ejercicio 12 — "El servidor está lento"

```bash
# 1) ¿Hay cola? Load average contra cantidad de núcleos.
uptime; nproc
#    load > nproc => hay procesos esperando. Mirá la tendencia (1/5/15 min).

# 2) ¿Quién come CPU y quién come RAM?
top          # P ordena por CPU, M por memoria, c muestra el comando completo
#    o sin interactividad:
ps aux --sort=-%cpu | head -6
ps aux --sort=-%mem | head -6

# 3) ¿Queda memoria? ¿Está swapeando?
free -h
#    mirá 'available' (no 'free'). Si Swap used crece y crece => la máquina agoniza.

# 4) ¿Disco lleno? ¡Y los inodos!
df -h
df -i
#    se puede tener 40% libre de espacio y 100% de inodos ocupados (millones de
#    archivitos chicos): el sistema dice "No space left on device" igual.

# 5) ¿Está esperando entrada/salida?
vmstat 1 5
#    columna 'wa' alta => el cuello de botella es el disco, no la CPU.
#    columnas si/so con movimiento => está swapeando.

# 6) ¿Hay zombies?
ps aux | awk '$8 ~ /^Z/ {print $2, $11, $12}'
#    pocos zombies no molestan; miles = un padre roto que no hace wait().

# 7) ¿Qué dice el sistema?
journalctl -p err -n 50 --no-pager
dmesg -T | tail -30
#    buscá OOM-killer, errores de disco, servicios reiniciándose en loop.
```

**Recién después** de entender qué pasa, actuás: `renice` al proceso glotón,
`kill 15` (nunca `-9` de entrada), liberar disco, o avisarle al dueño del servicio.
Diagnosticar primero, actuar después.

### Bonus — `df` lleno pero `du` no encuentra nada

Hay un **archivo borrado que todavía está abierto** por un proceso. En Linux, `rm`
solo borra el **nombre**; el espacio se libera cuando **se cierra el descriptor**.
Un típico: alguien borró un log de 40 GB mientras el servicio lo tenía abierto —
`du` ya no lo ve (no hay nombre) pero el disco sigue ocupado.

```bash
sudo lsof | grep -i deleted
# o más acotado:
sudo lsof +L1
```

**Cómo se arregla:** reiniciando el servicio que lo tiene abierto (o truncándolo con
`: > /proc/<pid>/fd/<n>`, que es más quirúrgico y no corta el servicio).

> 🔒 **Lo que NO se hace:** `kill -9` al servicio para "liberar el disco". Se cierra
> igual pero a lo bruto, y si era una base de datos te llevaste los datos puestos.
> `SIGTERM` o un `systemctl restart` ordenado.
