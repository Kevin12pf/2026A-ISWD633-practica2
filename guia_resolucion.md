# Guía de Resolución para la Práctica 2 - Docker

A continuación encontrarás el análisis y la solución paso a paso para completar los puntos de los archivos del 1 al 4. Esta guía te proporciona los comandos exactos y las respuestas requeridas para que puedas copiar, ejecutar y generar las capturas de pantalla para tu práctica.

---

## 1. Variables de Entorno (`1-variables-de-entorno.md`)

**¿Qué son las variables de entorno?**
Son valores dinámicos que afectan el comportamiento de los procesos o programas que se ejecutan en un sistema operativo o contenedor. En Docker, nos permiten pasar configuraciones, credenciales o personalizaciones a un contenedor en el momento de su creación, sin tener que alterar el código original o reconstruir la imagen.

**Crear un contenedor de nginx con variables de entorno (username y role):**

```bash
docker run -d --name mi-nginx-env -e username=Kevin -e role=admin nginx:alpine
```

_(Para comprobar las variables y tomar la captura, puedes usar el comando: `docker exec mi-nginx-env env | grep -E 'username|role'`)_

**Crear un contenedor con la imagen de mysql, mapear todos los puertos:**

```bash
docker run -d --name mi-mysql -P mysql
```

**¿El contenedor se está ejecutando?**
No. Si revisas con `docker ps -a`, verás que el contenedor se detuvo (estado `Exited`).

**Identificar el problema:**
El contenedor falla al iniciar porque la imagen de MySQL exige obligatoriamente que se configure una variable de entorno para establecer la contraseña del administrador. Al no pasarle la variable `MYSQL_ROOT_PASSWORD` (ni ninguna otra de las permitidas para el inicio), se produce un error y el contenedor se detiene.

**¿Qué bases de datos existen en el contenedor creado?**
Para poder verlo, primero debemos crear el contenedor correctamente e ingresar:

```bash
docker run -d --name mi-mysql-bien -e MYSQL_ROOT_PASSWORD=secreta -P mysql
docker exec -it mi-mysql-bien mysql -uroot -psecreta -e "SHOW DATABASES;"
```

_(Bases de datos por defecto: `information_schema`, `mysql`, `performance_schema`, `sys`)_

---

## 2. Ejercicio Postgres y PgAdmin (`2-ejercicio.md`)

**Crear contenedor de Postgres sin exponer puertos:**

```bash
docker run -d --name mi-postgres -e POSTGRES_PASSWORD=secreta postgres:alpine
```

**Crear un cliente de Postgres (PgAdmin):**

```bash
docker run -d --name mi-pgadmin -p 8081:80 -e PGADMIN_DEFAULT_EMAIL=admin@admin.com -e PGADMIN_DEFAULT_PASSWORD=admin dpage/pgadmin4
```

**Valores del esquema:**

- **a:** `5432` (Puerto origen/interno por el cual se comunica y escucha Postgres)
- **b:** `8081` (Puerto host/mapeado de nuestra máquina para acceder a PgAdmin)
- **c:** `80` (Puerto de escucha interno del contenedor web PgAdmin)

**(Para la captura de Login):** Entra a `http://localhost:8081` en tu navegador y usa `admin@admin.com` y `admin`.
En la configuración del Servidor en PgAdmin (Add New Server), en la pestaña "Connection", usa como Host la IP local del contenedor Postgres. Para saber la IP de Postgres ejecuta: `docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mi-postgres`. (También introduce el puerto 5432, usuario `postgres` y contraseña `secreta`).

**(Para crear la base de datos `info`, tabla `personas` y registros "desde el cliente" PgAdmin)**:
1. Conéctate a tu servidor en PgAdmin.
2. Puedes hacerlo de forma gráfica: haz clic derecho sobre el servidor -> **Create** -> **Database...** -> Nómbrala `info` y guarda.
   Alternativamente, abre **Query Tool** en la base de datos por defecto (`postgres`) y ejecuta:
   ```sql
   CREATE DATABASE info;
   ```
3. Ahora, selecciona (despliega) la base de datos `info` recién creada en el menú lateral izquierdo, haz clic derecho sobre ella y selecciona **Query Tool**.
4. Pega el siguiente código para crear la tabla e insertar los registros, luego presiona el botón de *"Play"* (Execute/Refresh) en la barra superior:
```sql
CREATE TABLE personas (id SERIAL PRIMARY KEY, nombre VARCHAR(100));
INSERT INTO personas (nombre) VALUES ('Kevin'), ('Maria');
```
*(Toma captura de este proceso donde se evidencie su ejecución exitosa en la pantalla del cliente PgAdmin)*.

## Desde el servidor postgresl
**Conectarse por consola al servidor interno de Postgres y a la BD info:**

```bash
docker exec -it mi-postgres psql -U postgres
\c info
```

**Realizar un select \* from personas:**

```sql
SELECT * FROM personas;
```

_(Haz una captura del resultado en consola)._

---

## 3. Redes (`3-redes.md`)

_Nota: Asumiendo que el esquema muestra al menos dos redes puente o contenedores conectados entre sí en redes separadas. Aquí tienes los comandos base según cómo suele ser la topología del esquema de este ejercicio._

**Comandos para crear las redes:**

```bash
docker network create mi-red-1 -d bridge
docker network create mi-red-2 -d bridge
```

**Comandos para crear los contenedores (usando nginx:alpine) según un esquema común:**

```bash
docker run -d --name nginx1 --network mi-red-1 nginx:alpine
docker run -d --name nginx2 --network mi-red-1 nginx:alpine
# Si nginx2 también estuviera en red-2:
docker network connect mi-red-2 nginx2
docker run -d --name nginx3 --network mi-red-2 nginx:alpine
```

**Capturar Redes:**
Usa `docker network ls` para la captura inicial.
Usa `docker inspect nginx1`, `docker inspect nginx2`, etc., y ve hacia el apartado `Networks` para demostrar las vinculaciones y tomar esas capturas.

---

## 4. Ejercicio WordPress + MySQL (`4-ejercicio.md`)

**Crear la red:**

```bash
docker network create red-wordpress -d bridge
```

**Crear el contenedor MySQL:**

```bash
docker run -d --name bd-wordpress --network red-wordpress \
  -e MYSQL_ROOT_PASSWORD=contrasena_segura \
  -e MYSQL_DATABASE=wordpress_db \
  -e MYSQL_USER=wp_user \
  -e MYSQL_PASSWORD=wp_pass \
  mysql:8
```

**Crear el contenedor WordPress:**

```bash
docker run -d --name my-wordpress -p 9300:80 --network red-wordpress \
  -e WORDPRESS_DB_HOST=bd-wordpress:3306 \
  -e WORDPRESS_DB_USER=wp_user \
  -e WORDPRESS_DB_PASSWORD=wp_pass \
  -e WORDPRESS_DB_NAME=wordpress_db \
  wordpress
```

**Puerto "a" del esquema:**
De acuerdo con el desarrollo, el puerto "a" es **`9300`**.

**Instalación:**
Ingresa a `http://localhost:9300/`, instala Wordpress, cambia un tema y haz una publicación de prueba _(Recuerda tomar captura)_.

**Eliminar contenedor WordPress:**

```bash
docker rm -f my-wordpress
```

**Crear nuevamente el contenedor de WordPress:**
Vuelve a ejecutar exactamente el mismo comando de creación que utilizaste para `my-wordpress` hace unos momentos.

**¿Qué ha sucedido, qué puede observar?**
A pesar de haber eliminado el contenedor del sitio web de WordPress, al crearlo nuevamente y apuntarlo hacia la misma base de datos MySQL preexistente, **el sitio, el tema, la configuración y las publicaciones vuelven a estar allí sin necesidad de configuración adicional**.
Esto demuestra que el estado y los datos persistentes de la aplicación no estaban guardados en el contenedor web (que es efímero/"stateless"), sino centralizados en el servicio de base de datos MySQL, el cual nunca fue eliminado.
