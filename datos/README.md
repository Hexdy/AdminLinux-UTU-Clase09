# 📁 datos/ — para practicar sin romper nada

| Archivo | Qué es |
|---|---|
| `ps_aux_practica.txt` | Un volcado de `ps aux` de una máquina de escritorio. Sirve para entrenar `awk`, `sort` y la lectura de columnas **sin** depender de qué esté corriendo en tu máquina. |

Las columnas son las de `ps aux`:

```
USER   PID   %CPU  %MEM   VSZ    RSS   TTY   STAT  START  TIME   COMMAND
 $1     $2    $3    $4     $5     $6    $7    $8     $9    $10    $11...
```

> Es un **archivo de texto**, no procesos reales: practicá `awk`, `sort`, `grep` y
> después aplicá lo mismo contra la salida real de `ps aux` en tu sistema.

```bash
# Ejemplo: los 5 que más CPU usan (salteando el encabezado)
awk 'NR>1 {print $3, $2, $11}' ps_aux_practica.txt | sort -rn | head -5
```

> ⚠️ `datos/reto/` (la carpeta que genera el playbook del 🏁 reto) está en
> `.gitignore`: es tuya y **no se commitea**.
