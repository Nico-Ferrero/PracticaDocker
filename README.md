# Practica Docker

El objetivo de esta practica es poner en marcha nuestra aplicacion web utilizando contenedores Docker. Todo se orquesta mediante un unico archivo `docker-compose.yml`.

## Estructura del Proyecto

- **MySQL**: La base de datos.
- **Backend**: La API en PHP.
- **Frontend**: La web en Vue.js.
- **phpMyAdmin**: Para ver la base de datos visualmente.

### MySQL
- **Contenedor**: `mysql_contenidor`
- **Datos**: Guardamos los datos en un volumen `mysql_data` para que no se borren al apagar.
- **Inicio**: Al arrancar, ejecuta automaticamente el script de la carpeta `mysql-init` para crear las tablas.
- **Credenciales**: Configuramos usuario `usuario_db` y contraseña `password_db` para que el backend pueda entrar.

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

### Backend
- **Contenedor**: `backend_contenidor`
- **Puerto**: 8000
- **Nota**: Espera a que la base de datos este lista antes de arrancar (usando `wait-for-it.sh`).

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

### Frontend
- **Contenedor**: `frontend_contenidor`
- **Puerto**: 8080
- **Iniciar**: Instala las dependencias (`npm install`) y arranca el servidor de desarrollo (`npm run serve`).

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

### phpMyAdmin
Desde aqui podremos gestionar la base de datos.
- **Contenedor**: `adminMySQL_contenidor`
- **Puerto**: 8081
- **Conexion**: Se conecta automaticament al servidor de `MySql`.

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

## Iniciar el proyecto

Para iniciar todo ejecuta este comando:

```bash
docker-compose up -d --build
```

Esto descargara, buildeara y iniciara todo.

## Parar el proyecto

Para apagar y borrar los contenedores escribe esto:

```bash
docker-compose down
```

## Acceso a los servidores

Una vez iniciado, entra aqui:

- **Frontend**: [http://localhost:8080](http://localhost:8080)
- **Backend**: [http://localhost:8000/api/items](http://localhost:8000/api/items)
- **Base de Datos**: [http://localhost:8081](http://localhost:8081)
    - Usuario: `usuario_db`
    - Contraseña: `password_db`
