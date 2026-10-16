# Hello Docker

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)

Mini aplicación web en **Python + Streamlit** creada para aprender **Docker paso a paso**.

---

## Punto de partida

Esta rama empieza donde terminan las **Partes 1 a 4**. Ya tienes una app Streamlit con su `Dockerfile` que funciona en local y en Docker, y esa parte ya la has hecho.

Aquí la evolucionamos en dos pasos:

1. La app guarda los nombres en **PostgreSQL** en vez de en un archivo.
2. Un **Docker Compose** levanta la app y la base de datos a la vez.

En esta rama encontrarás el `Dockerfile` y el código ya cambiado. El `docker-compose.yml` lo creas tú en la Parte 5.

### Cómo clonar esta rama

```bash
git clone -b docker_compose https://github.com/Factoria-F5-madrid/hello_docker.git
cd hello_docker
```

Si ya tienes la carpeta de la Parte 1, no clones encima: clona en otra carpeta o usa `git fetch` y `git switch docker_compose` dentro de la tuya.

---

## Objetivo del ejercicio

Con este proyecto aprenderás a:

- Ejecutar una aplicación web en Python en local
- Crear tu propio Dockerfile
- Construir una imagen Docker
- Ejecutar un contenedor
- Etiquetar tu imagen para Docker Hub
- Subir tu imagen a Docker Hub
- Ejecutar la app desde Docker Hub


---

## Ramas del repositorio

Este repositorio contiene **dos ramas** con objetivos distintos:

### Rama `main` (ejercicio para realizarlo)
- Es la rama que deben clonar para realizar el ejercicio
- Contiene la aplicación en Python + Streamlit
- Presentación Docker en PDF
- **NO incluye Dockerfile**
- El objetivo es que el alumno:
  - cree su propio Dockerfile
  - construya la imagen Docker
  - ejecute el contenedor
  - trabaje con volúmenes
  - suba su imagen a Docker Hub

 **Esta es la rama que se debe usar para realizar el ejercicio.**

---

### Rama `docker_solution` (solución completa)
- Contiene el proyecto **completamente terminado**
- Incluye:
  - Dockerfile
  - proyecto listo para ejecutarse
  - **guía de instalación de Docker Desktop**
- Pensada como referencia o solución final

 Para acceder a esta rama:

```bash
git checkout docker_solution

```

## Estructura del proyecto
```
hello_docker/
│
├── app.py # Aplicación Streamlit
├── requirements.txt # Dependencias Python
├── Dockerfile # Instrucciones para Docker
├── docker-compose.yml # Levanta app + base de datos
└── README.md # Este archivo
```
---

## PARTE 1: Ejecutar la aplicación en local

Antes de dockerizar, vamos a probar que todo funciona correctamente.

### 1. Crear entorno virtual
```bash
python -m venv .venv
```

### 2. Activar el entorno virtual

**Windows:**
```bash
.venv\Scripts\activate
```

**Mac / Linux:**
```bash
source .venv/bin/activate
```

### 3. Instalar dependencias
```bash
pip install -r requirements.txt
```

### 4. Ejecutar la aplicación
```bash
streamlit run app.py
```

 La aplicación debería abrirse automáticamente en `http://localhost:8501`

---

## PARTE 2: Dockerizar la aplicación

Ahora vamos a crear un contenedor Docker para que la aplicación pueda ejecutarse en cualquier lugar.

### Paso 1: Crear el Dockerfile

Crea un archivo llamado `Dockerfile` (sin extensión) en la raíz del proyecto y añade el siguiente contenido:
```dockerfile
# Imagen base de Python
FROM python:3.11-slim

# Establecer directorio de trabajo dentro del contenedor
WORKDIR /app

# Copiar el archivo de dependencias
COPY requirements.txt .

# Instalar las dependencias
RUN pip install --no-cache-dir -r requirements.txt

# Copiar el código de la aplicación
COPY app.py .

# Exponer el puerto 8501 (puerto por defecto de Streamlit)
EXPOSE 8501

# Comando para ejecutar la aplicación
CMD ["streamlit", "run", "app.py", "--server.address=0.0.0.0"]
```

#### ¿Qué hace cada línea?

- `FROM python:3.11-slim` → Usa una imagen ligera de Python 3.11
- `WORKDIR /app` → Crea y establece `/app` como directorio de trabajo
- `COPY requirements.txt .` → Copia el archivo de dependencias al contenedor
- `RUN pip install...` → Instala las dependencias de Python
- `COPY app.py .` → Copia el código de la aplicación
- `EXPOSE 8501` → Indica que el contenedor escucha en el puerto 8501
- `CMD [...]` → Comando que se ejecuta al iniciar el contenedor

---

### Paso 2: Construir la imagen Docker

Abre una terminal en la carpeta del proyecto y ejecuta:
```bash
docker build -t hello_docker .
```

#### ¿Qué hace este comando?

- `docker build` → Construye una imagen Docker
- `-t hello_docker` → Asigna un nombre (tag) a la imagen
- `.` → Indica que el Dockerfile está en el directorio actual

---

### Paso 3: Ejecutar el contenedor

Una vez construida la imagen, ejecuta:
```bash
docker run -p 8501:8501 hello_docker
```

#### ¿Qué hace este comando?

- `docker run` → Ejecuta un contenedor desde una imagen
- `-p 8501:8501` → Mapea el puerto 8501 del contenedor al puerto 8501 de tu máquina
- `hello_docker` → Nombre de la imagen a ejecutar

 Abre tu navegador en `http://localhost:8501` y verás tu aplicación corriendo en Docker.

---


## PARTE 2.5: Persistencia de datos con volúmenes (MUY IMPORTANTE)

Nuestra aplicación guarda los nombres introducidos por el usuario en un archivo:
```
data/nombres.txt
```

Este archivo **se crea automáticamente** cuando ejecutas la app por primera vez.

---

### Ejecutar el contenedor SIN volumen (los datos se pierden)

Ejecuta el contenedor de forma normal:
```bash
docker run -p 8501:8501 hello_docker
```

1. Escribe varios nombres en la aplicación
2. Verás que aparecen en la tabla
3. Detén y elimina el contenedor:
```bash
docker ps
docker rm -f
```

4. Vuelve a ejecutar el contenedor:
```bash
docker run -p 8501:8501 hello_docker
```

** Resultado:** Los nombres se han perdido.

** ¿Por qué?** Los datos estaban dentro del contenedor, y los contenedores son temporales.

---

### Ejecutar el contenedor CON volumen (los datos se conservan)

Ahora ejecuta el contenedor usando un volumen Docker:
```bash
docker run -p 8501:8501 -v datos_app:/app/data hello_docker
```

#### ¿Qué hace este comando?

- `-v datos_app:/app/data` → Crea un volumen llamado `datos_app` y lo monta en `/app/data` del contenedor

**Prueba:**

1. Escribe varios nombres en la aplicación
2. Verás que aparecen en la tabla
3. Detén y elimina el contenedor:
```bash
docker ps
docker rm -f
```

4. Vuelve a ejecutar el contenedor **con el mismo comando**:
```bash
docker run -p 8501:8501 -v datos_app:/app/data hello_docker
```

** Resultado:** Los nombres siguen apareciendo.

** Explicación:** El volumen `datos_app` guarda los datos **fuera del contenedor**, por eso no se pierden.

---

### Comandos útiles para volúmenes
```bash
# Ver todos los volúmenes
docker volume ls

# Ver detalles de un volumen específico
docker volume inspect datos_app

# Eliminar un volumen (CUIDADO: borra todos los datos)
docker volume rm datos_app

# Eliminar todos los volúmenes no usados
docker volume prune
```

> **Importante:** Si eliminas el volumen con `docker volume rm datos_app`, perderás todos los datos guardados.

---

## PARTE 3: Subir tu imagen a Docker Hub

Para compartir tu imagen con otros o ejecutarla en cualquier máquina, necesitas subirla a Docker Hub.

### Paso 0: Crear cuenta en Docker Hub (si no la tienes)

1. Ve a [hub.docker.com](https://hub.docker.com)
2. Crea una cuenta gratuita
3. Anota tu nombre de usuario (lo necesitarás en los siguientes pasos)

### 1. Iniciar sesión en Docker Hub
```bash
docker login
```

Introduce tu usuario y contraseña de Docker Hub.

### 2. Etiquetar la imagen con tu usuario y versión

Antes de subir la imagen, necesitas etiquetarla con tu nombre de usuario de Docker Hub y una versión:
```bash
docker tag hello_docker TU_USUARIO/hello_docker:v1.0
docker tag hello_docker TU_USUARIO/hello_docker:latest
```

**Ejemplos:**
- Si tu usuario es `gema`:
```bash
  docker tag hello_docker gema/hello_docker:v1.0
  docker tag hello_docker gema/hello_docker:latest
```
- Si tu usuario es `juan123`:
```bash
  docker tag hello_docker juan123/hello_docker:v1.0
  docker tag hello_docker juan123/hello_docker:latest
```

#### ¿Por qué hacer esto?

- **Usuario:** Docker Hub organiza las imágenes por usuario (como GitHub con los repositorios). El formato es: `usuario/nombre-imagen:version`
- **Versión (`v1.0`):** Identifica versiones específicas de tu aplicación. Útil para mantener diferentes releases.
- **Tag `latest`:** Apunta siempre a la versión más reciente. Es la que se descarga por defecto si no especificas versión.

> **Buena práctica:** Siempre etiqueta con una versión específica (`v1.0`, `v2.0`) Y con `latest`. Así los usuarios pueden elegir una versión concreta o usar siempre la última.

### 3. Subir las imágenes
```bash
docker push TU_USUARIO/hello_docker:v1.0
docker push TU_USUARIO/hello_docker:latest
```

**Ejemplo:**
```bash
docker push gema/hello_docker:v1.0
docker push gema/hello_docker:latest
```

 Tu imagen ya está disponible públicamente en Docker Hub con ambas etiquetas.

---

## PARTE 4: Ejecutar la app desde Docker Hub

Cualquier persona puede ejecutar tu aplicación de estas formas:

**Usando la versión específica:**
```bash
docker run -p 8501:8501 -v datos_app:/app/data TU_USUARIO/hello_docker:v1.0
```

**Usando la última versión (latest):**
```bash
docker run -p 8501:8501 -v datos_app:/app/data TU_USUARIO/hello_docker:latest
```

**Sin especificar versión (descarga `latest` por defecto):**
```bash
docker run -p 8501:8501 -v datos_app:/app/data TU_USUARIO/hello_docker
```

**Ejemplo:**
```bash
docker run -p 8501:8501 -v datos_app:/app/data gema/hello_docker:v1.0
```

Docker descargará automáticamente la imagen desde Docker Hub y la ejecutará.

> **Tip:** Si haces cambios en tu app y quieres subir una nueva versión, cambia el número de versión (ej: `v2.0`) y repite los pasos de etiquetado y push.

---

## PARTE 5: De un Dockerfile a Docker Compose con base de datos

### Qué cambiamos

Hasta la Parte 4 la app se ejecutaba en un solo contenedor y guardaba los nombres en un archivo dentro de `data/`. Ese archivo vive dentro del contenedor, así que los datos dependen de un volumen.

En esta parte la app evoluciona en dos pasos:

| Antes (Partes 1 a 4) | Ahora (Parte 5) |
|---|---|
| Un contenedor: la app | Dos servicios: `app` y `db` |
| Los nombres se guardan en `data/nombres.txt` | Los nombres se guardan en una tabla de PostgreSQL |
| `docker build` y `docker run` con muchas opciones | Un solo `docker compose up --build` |

### Paso 1: Clonar esta rama

Esta rama tiene el código de la app (con el cambio a PostgreSQL) y el `Dockerfile`, pero **no tiene** `docker-compose.yml`. Ese archivo lo creas tú en el paso 3.

```bash
git clone -b docker_compose https://github.com/Factoria-F5-madrid/hello_docker.git
cd hello_docker
```

- `git clone -b docker_compose`: descarga la rama `docker_compose` en vez de `main`.
- `cd hello_docker`: entra en la carpeta que se acaba de crear.

> Si ya tienes la carpeta de la Parte 1, no clones encima. Clona en otra carpeta o cambia de rama dentro de la tuya con `git fetch` y `git switch docker_compose`.

Comprueba que tienes estos archivos:

```bash
ls
```

Debe salir `Dockerfile`, `app.py`, `requirements.txt` y `README.md`. No debe aparecer `docker-compose.yml`.

### Paso 2: Qué cambia en `app.py`

Busca los comentarios `CAMBIO` en el código. Son los únicos cambios:

| Bloque | Antes | Ahora |
|---|---|---|
| `import psycopg2` | No existía | Driver para conectar Python con PostgreSQL |
| `DB_CONFIG` | No existía | Datos de conexión leídos de variables de entorno |
| Crear tabla | Crear carpeta `data/` | `CREATE TABLE IF NOT EXISTS nombres` |
| Guardar nombre | `f.write` en `nombres.txt` | `INSERT` en la tabla |
| Mostrar nombres | Leer `nombres.txt` | `SELECT` de la tabla |

En `requirements.txt` se añade `psycopg2-binary`. El `Dockerfile` no cambia.

### Paso 3: Crear `docker-compose.yml`

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

### Paso 4: Levantar los dos servicios

```bash
docker compose up --build
```

- `up`: crea y arranca los servicios.
- `--build`: construye la imagen de `app` con el `Dockerfile`.

Cuando veas que la app arranca, abre `http://localhost:8501`. Para parar pulsa `Ctrl+C`.

### Paso 5: Comprobar que se guarda en PostgreSQL

1. Escribe un nombre en la app y pulsa **¡Saludar!**.
2. Recarga la página: el nombre sigue en la tabla.
3. Mira los datos directamente en la base de datos:

```bash
docker compose exec db psql -U hello -d hello -c "SELECT * FROM nombres;"
```

### Paso 6: Parar y limpiar

```bash
# Para y borra los contenedores. Los datos se conservan
docker compose down

# Para, borra los contenedores y el volumen. Se pierden los nombres
docker compose down -v
```

---

## Comandos útiles de Docker
```bash
# Ver imágenes locales
docker images

# Ver contenedores en ejecución
docker ps

# Ver todos los contenedores (incluyendo detenidos)
docker ps -a

# Detener un contenedor
docker stop <container_id>

# Eliminar un contenedor
docker rm <container_id>

# Eliminar una imagen
docker rmi <image_name>

# Ver logs de un contenedor
docker logs <container_id>

# Ejecutar contenedor en segundo plano (detached mode)
docker run -d -p 8501:8501 hello_docker
```

---

## Resumen del flujo completo
```
1. Crear Dockerfile
   ↓
2. docker build -t hello_docker .
   ↓
3. docker run -p 8501:8501 -v datos_app:/app/data hello_docker
   ↓
4. docker login
   ↓
5. docker tag hello_docker TU_USUARIO/hello_docker:v1.0
   docker tag hello_docker TU_USUARIO/hello_docker:latest
   ↓
6. docker push TU_USUARIO/hello_docker:v1.0
   docker push TU_USUARIO/hello_docker:latest
   ↓
7. Cualquiera puede ejecutar:
   docker run -p 8501:8501 -v datos_app:/app/data TU_USUARIO/hello_docker:v1.0
   o
   docker run -p 8501:8501 -v datos_app:/app/data TU_USUARIO/hello_docker
```

---

## Preguntas frecuentes

### ¿Qué es Docker?
Docker es una plataforma que permite empaquetar aplicaciones y sus dependencias en contenedores, asegurando que funcionen igual en cualquier entorno.

### ¿Qué es una imagen Docker?
Es una plantilla de solo lectura que contiene todo lo necesario para ejecutar una aplicación (código, dependencias, configuración).

### ¿Qué es un contenedor Docker?
Es una instancia en ejecución de una imagen Docker. Es ligero, aislado y portable.

### ¿Por qué necesito etiquetar mi imagen antes de subirla?
Docker Hub organiza las imágenes por usuario. Al etiquetar con `usuario/nombre`, Docker sabe a qué cuenta pertenece la imagen.

### ¿Puedo tener varias versiones de la misma imagen?
Sí, puedes usar tags adicionales:
```bash
docker tag hello_docker gema/hello_docker:v1.0
docker tag hello_docker gema/hello_docker:latest
```

### ¿Por qué usar Docker?
- Mismo entorno en desarrollo, testing y producción
- Fácil distribución de aplicaciones
- Aislamiento entre aplicaciones
- Escalabilidad y eficiencia

---

## Recursos adicionales

- [Documentación oficial de Docker](https://docs.docker.com/)
- [Documentación de Streamlit](https://docs.streamlit.io/)
- [Docker Hub](https://hub.docker.com/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

## Consejos para principiantes

1. **¿Error al construir?** → Asegúrate de estar en la carpeta correcta (donde está el Dockerfile)
2. **¿Puerto ocupado?** → Cambia el puerto: `docker run -p 8502:8501 hello_docker`
3. **¿Cambios no se reflejan?** → Reconstruye la imagen: `docker build -t hello_docker .`
4. **¿Contenedor no se detiene?** → Usa `Ctrl+C` o `docker stop <container_id>`
5. **¿Olvidaste tu container_id?** → Usa `docker ps` para verlo

---

## ¡Enhorabuena!

Si has llegado hasta aquí, ya sabes cómo dockerizar una aplicación Python. Este conocimiento es fundamental en el desarrollo de software moderno.

---

<div align="center">

**Proyecto realizado por Gema Yébenes con para aprender Docker**

![Docker Logo](https://www.docker.com/wp-content/uploads/2022/03/Moby-logo.png)

</div>