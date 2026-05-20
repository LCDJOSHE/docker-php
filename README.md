# INFORME DE PRÁCTICA: DESPLIEGUE DE INFRAESTRUCTURA WEB CONTAINERIZADA CON DOCKER COMPOSE

---

# 1. Título

## Orquestación de una Plataforma CMS WordPress mediante Contenedores Docker: Configuración Integrada de Servidor Web, Motor de Base de Datos Relacional y Administrador Gráfico phpMyAdmin utilizando Docker Compose

---

# 2. Tiempo de Duración

| Parámetro | Detalle |
|---|---|
| Duración estimada | 120 minutos |
| Duración real | ~110 minutos |
| Tipo de actividad | Práctica de laboratorio guiada |

---

# 3. Plataforma de Simulación

Se utilizó Killercoda como entorno de laboratorio virtualizado en la nube, específicamente el escenario denominado **"Port-forwarding in Docker"**. Esta plataforma proporciona un playground preconfigurado con sistema operativo Ubuntu Linux, Docker Engine y Docker Compose ya instalados, lo que permite concentrarse directamente en la construcción de la infraestructura sin necesidad de instalar dependencias previamente.

La elección de este escenario fue estratégica, ya que habilita una pestaña de **Traffic / Port Forwarding** que permite acceder remotamente a los puertos expuestos por los contenedores desde el navegador del usuario.

---

# 4. Fundamentos Teóricos

## 4.1 Contenedores y su Impacto en el Desarrollo Moderno

La virtualización basada en contenedores representa un cambio de paradigma en la manera en que se empaquetan, distribuyen y ejecutan las aplicaciones. A diferencia de las máquinas virtuales convencionales —que requieren un sistema operativo completo por cada instancia—, los contenedores comparten el kernel del sistema operativo anfitrión. Esto se traduce en:

- Menor consumo de recursos (CPU, RAM, almacenamiento).
- Tiempos de arranque casi instantáneos (segundos vs. minutos).
- Portabilidad garantizada: una aplicación empaquetada en un contenedor se ejecuta de forma idéntica en cualquier máquina que tenga Docker instalado, eliminando el clásico problema de *"en mi máquina sí funciona"*.

## 4.2 Docker como Motor de Contenedores

Docker es la plataforma líder del ecosistema de contenedores. Su arquitectura se fundamenta en tres pilares:

| Componente | Función |
|---|---|
| Docker Engine | Daemon que gestiona el ciclo de vida de los contenedores |
| Docker Images | Plantillas de solo lectura que definen el sistema de archivos del contenedor |
| Docker Hub | Registro público de imágenes oficiales y comunitarias |

## 4.3 Docker Compose: Orquestación Declarativa

Cuando una aplicación se compone de múltiples servicios interdependientes (patrón de microservicios), ejecutar cada contenedor manualmente con `docker run` resulta tedioso, propenso a errores y difícil de reproducir. Docker Compose resuelve este problema al permitir definir toda la arquitectura de servicios en un único archivo declarativo en formato YAML (`docker-compose.yml`).

Las ventajas principales de Docker Compose incluyen:

- Definición centralizada: servicios, redes, volúmenes y variables de entorno en un solo archivo.
- Comandos unificados: levantar (`up`) o detener (`down`) toda la infraestructura con una sola instrucción.
- Reproducibilidad: cualquier persona con el archivo YAML puede replicar el mismo entorno exacto.
- Gestión de dependencias: el atributo `depends_on` garantiza el orden correcto de inicio de los servicios.

## 4.4 Componentes de la Arquitectura Implementada

En esta práctica se orquestaron tres microservicios que operan de manera coordinada:

| Servicio | Imagen Docker | Rol en la Arquitectura |
|---|---|---|
| MySQL 8.0 | `mysql:8.0` | Motor de base de datos relacional que almacena toda la información del CMS |
| WordPress | `wordpress:latest` | Sistema de gestión de contenidos (CMS) con PHP y Apache integrados |
| phpMyAdmin | `phpmyadmin:latest` | Herramienta gráfica para administración visual de MySQL |

## 4.5 Conceptos Clave de Networking y Persistencia

### Red Bridge Personalizada
Docker permite crear redes virtuales internas donde los contenedores se comunican entre sí mediante sus nombres de servicio como alias DNS, sin necesidad de conocer direcciones IP internas.

### Volúmenes Persistentes
Por defecto, los datos dentro de un contenedor son efímeros. Al asignar volúmenes con nombre, los datos sobreviven a la destrucción y recreación del contenedor.

---

# 5. Conocimientos Previos Requeridos

Para la correcta ejecución de esta práctica, se requirieron los siguientes conocimientos:

- Sintaxis y reglas de indentación del formato YAML.
- Manejo de terminal Bash y comandos Linux.
- Diferencia entre Docker CLI v1 (`docker-compose`) y Docker CLI v2 (`docker compose`).
- Conceptos de redes virtuales tipo Bridge y mapeo de puertos.
- Fundamentos de bases de datos relacionales MySQL.

---

# 6. Objetivos

## 6.1 Objetivo General

Desplegar una infraestructura web multi-servicio completa mediante un archivo declarativo `docker-compose.yml`, integrando un CMS WordPress, un motor de base de datos MySQL y un panel de administración phpMyAdmin dentro del entorno virtualizado de Killercoda.

## 6.2 Objetivos Específicos

- Redactar un archivo `docker-compose.yml` sintácticamente válido.
- Configurar e integrar los servicios MySQL, WordPress y phpMyAdmin.
- Establecer una red puente personalizada (`red_practica`).
- Asignar volúmenes persistentes para MySQL y WordPress.
- Validar el correcto funcionamiento de cada servicio.
- Resolver incidencias técnicas durante el despliegue.

---

# 7. Equipo y Herramientas Necesarias

| Recurso | Especificación |
|---|---|
| Computador | Navegador moderno (Chrome, Firefox, Edge) |
| Conexión a Internet | Acceso estable a Killercoda y Docker Hub |
| Cuenta Killercoda | Cuenta activa |
| Escenario | Port-forwarding in Docker |

---

# 8. Material de Apoyo

- Documentación oficial de Docker Compose:  
  https://docs.docker.com/compose/compose-file/

- Docker Hub:  
  https://hub.docker.com/

- Guía YAML:  
  https://yaml.org/spec/

---

# 9. Procedimiento Detallado

## Paso 1 — Selección del Escenario y Diagnóstico Inicial

Se ingresó a Killercoda y se seleccionó el escenario **Port-forwarding in Docker**.

Se verificó que Docker estuviera operativo ejecutando:

```bash
docker --version
docker-compose --version
```

---

## Paso 2 — Creación del Archivo docker-compose.yml

Se creó el archivo de configuración utilizando:

```bash
cat << 'EOF' > docker-compose.yml
version: '3.8'

services:

  mysql:
    image: mysql:8.0
    container_name: servidor_mysql
    restart: always

    environment:
      MYSQL_ROOT_PASSWORD: "12345"
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: "12345"

    volumes:
      - datos_web:/var/lib/mysql

    networks:
      - red_practica

  wordpress:
    image: wordpress:latest
    container_name: sitio_wordpress
    restart: always

    ports:
      - "8080:80"

    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: "12345"
      WORDPRESS_DB_NAME: wordpress_db

    volumes:
      - archivos_web:/var/www/html

    networks:
      - red_practica

    depends_on:
      - mysql

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: panel_phpmyadmin
    restart: always

    ports:
      - "8081:80"

    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: "12345"

    networks:
      - red_practica

    depends_on:
      - mysql

networks:
  red_practica:
    driver: bridge

volumes:
  datos_web:
  archivos_web:
EOF
```

### Explicación de las Directivas

| Directiva | Propósito |
|---|---|
| image | Define la imagen Docker |
| container_name | Asigna nombre personalizado |
| restart: always | Reinicio automático |
| ports | Mapeo de puertos |
| environment | Variables de entorno |
| volumes | Persistencia de datos |
| networks | Comunicación entre contenedores |
| depends_on | Dependencias entre servicios |

---

## Paso 3 — Verificación del Archivo YAML

Se validó el contenido mediante:

```bash
cat docker-compose.yml
```

Se revisó la correcta indentación utilizando únicamente espacios.

---

## Paso 4 — Despliegue de la Infraestructura

Se ejecutó:

```bash
docker-compose up -d
```

Este comando:

- Descargó las imágenes necesarias.
- Creó la red `red_practica`.
- Creó los volúmenes persistentes.
- Levantó los contenedores MySQL, WordPress y phpMyAdmin.

---

## Incidencias Técnicas Detectadas y Resueltas

### Incidencia 1 — Docker Compose V1 vs V2

El comando moderno:

```bash
docker compose up -d
```

presentó errores debido a incompatibilidad con Docker Compose V1.

### Solución

```bash
docker-compose up -d
```

---

### Incidencia 2 — Error DNS en Ubuntu Local

Error detectado:

```bash
Temporary failure in name resolution
```

### Solución Aplicada

```bash
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'

sudo systemctl restart systemd-resolved
```

---

## Paso 5 — Verificación de Contenedores

Se utilizó:

```bash
docker ps
```

Resultado:

| Contenedor | Imagen | Puerto | Estado |
|---|---|---|---|
| sitio_wordpress | wordpress:latest | 8080 → 80 | ✅ Up |
| panel_phpmyadmin | phpmyadmin:latest | 8081 → 80 | ✅ Up |
| servidor_mysql | mysql:8.0 | 3306 interno | ✅ Up |

---

## Paso 6 — Validación Funcional

### WordPress
Acceso mediante:

```text
Puerto 8080
```

### phpMyAdmin
Acceso mediante:

```text
Puerto 8081
```

---

# 10. Diagrama de la Arquitectura

```text
┌─────────────────────────────────────────────────────────┐
│                  NAVEGADOR DEL USUARIO                  │
│              (Killercoda - Pestaña Traffic)             │
└──────────────┬──────────────────────┬───────────────────┘
               │                      │
          Puerto 8080            Puerto 8081
               │                      │
┌──────────────▼──────────┐ ┌────────▼────────────────┐
│      WORDPRESS          │ │       phpMyAdmin         │
│  Apache + PHP           │ │  Interfaz Web MySQL      │
└──────────────┬──────────┘ └────────┬────────────────┘
               │                      │
               └──────────┬───────────┘
                          │
               ┌──────────▼──────────┐
               │    red_practica      │
               │    Driver: Bridge    │
               └──────────┬──────────┘
                          │
               ┌──────────▼──────────┐
               │       MySQL 8.0      │
               │   Base de Datos CMS  │
               └──────────────────────┘
```

---

# 11. Resultados Obtenidos y Evidencias
<img width="1024" height="480" alt="image" src="https://github.com/user-attachments/assets/f4948d08-05ab-4cbb-bed0-b656595cf112" />

<img width="538" height="600" alt="image" src="https://github.com/user-attachments/assets/94a5ca82-c9af-41bb-bafd-3d7460da779e" />

<img width="589" height="418" alt="image" src="https://github.com/user-attachments/assets/bbdbc590-8b6a-4ab7-a4fd-760f2f2c041b" />

<img width="589" height="483" alt="image" src="https://github.com/user-attachments/assets/5fc4f828-f33d-4a3d-baee-820be84c6c2f" />


## 11.1 Despliegue Exitoso

Los tres contenedores se ejecutaron correctamente en segundo plano.

## 11.2 Validación de WordPress

Se visualizó la pantalla inicial de instalación del CMS WordPress, confirmando:

- Correcto funcionamiento de Apache.
- Integración PHP funcional.
- Comunicación exitosa con MySQL.

## 11.3 Validación de phpMyAdmin

El panel de autenticación cargó correctamente utilizando:

- Usuario: `root`
- Contraseña: `12345`

---

## 11.4 Resumen de Validaciones

| Validación | Resultado |
|---|---|
| Descarga de imágenes | ✅ Exitoso |
| Creación de red Bridge | ✅ Exitoso |
| Creación de volúmenes | ✅ Exitoso |
| WordPress operativo | ✅ Exitoso |
| phpMyAdmin operativo | ✅ Exitoso |
| Comunicación entre contenedores | ✅ Exitoso |
| Persistencia de datos | ✅ Exitoso |

---

# 12. Conclusiones

- Docker Compose simplifica enormemente la administración de arquitecturas multi-servicio.
- WordPress integra Apache y PHP de manera nativa, facilitando el despliegue.
- Las redes Bridge permiten comunicación segura mediante resolución DNS interna.
- Los volúmenes persistentes garantizan la conservación de datos.
- La resolución de errores reales fortaleció el aprendizaje práctico.
- Killercoda resultó ser una excelente plataforma para laboratorios Docker.

---

# 13. Recomendaciones

- Nunca utilizar contraseñas débiles en producción.
- Implementar Docker Secrets para mayor seguridad.
- Migrar hacia Docker Compose V2.
- Añadir `healthchecks` en producción.
- Utilizar imágenes oficiales y mantenerlas actualizadas.

---

# 14. Bibliografía

1. Docker Inc. (2024). *Compose file version 3 reference*.  
   https://docs.docker.com/compose/compose-file/compose-version3/

2. Docker Inc. (2024). *Docker Hub Official Repository*.  
   https://hub.docker.com/

3. Docker Inc. (2024). *Networking in Compose*.  
   https://docs.docker.com/compose/networking/

4. Méndez, A. R. (2021). *Contenedores de software: una alternativa para el despliegue de aplicaciones web*. Revista Cubana de Ciencias Informáticas.

5. Killercoda. (2024). *Port-forwarding in Docker - Interactive Scenario*.  
   https://killercoda.com/
