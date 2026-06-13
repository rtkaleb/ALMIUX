🛒 Abarrotes Almiux
Proyecto final del Bootcamp Generation México — Cohorte 66
Equipo 404 Team Not Found

Abarrotes Almiux es una tienda de abarrotes en línea con entrega a domicilio. Permite a los clientes explorar productos, gestionar su carrito y realizar pedidos, mientras que los administradores pueden gestionar el catálogo y los pedidos desde un panel dedicado.

Inicio Almiux

📋 Tabla de contenidos
Tecnologías
Arquitectura
Base de datos
Backend
Frontend
Despliegue en AWS
Equipo
Tecnologías
Capa	Tecnología	Versión
Backend	Java + Spring Boot	17 / 3.x
ORM	Hibernate + Spring Data JPA	—
Seguridad	Spring Security + BCrypt	—
Base de datos	MySQL	8+
Frontend	HTML, CSS, JavaScript Vanilla	—
Servidor web	Nginx (reverse proxy)	—
Infraestructura	AWS EC2 (Ubuntu)	—
Build	Gradle	8+
Gestión de proyecto	Jira	—
Arquitectura
Usuario
  │
  ▼
Nginx (puerto 80)  ──reverse proxy──▶  Spring Boot (puerto 8080)
                                              │
                                    ┌─────────┴─────────┐
                                 Archivos          API REST
                                 estáticos       /api/v1.0/...
                                 (HTML/CSS/JS)
                                              │
                                              ▼
                                         MySQL 8
Nginx actúa como reverse proxy: recibe las peticiones en el puerto 80 y las redirige al servidor Spring Boot en el puerto 8080. Spring Boot sirve tanto el frontend estático como la API REST desde el mismo proceso.

Base de datos
Diagrama Entidad-Relación
Diagrama ER

Tablas
Tabla	Descripción	Relaciones
usuarios	Clientes y administradores	1:N con pedidos
categorias	Grupos de productos	1:N con productos
productos	Catálogo de la tienda	N:1 con categorias
pedidos	Órdenes de compra	N:1 con usuarios, 1:N con detalle_pedido
detalle_pedido	Líneas de cada pedido	N:1 con pedidos y productos
Configuración local
1. Crear la base de datos:

CREATE DATABASE almiux_db;
2. Copiar la plantilla de configuración:

# Mac / Linux
cp src/main/resources/application.properties.example src/main/resources/application.properties

# Windows
copy src\main\resources\application.properties.example src\main\resources\application.properties
3. Editar con tus credenciales MySQL:

spring.datasource.username=TU_USUARIO
spring.datasource.password=TU_CONTRASEÑA
application.properties está en .gitignore y nunca se sube al repositorio.

Backend
API REST desarrollada con Spring Boot 3.

Spring Boot

Levantar el servidor
./gradlew bootRun
El servidor inicia en http://localhost:8080. Hibernate crea las tablas automáticamente.

Endpoints — Base URL: /api/v1.0
Usuarios /users
Método	Endpoint	Descripción
GET	/users	Todos los usuarios
GET	/users/{id}	Usuario por ID
GET	/users/email?email=	Usuario por email
POST	/users	Crear usuario
PUT	/users/{id}	Actualizar usuario
DELETE	/users/{id}	Eliminar usuario
Categorías /category
Método	Endpoint	Descripción
GET	/category/categories	Todas las categorías
GET	/category/{id}	Categoría por ID
POST	/category	Crear categoría
PUT	/category/{id}	Actualizar categoría
DELETE	/category/{id}	Eliminar categoría
Productos /products
Método	Endpoint	Descripción
GET	/products	Todos los productos
GET	/products/{id}	Producto por ID
POST	/products	Crear producto
PUT	/products/{id}	Actualizar producto
DELETE	/products/{id}	Eliminar producto
Pedidos /orders
Método	Endpoint	Descripción
GET	/orders	Todos los pedidos
GET	/orders/{id}	Pedido por ID
POST	/orders	Crear pedido
PUT	/orders/{id}	Actualizar pedido
DELETE	/orders/{id}	Eliminar pedido
Estatus válidos: PENDIENTE · EN_PROCESO · ENVIADO · ENTREGADO · CANCELADO

Detalles de pedido
Método	Endpoint	Descripción
GET	/orders/{id}/detalles	Detalles de un pedido
POST	/orders/{id}/detalles	Agregar detalle
PUT	/detalles/{id}	Actualizar detalle
DELETE	/detalles/{id}	Eliminar detalle
Roles
Rol	Acceso
CLIENTE	Explorar productos, realizar pedidos, ver su historial
ADMIN	Panel de administración completo
Estructura del proyecto
src/main/java/org/generation/ALMIUX/
├── config/        # Spring Security, BCrypt
├── controller/    # Endpoints REST
├── exceptions/    # Manejo centralizado de errores
├── model/         # Entidades JPA
├── repository/    # Interfaces de acceso a BD
└── service/       # Lógica de negocio
Frontend
Desarrollado con HTML, CSS y JavaScript Vanilla. Se sirve como archivos estáticos desde Spring Boot (src/main/resources/static/).

Páginas
Página	Ruta	Descripción
Inicio	/index.html	Landing page principal
Productos	/productos.html	Catálogo con filtros y búsqueda
Carrito	/carrito.html	Drawer del carrito de compras
Nosotros	/nosotros.html	Información del equipo
Promociones	/promociones.html	Ofertas especiales
Login	/usuario/login.html	Inicio de sesión
Registro	/usuario/registro.html	Crear cuenta
Mi cuenta	/usuario/index.html	Perfil del cliente
Mis pedidos	/usuario/mis-pedidos.html	Historial de pedidos
Panel Admin	/admin/index.html	Gestión de productos y pedidos
Estructura del frontend
static/
├── index.html              # Página de inicio
├── styles.css              # Estilos globales
├── js/
│   ├── api.js              # Llamadas a la API REST
│   ├── auth.js             # Autenticación y sesión
│   ├── navbar.js           # Comportamiento del navbar
│   ├── carrito.js          # Lógica del carrito
│   ├── login.js            # Flujo de login
│   ├── productos.js        # Catálogo y filtros
│   └── utils.js            # Utilidades compartidas
├── usuario/                # Páginas del cliente
├── admin/                  # Panel de administración
└── utils/
    ├── navbar/             # Fragmento del navbar
    └── footer/             # Fragmento del footer
Despliegue en AWS
El proyecto está desplegado en una instancia AWS EC2 con Ubuntu.

Stack de producción
EC2 — Ubuntu Server
Nginx — Reverse proxy en el puerto 80
Spring Boot — Corre como servicio systemd en el puerto 8080
MySQL — Base de datos en Hostinger
Configuración de Nginx
server {
    listen 80;
    server_name 18.188.12.224;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
Servicio systemd
El backend corre como servicio con reinicio automático (Restart=always), configurado en /etc/systemd/system/almiux.service.

Comandos de administración
# Ver estado del servicio
sudo systemctl status almiux

# Reiniciar el backend
sudo systemctl restart almiux

# Recargar Nginx
sudo service nginx restart
Actualizar el servidor
# 1. Construir el nuevo JAR (local)
./gradlew bootJar

# 2. Detener el servicio y borrar el JAR anterior (VM)
sudo systemctl stop almiux && rm ~/src/almiux.jar

# 3. Subir el nuevo JAR (local)
scp -i 404notfound.pem build/libs/*.jar ubuntu@18.188.12.224:~/src/almiux.jar

# 4. Reiniciar el servicio (VM)
sudo systemctl start almiux
Gestión del proyecto
Tablero Jira

El proyecto se gestionó con metodología Scrum usando Jira para el seguimiento de tareas y sprints.

Equipo
404 Team Not Found — Generation México Bootcamp · Cohorte 66

Integrante	Rol
Kaleb Torres	Full Stack Developer · Scrum Master
Danna Remigio	Frontend Developer
Arturo Ramírez	Frontend Developer
Yarilis Hernández	Frontend Developer
Zared Ortiz	Backend Developer
Noé Hernández	QA Tester
Diego Quiñónez	Backend Developer
<div align="center">
🛒 Abarrotes Almiux

Proyecto académico desarrollado para el Bootcamp de Desarrollo Web Full Stack
Generation México · Cohorte 66 · 2026

Hecho en México con ❤️

</div>