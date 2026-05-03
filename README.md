# MPO: Proyecto Final Integrador

## Descripción

Este proyecto es el Proyecto Final Integrador del módulo profesional optativo "Desarrollo y despliegue en entornos colaborativos – Computación en la nube", perteneciente al Ciclo Formativo de Grado Superior en Administración de Sistemas Informáticos en Red (ASIR).

La aplicación consiste en un servidor web nginx que sirve una página web estática desarrollada originalmente en la Tarea 1 del módulo (MPO01) como proyecto de aprendizaje de Git y GitHub. En el presente proyecto, dicha página web se conteneriza mediante Docker y se orquesta con Docker Compose, de forma que pueda desplegarse de manera reproducible en cualquier sistema.

El proyecto integra las competencias adquiridas a lo largo del módulo: control de versiones con Git y GitHub (MPO01), contenerización y orquestación con Docker y Docker Compose (MPO02), y computación en la nube (MPO03).

---

## Estructura del proyecto

```
MPO-PFI/
├── Dockerfile            # Definición de la imagen Docker
├── docker-compose.yml    # Orquestación del servicio
├── index.html            # Página web estática servida por nginx
├── .gitignore            # Archivos excluidos del repositorio
└── README.md             # Documentación del proyecto
```

---

## Tecnologías utilizadas

| Tecnología | Descripción |
|---|---|
| **HTML5** | Página web estática |
| **nginx:stable** | Servidor web dentro del contenedor |
| **Docker** | Contenerización de la aplicación |
| **Docker Compose** | Orquestación del servicio |
| **Git / GitHub** | Control de versiones y repositorio remoto |

---

## Requisitos previos

- Docker instalado y en ejecución
- Docker Compose v2 o superior
- Git instalado
- Conexión a internet para la primera descarga de la imagen nginx

---

## Instalación y despliegue

### 1. Clonar el repositorio

```bash
git clone git@github.com:b8m2/MPO-PFI.git
cd MPO-PFI
```

### 2. Construir la imagen

```bash
docker build -t mpo-pfi:v1 .
```

### 3. Levantar los servicios con Docker Compose

```bash
docker compose up -d --build
```

### 4. Acceder a la aplicación

La aplicación estará disponible en:

```
http://localhost:8080
```

### 5. Verificar que el servicio está activo

```bash
docker compose ps
```

El campo STATUS debe mostrar `Up (healthy)`.

### 6. Parar los servicios

```bash
docker compose down
```

---

## Explicación del Dockerfile

```dockerfile
FROM nginx:stable
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

- **FROM nginx:stable**: imagen base oficial de nginx en su rama estable. Se elige `stable` en lugar de `latest` para garantizar estabilidad, y en lugar de `nginx:alpine` porque la imagen `stable` incluye `curl` de forma nativa, necesario para el healthcheck.
- **COPY**: copia el archivo `index.html` al directorio donde nginx sirve contenido estático por defecto.
- **EXPOSE 80**: documenta que el contenedor escucha en el puerto 80 (HTTP).
- **HEALTHCHECK**: comprueba periódicamente que nginx responde correctamente. Los valores de `interval` y `start_period` se han ajustado a 30s y 10s respectivamente, más adecuados para un servidor web estático que los valores por defecto de la documentación oficial.

---

## Explicación del docker-compose.yml

```yaml
services:
  web:
    build: .
    image: mpo-pfi:v1
    container_name: mpo-pfi-web
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html:ro
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
      start_interval: 5s
```

- **build**: construye la imagen a partir del Dockerfile del directorio actual.
- **image**: nombre asignado a la imagen construida.
- **container_name**: nombre fijo del contenedor, evitando el nombre automático generado por Compose.
- **ports**: mapea el puerto 8080 del host al puerto 80 del contenedor.
- **volumes**: bind-mount que monta el `index.html` local directamente en el contenedor en modo solo lectura (`:ro`). Permite actualizar el contenido web sin reconstruir la imagen.
- **restart**: política de reinicio automático ante caídas inesperadas.
- **healthcheck**: control de salud del servicio. El parámetro `start_interval: 5s` (disponible desde Docker Compose v2.20.2) acelera los checks durante el arranque inicial del contenedor.

---

## Posibles problemas y soluciones

**Puerto 8080 ocupado**

Si al ejecutar `docker compose up` aparece el error `address already in use`, significa que otro proceso está usando el puerto 8080 en el host. Solución: identificar y parar el proceso con `sudo lsof -i :8080`, o cambiar el puerto en `docker-compose.yml` (por ejemplo, `"8081:80"`).

**Permiso denegado al ejecutar Docker**

Si aparece `permission denied while trying to connect to the Docker daemon socket`, el usuario no pertenece al grupo docker. Solución (documentada en la Tarea 2 del módulo):

```bash
sudo usermod -aG docker $USER
newgrp docker
```

**La imagen nginx:stable no se descarga**

Verificar la conexión a internet y que Docker está en ejecución con `docker info`.

---

## Organización del proyecto y flujo de trabajo con ramas

El proyecto sigue el flujo de trabajo GitHub Flow, con una rama `main` estable y ramas de desarrollo por funcionalidad:

| Rama | Descripción |
|---|---|
| `main` | Rama principal estable |
| `feature/web-content` | Incorporación del contenido web (index.html) |
| `feature/docker` | Creación del Dockerfile |
| `feature/compose` | Creación del docker-compose.yml |
| `feature/docs` | Documentación del proyecto (README.md) |

Cada rama se fusiona a `main` mediante un Pull Request, garantizando que la rama principal siempre contiene una versión funcional del proyecto.

---

## Diagrama de arquitectura

```
┌─────────────────────────────────────────────┐
│              Host (Ubuntu 22.04)            │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │         Docker Compose               │   │
│  │                                      │   │
│  │  ┌────────────────────────────────┐  │   │
│  │  │  Contenedor: mpo-pfi-web       │  │   │
│  │  │  Imagen: nginx:stable          │  │   │
│  │  │  Puerto interno: 80            │  │   │
│  │  │  Volumen bind-mount:           │  │   │
│  │  │  ./index.html →                │  │   │
│  │  │  /usr/share/nginx/html/        │  │   │
│  │  └────────────┬───────────────────┘  │   │
│  └───────────────│──────────────────────┘   │
│                  │ puerto 8080:80           │
└──────────────────┼──────────────────────────┘
                   │
        http://localhost:8080
             (navegador / curl)
```

---

## Autor

**Fernando Estela Bravo**
Desarrollo y despliegue en entornos colaborativos - Computación en la nube (ASIR)
[Mi perfil de GitHub](https://github.com/b8m2)
Curso 2025-2026
