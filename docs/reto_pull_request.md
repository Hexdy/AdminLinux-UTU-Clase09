# Reto de la semana — Clase 9 🏁 (la bandera)

Cerramos procesos como en un trabajo real: **desplegás el entorno con Ansible**,
hacés el **post-mortem** de un incidente leyendo procesos, capturás la **bandera**
y **se la decís al profe**.

> 🐧 No es hacking: es el laburo del SRE de guardia (mirar la foto forense de un
> servidor caído y sacar el diagnóstico). La bandera es la **recompensa** por hacerlo bien.

> 🏁 **Este reto NO se entrega por Pull Request.** No tenés que subir nada al repo:
> cuando tengas la bandera verificada, **se la decís al profe** (en clase o por el canal
> del curso). Eso es todo.

## Pasos

1. **Cloná fuera de Drive**:
   ```bash
   git clone <url> C:\src\AdminLinux-UTU-Clase09
   cd C:\src\AdminLinux-UTU-Clase09\playbooks
   ```
2. **Desplegá la estación con Ansible** (idempotente; simulá primero con `--check`):
   ```bash
   ansible-playbook armar_reto.yml -l local --check --diff
   ansible-playbook armar_reto.yml -l local
   ```
   Esto instala `htop`, crea `datos/reto/` con la foto forense y te deja tu `MISION.txt`.
   Corré el playbook **dos veces**: la 2ª da `changed=0` (idempotencia).
3. **Sacá los 3 datos del post-mortem** sobre `datos/reto/` (leé `MISION.txt`):
   - **R1** — el **PID** del proceso que se comió la CPU.
   - **R2** — **cuántos** procesos quedaron en estado **zombie**.
   - **R3** — el **número** de la señal que mandás **primero** para terminarlo ordenado.
4. **Armá la bandera** con el formato exacto y **verificala** con Ansible:
   ```bash
   ansible-playbook comprobar_bandera.yml -e "bandera=BANDERA{<pid>-<zombies>-<senal>}"
   ```
   Tiene que dar **✅ BANDERA CORRECTA**.
5. **Decile la bandera al profe** — en clase o por el canal del curso. Con eso el reto
   queda entregado. **No hace falta subir nada al repo** ni abrir un Pull Request.

## Cómo se evalúa (autoevaluación)

- [ ] Desplegaste con `armar_reto.yml` y lo corriste **2 veces** (la 2ª da `changed=0` → idempotente).
- [ ] **R1** sale de ordenar por `%CPU` **numéricamente** (columna 3), no alfabéticamente.
- [ ] **R2** cuenta solo los `STAT` (columna 8) que **empiezan** con `Z` — los `D` **no** cuentan.
- [ ] **R3** es la señal **ordenada**, no la brutal.
- [ ] Salteaste el **encabezado** del volcado (`NR>1` o `tail -n +2`).
- [ ] No te comiste la trampa de la nota de la guardia (trae un dato **incorrecto**).
- [ ] `comprobar_bandera.yml` te dio la bandera **correcta** (✅).
- [ ] Le **dijiste la bandera al profe** (en clase o por el canal del curso).

> 🔒 El repo no guarda la bandera en claro, solo su **hash SHA-256**. Así se guardan las
> contraseñas de verdad: nunca el secreto, solo su huella.

> 🤖 Si le pedís el `awk` a la IA, revisá que cada `$N` sea la columna correcta (`$3` es
> `%CPU`, `$8` es `STAT`) y probalo con `head` sobre una muestra antes de confiar en el
> número final.

---

## 📤 Y lo que SÍ va por Pull Request: los ejercicios

Los 12 ejercicios de [`ejercicios/ejercicios.md`](../ejercicios/ejercicios.md) sí se
entregan por PR:

```bash
git switch -c entrega/<tu-apellido>-clase09
# resolvés y guardás tus comandos + salidas
git add .
git commit -m "Clase 09: entrega de ejercicios"
git push -u origin entrega/<tu-apellido>-clase09
# abrís el Pull Request desde GitHub y completás la plantilla
```

> ⚠️ **Nunca** pongas la bandera en el PR, ni en un commit, ni en un issue. Se dice,
> no se escribe.
