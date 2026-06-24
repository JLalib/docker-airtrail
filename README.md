# 🛫  AirTrail Docker - Rastreador de Vuelos Autohospedado

**AirTrail Docker** es la implementación en contenedores de **AirTrail**, un **rastreador de vuelos personal y open source** que permite visualizar tus vuelos en un mapa interactivo con estad
ísticas detalladas. Diseñado para viajeros frecuentes, enthusiasts de aviación y equipos que buscan **control total de sus datos**. Autohospedable con Docker, multi-usuario y con soporte para
 importar datos desde servicios como FlightRadar24, TripIt o Flighty.

---

## ✨  Características principales

- **Mapa interactivo global**: Visualiza todas tus rutas de vuelo con detalles de aeropuertos, aerolíneas y fechas.
- **Estadísticas avanzadas**: Kilómetros volados, horas en el aire, aeropuertos visitados y desgloses por aerolínea o aeronave.
- **Importación de datos**: Conecta con MyFlightRadar24, JetLog, TripIt, Flighty, App in the Air y más.
- **Multi-usuario**: Soporte para varios usuarios con autenticación local o OAuth (GitHub, Google, etc.).
- **Tema claro/oscuro**: Interfaz adaptable a tus preferencias.
- **Responsive**: Funciona en desktop, tablet y móvil.
- **API REST**: Integración con aplicaciones externas o scripts.
- **Autohospedado**: Tus datos permanecen en tu servidor, sin tracking ni publicidad.

---

## ⚠️  Requisitos previos

- **Docker** y **Docker Compose** (versión 2.x o superior).
- **1-2 GB RAM** (para SvelteKit + PostgreSQL).
- **5 GB mínimo de almacenamiento** (para la base de datos y datos de vuelos).
- **Puerto 3000** disponible (configurable).
- **Navegador moderno** (Chrome, Firefox, Edge, Safari).

---

## ⚙️  Configuración con Docker Compose

### 1. Clona este repositorio y accede al directorio:
```bash
git clone https://github.com/JLalib/docker-airtrail.git
cd airtrail-docker
```

### 2. Archivo `docker-compose.yml`
Crea el archivo `docker-compose.yml` con la siguiente configuración:

```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    container_name: airtrail-db
    environment:
      POSTGRES_USER: airtrail
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: airtrail
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U airtrail"]
      interval: 5s
      timeout: 5s
      retries: 5

  app:
    image: johanohly/airtrail:latest
    container_name: airtrail-app
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: ${DATABASE_URL}
      ORIGIN: ${ORIGIN}
      ORIGINS: ${ORIGINS}
    ports:
      - "3000:3000"
    volumes:
      - ./config:/app/config
    restart: unless-stopped
```

### 3. Configura las variables de entorno
Crea un archivo `.env` basado en el ejemplo anterior y ajusta los valores:

```env
# URL base de la aplicación (ej: localhost, dominio o IP)
ORIGIN=http://localhost:3000
ORIGINS=http://localhost:3000,https://tudominio.com

# Configuración de PostgreSQL
DB_PASSWORD=tu_contraseña_segura
DATABASE_URL=postgres://airtrail:tu_contraseña_segura@db:5432/airtrail
```

> **⚠️  Importante**:
> - Usa una **contraseña segura** para `DB_PASSWORD` (solo caracteres alfanuméricos).
> - Si usas un dominio, configura HTTPS (ver sección [HTTPS con Caddy](#https-con-caddy)).

### 4. Despliega con Docker Compose
```bash
docker compose up -d
```

### 5. Verifica que los contenedores estén corriendo:
```bash
docker compose ps
```

### 6. Accede a la aplicación
Abre en tu navegador:
🔗  [http://localhost:3000](http://localhost:3000)

---

## 🛠️  Primeros pasos

### 1. Crea tu cuenta de administrador
- Regístrate en la interfaz web con un nombre de usuario y contraseña.
- La primera cuenta creada tendrá permisos de administrador.

### 2. Importa tus vuelos
Ve a **Settings → Import** y elige uno de los servicios compatibles (MyFlightRadar24, TripIt, etc.). Sigue los pasos para autenticar y sincronizar tus datos.

### 3. Añade vuelos manualmente
Si no usas servicios externos, puedes añadir vuelos uno a uno en la sección **Add Flight** con detalles como:
- Aeropuertos de origen/destino.
- Fecha y hora del vuelo.
- Aerolínea y número de vuelo.

### 4. Explora el mapa y estadísticas
- **Mapa interactivo**: Filtra por fechas, aerolíneas o rutas.
- **Estadísticas**: Revisa tus kilómetros totales, horas en el aire y gráficos anuales.

---

## 🔒  Seguridad y recomendaciones

### 1. Cambia las credenciales por defecto
- Actualiza `DB_PASSWORD` en `.env` antes de desplegar en producción.
- Usa contraseñas robustas para cuentas de usuario.

### 2. Configura HTTPS (producción)
Usa **Caddy** o **Nginx** como reverse proxy para habilitar HTTPS. Ejemplo con Caddy:

**`Caddyfile`** (guarda en el mismo directorio que `docker-compose.yml`):
```
airtrail.tudominio.com {
    reverse_proxy localhost:3000
}
```

Actualiza `ORIGINS` en `.env` para incluir tu dominio:
```env
ORIGINS=https://airtrail.tudominio.com,http://localhost:3000
```

Reinicia los servicios:
```bash
docker compose restart
```

### 3. Copias de seguridad
- **Base de datos**: Haz backup periódico de PostgreSQL:
  ```bash
  docker exec airtrail-db pg_dump -U airtrail -d airtrail > airtrail-backup-$(date +%Y%m%d).sql
  ```
- **Volúmenes**: Los datos persistentes están en `./config` y `./postgres-data`. Usa herramientas como `rsync` para backups externos.

---

## 📂  Estructura del proyecto

```
./
├── docker-compose.yml    # Definición de servicios (AirTrail + PostgreSQL)
├── .env                  # Variables de entorno
├── config/               # Volumen persistente (datos de la app)
├── postgres-data/        # Volumen de PostgreSQL
└── README-AirTrail-Docker.md  # Este archivo
```

---

## 🔄  Actualización y mantenimiento

### Actualizar AirTrail
1. Detén los contenedores:
   ```bash
   docker compose down
   ```
2. Descarga la última versión:
   ```bash
   docker compose pull
   ```
3. Reinicia:
   ```bash
   docker compose up -d
   ```

### Comandos útiles
| Comando                          | Descripción                          |
|----------------------------------|--------------------------------------|
| `docker compose logs -f`         | Ver logs en tiempo real.             |
| `docker compose restart`         | Reiniciar servicios.                 |
| `docker compose ps`              | Estado de los contenedores.          |
| `docker exec airtrail-db du -sh /var/lib/postgresql/data` | Tamaño de la base de datos. |

---

## 📊  Comparativa con alternativas

| Característica               | AirTrail Docker    | MyFlightRadar24 | Flighty       | JetLog          |
|------------------------------|--------------------|-----------------|---------------|-----------------|
| **Autohospedado**            | ✅  Sí              | ❌  No          | ❌  No         | ❌  No          |
| **Multi-usuario**            | ✅  Sí              | ❌  No          | ❌  No         | ✅  Sí          |
| **Mapa interactivo**         | ✅  Sí              | ✅  Sí          | ❌  No         | ❌  No          |
| **Importación de datos**     | ✅  (Varios)        | ✅  (Limitado)  | ✅  Sí         | ✅  Sí          |
| **API REST**                 | ✅  Sí              | ❌  No          | ❌  No         | ❌  No          |
| **Control de datos**         | ✅  Total           | ❌  Limitado    | ❌  Limitado   | ❌  Limitado    |

---

## 📚  Referencias

- [Repositorio oficial](https://github.com/johanohly/AirTrail)
- [Documentación](https://johanohly.github.io/AirTrail/)
- [Imagen Docker](https://hub.docker.com/r/johanohly/airtrail)
- [Demo en vivo](https://airtrail.johan.ohly.dk/)

---
