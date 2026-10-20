# Hello Docker

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)

## Punto de partida

Esta rama continúa la app de las Partes 1 a 4, que ya tienes con su `Dockerfile`. Aquí la evolucionamos en dos pasos:

1. La app guarda los nombres en **PostgreSQL** en vez de en un archivo.
2. Un **Docker Compose** levanta la app y la base de datos a la vez.

El `Dockerfile` no cambia. El `docker-compose.yml` no viene en la rama: lo creas tú en el paso 2.

---

## Dockerfile o Docker Compose: cuál usar

Piensa en un restaurante. El **Dockerfile** es la receta de un plato: explica cómo preparar una sola cosa. **Docker Compose** es la comanda completa: dice qué platos salen juntos y cómo se conectan entre sí.

| | Dockerfile | Docker Compose |
|---|---|---|
| Qué es | Receta para construir **una imagen** | Archivo que describe **varios contenedores** y cómo trabajan juntos |
| Qué crea | Una imagen (la plantilla de tu app) | Los contenedores, la red entre ellos y los volúmenes |
| Cómo se usa | `docker build` y `docker run` | `docker compose up` |
| Qué contiene | Qué instalar y cómo arrancar tu app | Servicios, puertos, variables, volúmenes y orden de arranque |
| Archivo | `Dockerfile` | `docker-compose.yml` |

### Cuándo usamos cada uno

- **Solo Dockerfile:** tu app funciona sola, sin depender de nada más. Es lo que hicimos en las Partes 1 a 4: una app que guarda en un archivo.
- **Docker Compose:** tu app necesita otros servicios, como una base de datos. Con `docker run` tendrías que lanzar cada contenedor a mano, crear la red y acordarte de todas las opciones. Compose lo deja todo escrito en un archivo y se levanta con un solo comando.

### Se usan juntos

No son rivales. El Dockerfile sigue construyendo la imagen de la app, y el Compose la usa con `build: .`. Para la base de datos no hace falta Dockerfile porque usamos una imagen ya hecha (`postgres:16-alpine`).

```
Dockerfile  ->  construye la imagen de la app  ->  Compose la arranca junto a la base de datos
```

---

## Paso 0: Crear la carpeta del proyecto y clonar esta rama

crear carpeta:
```bash
mkdir practica-docker-compose
cd practica-docker-compose
```
Clona la rama y entra a la carpeta del proyecto:

```bash
git clone -b docker_compose https://github.com/Factoria-F5-madrid/hello_docker.git
cd hello_docker
```

- `git clone -b docker_compose`: descarga la rama `docker_compose` en vez de `main`.

> Si ya tienes la carpeta de la Parte 1, no clones encima. Clona en otra carpeta o usa `git fetch` y `git switch docker_compose` dentro de la tuya.

Comprueba los archivos:

```bash
ls
```

Debe salir `Dockerfile`, `app.py`, `requirements.txt` y `README.md`. No debe aparecer `docker-compose.yml`.

---

## Paso 1: Revisar el cambio en `app.py`

Los cambios están marcados con `CAMBIO` en el código. Son estos:

| Bloque | Antes | Ahora |
|---|---|---|
| `import psycopg2` | No existía | Driver para conectar Python con PostgreSQL |
| `DB_CONFIG` | No existía | Datos de conexión leídos de variables de entorno |
| Crear tabla | Crear carpeta `data/` | `CREATE TABLE IF NOT EXISTS nombres` |
| Guardar nombre | `f.write` en `nombres.txt` | `INSERT` en la tabla |
| Mostrar nombres | Leer `nombres.txt` | `SELECT` de la tabla |

La conexión sale de variables de entorno, así que el mismo código sirve en local y dentro de Compose:

```python
DB_CONFIG = {
    "host": os.getenv("DB_HOST", "localhost"),
    "dbname": os.getenv("DB_NAME", "hello"),
    "user": os.getenv("DB_USER", "hello"),
    "password": os.getenv("DB_PASSWORD", "hello_pass"),
}
```

En `requirements.txt` se añade `psycopg2-binary`. No necesitas instalarlo a mano: se instala al construir la imagen.

---

## Paso 2: Crear `docker-compose.yml`

Crea un archivo llamado `docker-compose.yml` en la raíz del proyecto, al lado del `Dockerfile`, y pega esto:

```yaml
# Dos servicios: la app Streamlit y una base de datos PostgreSQL
services:
  app:
    build: .                          # construye la imagen con el Dockerfile
    ports:
      - "8501:8501"                   # puerto-de-tu-ordenador:puerto-del-contenedor
    environment:                      # datos de conexión que lee app.py (DB_HOST, DB_NAME, DB_USER, DB_PASSWORD)
      DB_HOST: db                     # 'db' es el nombre del servicio de abajo: Compose lo resuelve como host
      DB_NAME: hello
      DB_USER: hello
      DB_PASSWORD: ${DB_PASSWORD:-hello_pass}
    depends_on:
      db:
        condition: service_healthy    # espera a que PostgreSQL acepte conexiones, no solo a que arranque

  db:
    image: postgres:16-alpine         # imagen oficial de PostgreSQL
    environment:
      POSTGRES_USER: hello
      POSTGRES_PASSWORD: ${DB_PASSWORD:-hello_pass}   # usa DB_PASSWORD si existe en .env
      POSTGRES_DB: hello
    volumes:
      - datos_db:/var/lib/postgresql/data             # los nombres se guardan aquí y sobreviven a docker compose down
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hello -d hello"]
      interval: 2s
      timeout: 3s
      retries: 15

volumes:
  datos_db:
```

Qué hace cada servicio:

- `app`: se construye con tu `Dockerfile` y expone el puerto 8501.
- `db`: usa la imagen oficial de PostgreSQL. No necesita Dockerfile propio.
- `environment`: pasa a la app los datos de conexión. `DB_HOST: db` usa el nombre del servicio como dirección.
- `depends_on` con `service_healthy`: la app arranca cuando la base de datos ya acepta conexiones.
- `volumes`: los nombres se guardan en el volumen `datos_db`, fuera del contenedor.

> **Opcional:** crea un archivo `.env` junto a `docker-compose.yml` con `DB_PASSWORD=tu_clave`. Si no existe, se usa `hello_pass`. El `.env` ya está en `.gitignore`, así que no se sube a GitHub.

---

## Paso 3: Levantar los dos servicios

```bash
docker compose up --build
```

- `up`: crea y arranca los servicios.
- `--build`: construye la imagen de `app` con el `Dockerfile`.

Cuando la app arranque, abre `http://localhost:8501`. Para parar, pulsa `Ctrl+C`.

---

## Paso 4: Comprobar que se guarda en PostgreSQL

1. Escribe un nombre en la app y pulsa **¡Saludar!**.
2. Recarga la página: el nombre sigue en la tabla.
3. Mira los datos directamente en la base de datos:

```bash
docker compose exec db psql -U hello -d hello -c "SELECT * FROM nombres;"
```

---

## Paso 5: Parar y limpiar

```bash
# Para y borra los contenedores. Los datos se conservan
docker compose down

# Para, borra los contenedores y el volumen. Se pierden los nombres
docker compose down -v
```

---

## Preguntas frecuentes

**¿Por qué la app ya no guarda en un archivo?**
Un archivo dentro de un contenedor se pierde si el contenedor se borra. Una base de datos en su propio servicio, con su volumen, guarda los datos de forma independiente.

**¿Puedo cambiar el puerto o la contraseña?**
Sí. El puerto está en `ports` (`"8502:8501"` para usar el 8502 en tu ordenador). La contraseña se cambia con el `.env`.

**¿Por qué `DB_HOST` es `db` y no `localhost`?**
Dentro de Compose, cada servicio se comunica con los demás por su nombre. `localhost` sería el propio contenedor de la app.
