# 🏋️ Ejercicios — Clase 9: procesos, señales y recursos

12 ejercicios progresivos. Los primeros son sobre el dataset `datos/ps_aux_practica.txt`
(así practicás sin depender de qué esté corriendo en tu máquina); los últimos son
**en vivo** sobre tu propio sistema.

> 🧠 Intentá **antes** de mirar `soluciones.md`. Equivocarse es parte del laburo.
> Si te trabás, mirá `docs/chuleta_procesos.md` primero.

Salvo que se diga otra cosa, trabajá parado en la raíz del repo:

```bash
cd datos
```

Recordá las columnas de `ps aux`:

```
USER   PID   %CPU  %MEM   VSZ    RSS   TTY   STAT  START  TIME   COMMAND
 $1     $2    $3    $4     $5     $6    $7    $8     $9    $10    $11...
```

---

## Bloque A — Leer procesos (`ps`)

### Ejercicio 1 — Los 5 que más CPU comen 🟢

Sobre `ps_aux_practica.txt`, mostrá las **5 líneas con mayor `%CPU`**, con este
formato: `%CPU  PID  COMANDO`. **No** muestres el encabezado.

> 💡 El encabezado se saltea con `NR>1` (awk) o `tail -n +2`.

---

### Ejercicio 2 — Los 3 que más RAM ocupan 🟢

Lo mismo pero por `%MEM`, mostrando `%MEM  PID  USUARIO  COMANDO`.
Después contestá: ¿el que más CPU come es el mismo que más RAM ocupa?

---

### Ejercicio 3 — Contar por estado 🟡

Contá **cuántos procesos hay en cada estado** (la primera letra de `STAT`),
ordenados de mayor a menor. Salida esperada del estilo:

```
     24 S
      3 R
      ...
```

---

### Ejercicio 4 — Cazar zombies 🟡

1. Listá **solo** los procesos en estado zombie, con `PID` y `COMMAND`.
2. Contálos.
3. Contestá: ¿por qué ese comando aparece entre corchetes y con `<defunct>`?

> ⚠️ Cuidado: hay un proceso en estado `D` que **no** es zombie. Si tu comando lo
> cuenta, está mal.

---

### Ejercicio 5 — ¿Quién tiene más procesos? 🟡

Contá cuántos procesos tiene **cada usuario** y ordenalos de mayor a menor.
Después: ¿cuánto `%CPU` **suma** el usuario `ulyses` entre todos sus procesos?

---

### Ejercicio 6 — Árbol de procesos 🟢 (en vivo)

En tu máquina:

1. Mostrá el árbol de procesos con `ps`.
2. Mostrálo con `pstree` incluyendo los PIDs.
3. Averiguá el **PPID** de tu shell actual (`$$` es el PID de tu shell).
4. ¿Quién es el PID 1 en tu sistema?

---

## Bloque B — Señales y control

### Ejercicio 7 — El ciclo completo de una señal 🟡 (en vivo)

1. Lanzá `sleep 600` en segundo plano.
2. Encontrá su PID **sin** mirar lo que imprimió la shell.
3. Mandale `SIGSTOP` y comprobá que su `STAT` cambió a `T`.
4. Reanudalo con `SIGCONT` y comprobá que volvió a `S`.
5. Terminalo **ordenadamente** (no con `-9`) y verificá que ya no está.

---

### Ejercicio 8 — `pgrep` antes de `pkill` 🟡 (en vivo)

1. Lanzá **tres** `sleep 500` en segundo plano.
2. Antes de matar nada, listá **qué** procesos coinciden con el patrón `sleep`,
   mostrando su línea de comando completa.
3. Ahora sí, terminá **solo los tuyos** (los de tu usuario) con una sola orden.
4. Verificá que no quedó ninguno.

> 🔴 ¿Por qué el paso 2 no es opcional? Escribí la respuesta en una línea.

---

### Ejercicio 9 — Primer y segundo plano 🟡 (en vivo)

1. Lanzá `ping -c 100 127.0.0.1 > /dev/null` en **primer** plano.
2. Pausalo con `Ctrl+Z`. ¿En qué estado quedó?
3. Listá tus trabajos.
4. Mandalo a **segundo** plano y verificá que siguió avanzando.
5. Traelo de vuelta al **primer** plano y cortalo con `Ctrl+C`.
6. ¿Qué señal manda `Ctrl+C`? ¿Y `Ctrl+Z`?

---

### Ejercicio 10 — Que sobreviva al SSH 🟠 (en vivo)

Tenés que lanzar un proceso largo por SSH y cerrar la sesión sin que muera.

1. Lanzá `sleep 300` de forma que **sobreviva** al cierre de la terminal, con la
   salida guardada en un archivo de registro.
2. Cerrá esa terminal, abrí otra y comprobá que sigue vivo.
3. Mirá su `PPID`. ¿Por quién fue adoptado y por qué?
4. Terminalo ordenadamente.
5. Escribí en una línea **qué señal** es la que lo habría matado sin esa protección.

---

## Bloque C — Recursos y diagnóstico

### Ejercicio 11 — Prioridades 🟠 (en vivo)

1. Lanzá `sleep 400` con la prioridad **más amable posible** (`NI` = 19).
2. Comprobá su `NI` con `ps`.
3. Intentá bajarle el `NI` a `-5` **sin** `sudo`. ¿Qué pasa y por qué?
4. Hacelo con `sudo` y verificá.
5. Terminalo.

---

### Ejercicio 12 — "El servidor está lento" 🔴 (en vivo)

Escribí un **procedimiento de guardia** de 7 pasos, con el comando de cada paso y
qué buscás en su salida, para diagnosticar (no arreglar) un servidor lento.

Tiene que cubrir, como mínimo:

- carga vs. cantidad de núcleos
- quién consume CPU y quién consume RAM
- memoria disponible y swap
- espacio en disco **y** inodos
- espera de entrada/salida
- procesos zombie
- errores recientes del sistema

**Bonus:** el `df -h` dice 100 % lleno, pero `du -sh /*` no encuentra dónde está
el espacio. ¿Qué está pasando y con qué comando lo confirmás?

---

## 📤 Entrega

Los ejercicios se entregan por **Pull Request** (ver `docs/reto_pull_request.md`).
El **🏁 reto de la bandera** NO: esa se la decís al profe.
