# AdminLinux · Clase 9 — Procesos: `ps`, `top`, `htop`, `kill`, señales

Repo del alumno de la **Clase 9** de *Administración de Sistemas Linux* (UTU, S2).
Acá está todo lo que necesitás para practicar después de clase: chuleta, ejercicios
con soluciones, datos para jugar y el **🏁 reto de la bandera**.

---

## 🏁 El reto de hoy — empezalo YA, mientras escuchás

No esperes al final de la clase. Cloná esto ahora y andá resolviendo mientras
explico: llegás al cierre con la mitad hecha, igual que en un laburo real.

```bash
git clone <url-de-este-repo>
cd <carpeta-del-repo>/playbooks
bash correr.sh
cat ../datos/reto/MISION.txt
```

**¿Qué es el reto?** Un servidor se cayó de madrugada y ya lo reiniciaron. Te queda
la **foto forense** (`ps aux` + cabecera de `top`). Sos el/la SRE de guardia: tenés
que sacar 3 datos del post-mortem y armar la bandera.

**Los 3 pasos:**

1. **Desplegás** el escenario con Ansible (`bash correr.sh`).
2. **Diagnosticás** leyendo procesos: quién se comió la CPU, cuántos zombies quedaron,
   con qué señal se termina ordenado.
3. **Verificás** la 🏁 con `comprobar_bandera.yml` y **se la decís al profe**
   (no hace falta Pull Request).

👉 Todo el detalle en **[`playbooks/README.md`](playbooks/README.md)**.

---

## 🗂️ Qué hay acá

| Carpeta / archivo | Qué es |
|---|---|
| [`docs/chuleta_procesos.md`](docs/chuleta_procesos.md) | La chuleta de la clase: `ps`, `top`, `htop`, señales, `jobs`, `nice`, recursos. **Imprimila.** |
| [`docs/reto_pull_request.md`](docs/reto_pull_request.md) | Cómo se entrega cada cosa (la 🏁 se dice, los ejercicios van por PR). |
| [`ejercicios/ejercicios.md`](ejercicios/ejercicios.md) | 12 ejercicios progresivos, de leer `ps` a diagnosticar un incidente. |
| [`ejercicios/soluciones.md`](ejercicios/soluciones.md) | Las soluciones **comentadas** (mirá después de intentar, no antes). |
| [`datos/`](datos/) | Dataset de práctica (`ps_aux_practica.txt`) para entrenar sin romper nada. |
| [`playbooks/`](playbooks/) | 🏁 El reto de la bandera con Ansible (multi-distro e idempotente). |
| [`CONTEXTO_IA.md`](CONTEXTO_IA.md) | Pegale esto a tu IA para que te responda con el nivel de la clase. |

---

## 🚀 Arranque rápido

```bash
# 1. ¿Qué está corriendo en mi máquina?
ps aux | head

# 2. El árbol: quién es hijo de quién
ps -ef --forest | head -30

# 3. Monitor en vivo (q para salir)
top
# o mejor, si lo tenés:
htop

# 4. Buscar un proceso por nombre
pgrep -a firefox

# 5. Recursos: memoria y carga
free -h
uptime
```

¿No tenés `htop`? Se llama igual en las tres distros del curso:

```bash
sudo apt install htop      # Debian / Ubuntu
sudo pacman -S htop        # Arch
sudo dnf install htop      # Fedora / RHEL
```

---

## 🧠 Lo que tenés que llevarte de esta clase

- Un **programa** es la receta en el disco; un **proceso** es la receta **en ejecución**
  con su cocinero, su mesada y su número de orden (**PID**).
- Todo proceso tiene **padre** (`PPID`). El abuelo de todos es **PID 1** (`systemd`).
  **Al PID 1 no se lo mata.**
- `STAT` te dice cómo está: `R` corriendo, `S` durmiendo, `D` esperando disco,
  `T` detenido, `Z` **zombie**. Un zombie no consume CPU: es una entrada sin enterrar.
- **`kill` no mata: manda señales.** Primero `SIGTERM` (**15**) para que cierre
  ordenado; `SIGKILL` (**9**) es el último recurso porque no deja guardar nada.
- Un proceso lanzado en la terminal muere cuando la cerrás (`SIGHUP`). Para que
  sobreviva: `nohup`, `&` y, mejor todavía, `tmux`.
- **Load average** no es porcentaje: es *cuántos procesos hay en cola*. Se compara
  contra la cantidad de núcleos (`nproc`).

---

## ⚠️ Lo que NO se hace

| ❌ Antipatrón | ✅ Lo correcto |
|---|---|
| `kill -9` de entrada | `kill 15` primero, esperar, y `-9` solo si no cede |
| `sudo kill -9 1` | Nunca. PID 1 es `systemd`: se cae el sistema entero |
| `kill -9` a un proceso en `D` | No sirve: está esperando al kernel. Mirá el disco/NFS |
| `killall python3` a ciegas | `pgrep -a` primero, mirá qué vas a matar, después actuá |
| `sudo` para todo | Mínimo privilegio: a tus procesos los matás sin `sudo` |
| `chmod 777` "para que ande" | Permisos mínimos que funcionen (Clase 5) |
| Pegar el comando de la IA sin leer | Entender cada flag **antes** de apretar Enter |

---

## ✅ Autoevaluación

Contestá de memoria. Si dudás, volvé al apunte — la pista no te da la respuesta.

<details>
<summary>1. ¿Cuál es la diferencia entre un programa y un proceso, y qué es el PID?</summary>

💡 **Pista:** pensá en la receta del libro de cocina y en alguien cocinándola ahora.
¿Cuántas veces podés cocinar la misma receta al mismo tiempo? ¿Cómo distinguís una
olla de la otra?
</details>

<details>
<summary>2. Ves un proceso con <code>STAT</code> en <code>Z</code>. ¿Consume CPU? ¿Sirve hacerle <code>kill -9</code>? ¿Cómo se limpia?</summary>

💡 **Pista:** un zombie ya terminó. Lo que queda es una **entrada en la tabla**
esperando que alguien lea su código de salida. Preguntate **quién** tiene que leerlo
y qué pasa si ese alguien está roto o colgado.
</details>

<details>
<summary>3. ¿Por qué se manda <code>SIGTERM</code> antes que <code>SIGKILL</code>? ¿Qué pierde el proceso con <code>-9</code>?</summary>

💡 **Pista:** una señal se puede *atender*, la otra no llega ni al programa (la ejecuta
el kernel). Pensá en una base de datos a la mitad de escribir en disco: ¿qué pasa con
el buffer, los archivos temporales y los locks?
</details>

<details>
<summary>4. Tu servidor tiene <code>load average: 8.00</code>. ¿Está mal? ¿Qué te falta saber para contestar?</summary>

💡 **Pista:** load average no es un porcentaje, es una **cola**. La caja del súper con
1 cajero y la de 8 cajeros con la misma fila, ¿son el mismo problema? Hay un comando
de una palabra que te da el dato que falta.
</details>

<details>
<summary>5. Lanzás un proceso largo por SSH, cerrás el portátil y al volver no está. ¿Por qué, y cómo evitarlo?</summary>

💡 **Pista:** al morir la terminal, el sistema manda una señal específica a todo lo
que colgaba de ella. Buscá su nombre en la tabla de señales. Después pensá en dos
soluciones: una que **ignora** esa señal y otra en la que el proceso **nunca dependió**
de tu terminal.
</details>

---

## 📚 Para seguir

- `man ps`, `man top`, `man kill`, `man 7 signal`, `man nice`
- `kill -l` (lista todas las señales de tu sistema)
- Próxima clase: **servicios con `systemd`** — de matar procesos a mano a que el
  sistema los mantenga vivos solo.
