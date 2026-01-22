# Python 4º ESO — Ejercicios en Jupyter

Repositorio de actividades y ejercicios **resueltos** para alumnado de 4º ESO que está aprendiendo a programar en Python con Jupyter Notebook. 
El material está organizado por unidades (carpetas), con notebooks de trabajo para el alumnado y un `00_SOLUCIONES.ipynb` por unidad para corrección o repaso. 

## Estructura del repositorio

```

python_4eso/
02_control_flow/
00_SOLUCIONES.ipynb
00_if_elif_else.ipynb
01_while.ipynb
02_for_range.ipynb
03_break_continue.ipynb

03_collections/
00_SOLUCIONES.ipynb
00_listas.ipynb
01_tuplas_sets.ipynb
02_diccionarios.ipynb
03_comprehensions.ipynb

04_functions/
00_SOLUCIONES.ipynb
00_funciones_basicas.ipynb
01_scope.ipynb
02_tests_sencillos.ipynb

05_strings_files/
00_SOLUCIONES.ipynb
00_strings_pro.ipynb
01_archivos_txt.ipynb
02_csv_simple.ipynb

06_modules_packages/
00_SOLUCIONES.ipynb
00_import.ipynb
01_crear_modulo.ipynb

07_exceptions_debugging/
00_SOLUCIONES.ipynb
00_try_except.ipynb
01_debug_print.ipynb

08_oop/
00_SOLUCIONES.ipynb
00_clases_objetos.ipynb
01_encapsulacion_simple.ipynb
02_composicion.ipynb

09_projects/
proyecto_01_calculadora.ipynb
proyecto_02_adivina_numero.ipynb
proyecto_03_agenda_diccionario.ipynb
proyecto_04_pre_micropython.ipynb

99_reference/
00_cheatsheet.ipynb

```

## Qué hay en cada carpeta

- `02_control_flow/`: decisiones y bucles (if/elif/else, while, for/range, break/continue). 
- `03_collections/`: estructuras de datos (listas, tuplas, sets, diccionarios y comprehensions). 
- `04_functions/`: funciones, ámbito (scope) y autocomprobación con `assert`. 
- `05_strings_files/`: strings (métodos, f-strings) y ficheros (texto y CSV simple). 
- `06_modules_packages/`: imports y creación de módulos/paquetes propios. 
- `07_exceptions_debugging/`: manejo de errores (try/except) y depuración básica. 
- `08_oop/`: programación orientada a objetos (clases/objetos, encapsulación simple y composición). 
- `09_projects/`: proyectos integradores (calculadora, adivina el número, agenda con fichero, simulación tipo MicroPython). 
- `99_reference/`: chuleta rápida para repasar sintaxis y patrones frecuentes. 

## Cómo usar los notebooks (alumnado)

- Abre la carpeta `python_4eso/` en Jupyter (Lab o Notebook clásico) y entra en la unidad que toque. 
- En cada notebook de actividades, lee los ejemplos y completa las celdas `# TODO` de los 10 ejercicios y el reto. 
- Antes de entregar: **Kernel → Restart & Run All** para comprobar que todo funciona desde cero. 

## Soluciones (profesorado / repaso)

Cada unidad incluye un `00_SOLUCIONES.ipynb` con las soluciones completas de los ejercicios y el reto, pensado para corrección, repaso o autoevaluación. 

**********************************************

# Despliegue de JupyterHub (TLJH) con HTTPS (No‑IP + Let’s Encrypt)

## 1) Objetivo

Montar un servidor JupyterHub multiusuario en una VM accesible desde Internet y habilitar HTTPS válido con No‑IP + Let’s Encrypt.

## 2) Datos del entorno

- VM/Servidor: [you_vm_instance] (Linux, acceso por SSH).
- IP pública: [you_IP].
- Hostname No‑IP: [you_domain].
- Email Let’s Encrypt: [you_email].
- Admin TLJH/JupyterHub: [username].


## 3) Creación del servidor (requisitos base)

1) Crear una VM con un SO soportado por TLJH (Debian 11 o Ubuntu 20.04/22.04).
2) Asegurar recursos mínimos (TLJH sugiere al menos 1GB RAM) y conectividad pública por IP.
3) Abrir puertos en firewall/reglas cloud:

- TCP 22 (SSH).
- TCP 80 (HTTP).
- TCP 443 (HTTPS).
Esto es importante para el acceso web y para la emisión del certificado cuando se habilita HTTPS.


## 4) Instalación de TLJH (JupyterHub “The Littlest JupyterHub”)

### 4.1 Prerrequisitos en la VM

Conéctate por SSH y asegúrate de tener `python3`, `python3-dev`, `git` y `curl` instalados (en Ubuntu/Debian se instalan así).

```bash
sudo apt update
sudo apt install -y python3 python3-dev git curl
```


### 4.2 Ejecutar el instalador TLJH

La instalación típica se ejecuta con el script `bootstrap.py` (pipeado a `sudo python3 -`) y se indica el usuario admin inicial con `--admin`.
Ejemplo (admin `danrodcar`):

```bash
curl -L https://tljh.jupyter.org/bootstrap.py | sudo python3 - --admin admin_name
```

**Notas prácticas:**

- Este proceso puede tardar varios minutos.
- TLJH instala y configura JupyterHub + proxy (Traefik) y crea su estructura en `/opt/tljh` por defecto (según el bootstrap).


### 4.3 Primer acceso y verificación

1) Abre en navegador: `http://PUBLIC_IP` (al principio normalmente es HTTP).
2) En el servidor, comprueba la configuración TLJH:
```bash
sudo tljh-config show
```

`tljh-config` es la herramienta oficial para ver/cambiar la config de TLJH.

## 5) Copia de contenidos entre usuarios (/home)

Se necesitó copiar una carpeta (ej. `python_4eso`) entre homes.

Ejemplos:

```bash
sudo cp -r /home/jupyter-danrodcar/python_4eso /home/jupyter-abdou.diallo/
sudo cp -a python_4eso /home/
```

`-r` copia directorios recursivamente y `-a` preserva atributos (modo archive).

## 6) HTTPS sin dominio (problema inicial)

### 6.1 Síntoma

Al intentar habilitar HTTPS usando IP / certificados autofirmados, el navegador mostró “Su conexión no es privada” y `ERR_CERT_AUTHORITY_INVALID` (certificado no confiable).

### 6.2 Reversión temporal a HTTP

Para volver a HTTP:

```bash
sudo tljh-config set https.enabled false
sudo tljh-config reload proxy
```

`reload proxy` aplica cambios y reinicia el proxy para que entren en vigor.

## 7) Solución definitiva: No‑IP + Let’s Encrypt (HTTPS válido)

TLJH documenta HTTPS automático con Let’s Encrypt y requiere un dominio/hostname que apunte a la IP.

### 7.1 Crear hostname en No‑IP

- Se creó [tu_server] apuntando a [tu_ip].
- Verificación:

```bash
nslookup mihub.ddns.net
```

Debe devolver [tu_ip].

**Recordatorio importante (cuenta gratis):** No‑IP exige confirmar el hostname periódicamente (cada 30 días) para que no expire.

### 7.2 Activar Let’s Encrypt en TLJH

```bash
sudo tljh-config set https.enabled true
sudo tljh-config set https.letsencrypt.email carrion024@hotmail.com
sudo tljh-config add-item https.letsencrypt.domains tu_dominio
sudo tljh-config reload proxy
```

TLJH indica que tras recargar el proxy, este negocia con Let’s Encrypt para obtener el certificado.

### 7.3 Incidencia: configuración mezclada (manual TLS + Let’s Encrypt)

#### Síntoma

Aunque el dominio existía, el navegador seguía reportando certificado no válido.

#### Causa

En `tljh-config show` aparecían a la vez:

- `https.tls.key` / `https.tls.cert` (modo manual con cert propio/autofirmado)
- y `https.letsencrypt.*` (modo automático)
Esto provocaba que se siguiera sirviendo el certificado manual (no confiable).


#### Solución aplicada

Eliminar la configuración manual:

```bash
sudo tljh-config unset https.tls.key
sudo tljh-config unset https.tls.cert
sudo tljh-config reload proxy
```

Luego confirmar que ya no existe la sección `tls:`:

```bash
sudo tljh-config show
```


### 7.4 Pruebas de conectividad

```bash
curl -I http://mihub.ddns.net
```

Esperable: redirección (308/301) a `https://mihub.ddns.net/`.

```bash
curl -I https://mihub.ddns.net
```

Nota: `curl -I` hace HEAD; algunos endpoints pueden responder 405 aunque el sitio funcione con GET en navegador.

## 8) Mantenimiento

- Certificados Let’s Encrypt: TLJH los gestiona automáticamente una vez configurado, y el proxy es el componente que negocia el certificado tras `reload proxy`.
- Hostname No‑IP free: confirmar cada 30 días para que no expire.




