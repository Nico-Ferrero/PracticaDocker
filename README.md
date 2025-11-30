# Práctica Docker

El objetivo de esta práctica es poner en marcha nuestra aplicación web utilizando contenedores Docker. Todo se orquesta mediante un único archivo `docker-compose.yml`.

## Estructura del Proyecto

El sistema se compone de 4 partes (servicios) que funcionan juntas:

- **MySQL**: La base de datos.
- **Backend**: La API en PHP.
- **Frontend**: La web en Vue.js.
- **phpMyAdmin**: Para ver la base de datos visualmente.

### Servicio MySQL
Usamos la imagen oficial de MySQL 8.0.
- **Contenedor**: `mysql_contenidor`
- **Datos**: Guardamos los datos en un volumen (`mysql_data`) para que no se borren al apagar.
- **Inicio**: Al arrancar, ejecuta automáticamente el script de la carpeta `mysql-init` para crear las tablas.
- **Credenciales**: Configuramos usuario (`usuario_db`) y contraseña (`password_db`) para que el backend pueda entrar.

```yaml
  mysql:
    image: mysql:8.0
    container_name: mysql_contenidor
    environment:
      MYSQL_ROOT_PASSWORD: password_root
      MYSQL_DATABASE: elementos
      MYSQL_USER: usuario_db
      MYSQL_PASSWORD: password_db
    volumes:
      - ./mysql-init:/docker-entrypoint-initdb.d
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"
```

### Servicio Backend
Es nuestra API hecha en PHP.
- **Contenedor**: `backend_contenidor`
- **Puerto**: 8000
- **Conexión**: Espera a que la base de datos esté lista antes de arrancar (usando `wait-for-it.sh`).
- **Código**: Copiamos nuestros archivos PHP y activamos las extensiones necesarias para conectar con MySQL.

**Dockerfile del Backend:**
```dockerfile
FROM alpine:3.18 AS builder
WORKDIR /app
COPY . .

FROM php:8.1-cli-alpine

RUN apk add --no-cache bash \
    && docker-php-ext-install pdo pdo_mysql

WORKDIR /var/www/html

COPY --from=builder /app .

RUN chmod +x wait-for-it.sh

EXPOSE 8000
```

### Servicio Frontend
Es nuestra página web hecha con Vue.js.
- **Contenedor**: `frontend_contenidor`
- **Puerto**: 8080
- **Arranque**: Instala las dependencias (`npm install`) y arranca el servidor de desarrollo (`npm run serve`).

**Dockerfile del Frontend:**
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm install

FROM node:18-alpine
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

EXPOSE 8080

CMD ["npm", "run", "serve"]
```

### Servicio phpMyAdmin
Herramienta para gestionar la base de datos desde el navegador.
- **Contenedor**: `adminMySQL_contenidor`
- **Puerto**: 8081
- **Conexión**: Se conecta automáticamente al servicio de `mysql`.

```yaml
  phpmyadmin:
    image: phpmyadmin/phpmyadmin:5.2
    container_name: adminMySQL_contenidor
    depends_on:
      - mysql
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql_contenidor
```

---

## Cómo ejecutar el proyecto

Para poner todo en marcha, abre la terminal en la carpeta del proyecto y escribe:

```bash
docker-compose up -d --build
```

Esto descargará, construirá y arrancará todo.

## Cómo parar el proyecto

Para apagar y borrar los contenedores, escribe:

```bash
docker-compose down
```

## Acceso a los servicios

Una vez arrancado, puedes entrar aquí:

- **Web (Frontend)**: [http://localhost:8080](http://localhost:8080)
- **API (Backend)**: [http://localhost:8000/api/items](http://localhost:8000/api/items)
- **Base de Datos (phpMyAdmin)**: [http://localhost:8081](http://localhost:8081)
    - Usuario: `usuario_db`
    - Contraseña: `password_db`
