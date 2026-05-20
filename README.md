INFORME DE PRÁCTICA: DESPLIEGUE DE INFRAESTRUCTURA WEB CONTAINERIZADA CON DOCKER COMPOSE
1. Título
Orquestación de una Plataforma CMS WordPress mediante Contenedores Docker: Configuración Integrada de Servidor Web, Motor de Base de Datos Relacional y Administrador Gráfico phpMyAdmin utilizando Docker Compose

2. Tiempo de Duración
Parámetro	Detalle
Duración estimada	120 minutos
Duración real	~110 minutos
Tipo de actividad	Práctica de laboratorio guiada
3. Plataforma de Simulación
Se utilizó Killercoda como entorno de laboratorio virtualizado en la nube, específicamente el escenario denominado "Port-forwarding in Docker". Esta plataforma proporciona un playground preconfigurado con sistema operativo Ubuntu Linux, Docker Engine y Docker Compose ya instalados, lo que permite concentrarse directamente en la construcción de la infraestructura sin necesidad de instalar dependencias previamente.

La elección de este escenario fue estratégica, ya que habilita una pestaña de Traffic / Port Forwarding que permite acceder remotamente a los puertos expuestos por los contenedores desde el navegador del usuario.

4. Fundamentos Teóricos
4.1 Contenedores y su Impacto en el Desarrollo Moderno
La virtualización basada en contenedores representa un cambio de paradigma en la manera en que se empaquetan, distribuyen y ejecutan las aplicaciones. A diferencia de las máquinas virtuales convencionales —que requieren un sistema operativo completo por cada instancia—, los contenedores comparten el kernel del sistema operativo anfitrión. Esto se traduce en:

Menor consumo de recursos (CPU, RAM, almacenamiento).
Tiempos de arranque casi instantáneos (segundos vs. minutos).
Portabilidad garantizada: una aplicación empaquetada en un contenedor se ejecuta de forma idéntica en cualquier máquina que tenga Docker instalado, eliminando el clásico problema de "en mi máquina sí funciona".
4.2 Docker como Motor de Contenedores
Docker es la plataforma líder del ecosistema de contenedores. Su arquitectura se fundamenta en tres pilares:

Componente	Función
Docker Engine	Daemon que gestiona el ciclo de vida de los contenedores
Docker Images	Plantillas de solo lectura que definen el sistema de archivos del contenedor
Docker Hub	Registro público de imágenes oficiales y comunitarias
4.3 Docker Compose: Orquestación Declarativa
Cuando una aplicación se compone de múltiples servicios interdependientes (patrón de microservicios), ejecutar cada contenedor manualmente con docker run resulta tedioso, propenso a errores y difícil de reproducir. Docker Compose resuelve este problema al permitir definir toda la arquitectura de servicios en un único archivo declarativo en formato YAML (docker-compose.yml).

Las ventajas principales de Docker Compose incluyen:

Definición centralizada: servicios, redes, volúmenes y variables de entorno en un solo archivo.
Comandos unificados: levantar (up) o detener (down) toda la infraestructura con una sola instrucción.
Reproducibilidad: cualquier persona con el archivo YAML puede replicar el mismo entorno exacto.
Gestión de dependencias: el atributo depends_on garantiza el orden correcto de inicio de los servicios.
4.4 Componentes de la Arquitectura Implementada
En esta práctica se orquestaron tres microservicios que operan de manera coordinada:

Servicio	Imagen Docker	Rol en la Arquitectura
MySQL 8.0	mysql:8.0	Motor de base de datos relacional que almacena toda la información del CMS (usuarios, publicaciones, configuraciones, taxonomías)
WordPress	wordpress:latest	Sistema de gestión de contenidos (CMS) que incluye de forma nativa el intérprete PHP y el servidor web Apache HTTP Server
phpMyAdmin	phpmyadmin:latest	Herramienta de administración visual basada en web para gestionar, consultar y auditar la base de datos MySQL a través del navegador
4.5 Conceptos Clave de Networking y Persistencia
Red Bridge Personalizada: Docker permite crear redes virtuales internas donde los contenedores se comunican entre sí mediante sus nombres de servicio como alias DNS, sin necesidad de conocer direcciones IP internas.
Volúmenes Persistentes: Por defecto, los datos dentro de un contenedor son efímeros. Al asignar volúmenes con nombre, los datos sobreviven a la destrucción y recreación del contenedor.
5. Conocimientos Previos Requeridos
Para la correcta ejecución de esta práctica, se requirieron los siguientes conocimientos:

Sintaxis y reglas de indentación del formato YAML (uso obligatorio de espacios, no tabulaciones).
Manejo de la terminal Bash y comandos fundamentales de Linux (cat, mkdir, systemctl, etc.).
Diferencia entre Docker CLI v1 (docker-compose) y Docker CLI v2 (docker compose).
Conceptos de redes virtuales tipo Bridge, mapeo de puertos (host:contenedor) y resolución DNS interna.
Fundamentos de bases de datos relacionales MySQL: usuarios, contraseñas, esquemas.
6. Objetivos
6.1 Objetivo General
Desplegar una infraestructura web multi-servicio completa mediante un archivo declarativo docker-compose.yml, integrando un CMS WordPress, un motor de base de datos MySQL y un panel de administración phpMyAdmin dentro del entorno virtualizado de Killercoda.

6.2 Objetivos Específicos
Redactar un archivo docker-compose.yml sintácticamente válido, respetando la indentación y estructura jerárquica exigidas por YAML.
Configurar e integrar los tres servicios (mysql, wordpress, phpmyadmin) con sus respectivas variables de entorno, puertos y dependencias.
Establecer una red puente personalizada (red_practica) para aislar y facilitar la comunicación interna entre contenedores.
Asignar volúmenes con nombre independientes (datos_web, archivos_web) para garantizar la persistencia de datos de MySQL y WordPress.
Validar el correcto funcionamiento de cada servicio mediante acceso web remoto a través de los puertos mapeados.
Documentar y resolver las incidencias técnicas encontradas durante el proceso de despliegue.
7. Equipo y Herramientas Necesarias
Recurso	Especificación
Computador	Con navegador web moderno (Chrome, Firefox, Edge)
Conexión a Internet	Estable, para acceder a Killercoda y descargar imágenes de Docker Hub
Cuenta Killercoda	Cuenta activa y verificada en la plataforma
Escenario	Port-forwarding in Docker con Docker Engine y Compose preinstalados
8. Material de Apoyo
Documentación oficial de Docker Compose: https://docs.docker.com/compose/compose-file/
Repositorios oficiales de imágenes en Docker Hub: https://hub.docker.com/
Imagen oficial de WordPress
Imagen oficial de MySQL
Imagen oficial de phpMyAdmin
Guía de referencia de formato YAML: https://yaml.org/spec/
9. Procedimiento Detallado
A continuación se describe paso a paso el procedimiento ejecutado en la terminal del playground de Killercoda.

Paso 1 — Selección del Escenario y Diagnóstico Inicial
Se ingresó a la plataforma Killercoda y se seleccionó el escenario "Port-forwarding in Docker". Este escenario provee un entorno Ubuntu con Docker preinstalado y, fundamentalmente, una pestaña de Traffic que permite redirigir tráfico desde el navegador hacia los puertos expuestos por los contenedores.

Se verificó que Docker estuviera operativo ejecutando:

bash

docker --version
docker-compose --version
Paso 2 — Creación del Archivo docker-compose.yml
Se generó el archivo de configuración declarativo utilizando el operador de redirección cat << 'EOF', que permite inyectar contenido multilínea directamente desde la terminal sin necesidad de un editor de texto:

yaml

cat << 'EOF' > docker-compose.yml
version: '3.8'
# ============================================
# SERVICIOS: Definición de los 3 microservicios
# ============================================
services:
  # ── Servicio 1: Base de Datos MySQL ──
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
  # ── Servicio 2: CMS WordPress ──
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
  # ── Servicio 3: phpMyAdmin ──
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
# ============================================
# RED: Red virtual personalizada tipo Bridge
# ============================================
networks:
  red_practica:
    driver: bridge
# ============================================
# VOLÚMENES: Persistencia de datos
# ============================================
volumes:
  datos_web:
  archivos_web:
EOF
Explicación de las directivas clave:

Directiva	Propósito
image	Especifica la imagen base de Docker Hub que se descargará
container_name	Asigna un nombre legible al contenedor en lugar de uno aleatorio
restart: always	Reinicia automáticamente el contenedor si se detiene o si el host se reinicia
ports: "8080:80"	Mapea el puerto 80 interno del contenedor al puerto 8080 del host
environment	Define variables de entorno para configurar credenciales y conexiones
volumes	Monta un volumen con nombre para persistencia de datos
networks	Conecta el servicio a la red virtual personalizada
depends_on	Establece que el servicio actual depende de otro para iniciar
Paso 3 — Verificación del Archivo YAML
Se confirmó que el archivo se creó correctamente ejecutando:

bash

cat docker-compose.yml
Se revisó visualmente que la indentación fuera consistente (solo espacios, nunca tabulaciones) y que todas las variables de entorno estuvieran correctamente definidas.

Paso 4 — Despliegue de la Infraestructura
Se ejecutó el comando de orquestación en modo detached (segundo plano):

bash

docker-compose up -d
Este comando realiza las siguientes acciones de forma automática:

Descarga las imágenes mysql:8.0, wordpress:latest y phpmyadmin:latest desde Docker Hub.
Crea la red virtual red_practica con driver Bridge.
Crea los volúmenes datos_web y archivos_web.
Levanta los tres contenedores en el orden definido por depends_on (primero MySQL, luego WordPress y phpMyAdmin).
Ejecución exitosa de docker-compose up -d mostrando la descarga de imágenes y creación de la red y volúmenes

Figura 1. Salida del comando docker-compose up -d en la terminal de Killercoda, donde se observa la descarga completa de las capas de las imágenes MySQL y WordPress, así como la creación de la red root_red_practica y los volúmenes root_datos_web y root_archivos_web.

⚠️ Incidencias Técnicas Detectadas y Resueltas
Durante la ejecución de la práctica se presentaron dos incidencias que fueron resueltas satisfactoriamente:

Incidencia 1: Incompatibilidad de Versión de Docker CLI
Al intentar ejecutar la sintaxis moderna de Docker Compose V2:

bash

docker compose up -d    # ❌ Falló
El sistema arrojó un error de interpretación de parámetros. El playground operaba bajo Docker Compose V1 (binario independiente docker-compose), no la versión V2 integrada en el CLI de Docker.

Solución aplicada:

bash

docker-compose up -d    # ✅ Comando correcto para V1
Incidencia 2: Fallo de Resolución DNS en VM Externa
En pruebas paralelas realizadas en una máquina virtual Ubuntu local, se presentó el error Temporary failure in name resolution al intentar descargar las imágenes desde Docker Hub.

Solución aplicada:

bash

# Forzar servidor DNS público de Google
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
# Reiniciar el servicio de resolución de nombres
sudo systemctl restart systemd-resolved
Paso 5 — Verificación del Estado de los Contenedores
Se ejecutó el comando de auditoría:

bash

docker ps
La salida confirmó que los tres contenedores estaban en estado Up y con los puertos correctamente mapeados:

Contenedor	Imagen	Puerto Mapeado	Estado
sitio_wordpress	wordpress:latest	0.0.0.0:8080 → 80/tcp	✅ Up
panel_phpmyadmin	phpmyadmin:latest	0.0.0.0:8081 → 80/tcp	✅ Up
servidor_mysql	mysql:8.0	3306/tcp (interno)	✅ Up
Paso 6 — Validación Funcional mediante Acceso Web
Utilizando la pestaña Traffic / Port Forwarding de Killercoda, se accedió a cada servicio web:

Puerto 8080 → WordPress
Puerto 8081 → phpMyAdmin
10. Diagrama de la Arquitectura

┌─────────────────────────────────────────────────────────┐
│                  NAVEGADOR DEL USUARIO                  │
│              (Killercoda - Pestaña Traffic)              │
└──────────────┬──────────────────────┬───────────────────┘
               │                      │
          Puerto 8080            Puerto 8081
               │                      │
┌──────────────▼──────────┐ ┌────────▼────────────────┐
│      WORDPRESS          │ │       phpMyAdmin         │
│  ┌───────────────────┐  │ │  ┌──────────────────┐   │
│  │   Apache HTTP      │  │ │  │  Interfaz Web    │   │
│  │   + PHP Engine     │  │ │  │  de Gestión DB   │   │
│  └───────────────────┘  │ │  └──────────────────┘   │
│  Puerto interno: 80     │ │  Puerto interno: 80     │
│  Volumen: archivos_web  │ │                         │
└──────────────┬──────────┘ └────────┬────────────────┘
               │                      │
               └──────────┬───────────┘
                          │
               ┌──────────▼──────────┐
               │   RED: red_practica  │
               │   (Driver: Bridge)   │
               │                      │
               │  Resolución DNS por  │
               │  nombre de servicio  │
               │  ┌──────────────┐    │
               │  │ mysql → 3306 │    │
               │  └──────────────┘    │
               └──────────┬───────────┘
                          │
               ┌──────────▼──────────┐
               │       MySQL 8.0      │
               │   Puerto: 3306/tcp   │
               │   DB: wordpress_db   │
               │   Volumen: datos_web │
               └──────────────────────┘
Figura 2. Diagrama de topología de la infraestructura desplegada, mostrando la relación entre servicios, mapeo de puertos, red interna y volúmenes de persistencia.

11. Resultados Obtenidos y Evidencias
11.1 Despliegue Exitoso de los Contenedores
Los tres contenedores se levantaron exitosamente en modo detached, sin errores de conexión ni conflictos de puertos. Las imágenes fueron descargadas completamente desde Docker Hub y cada servicio reportó estado Up de forma estable.

11.2 Acceso al CMS WordPress (Puerto 8080)
Al acceder al puerto 8080 a través de la pestaña Traffic de Killercoda, se cargó correctamente la pantalla de instalación inicial de WordPress, lo que confirma:

La imagen de WordPress se ejecutó sin errores.
El servidor web Apache integrado está respondiendo peticiones HTTP en el puerto 80.
La conexión con la base de datos MySQL a través de la red red_practica fue exitosa.
Pantalla de instalación de WordPress mostrando el selector de idioma

Figura 3. Pantalla de selección de idioma del asistente de instalación de WordPress, accedida mediante el puerto 8080 redirigido desde Killercoda.

11.3 Acceso al Panel phpMyAdmin (Puerto 8081)
Al acceder al puerto 8081, se visualizó el panel de autenticación de phpMyAdmin, confirmando:

La imagen de phpMyAdmin se desplegó correctamente.
La variable PMA_HOST: mysql permitió la resolución DNS interna hacia el contenedor de la base de datos.
El inicio de sesión con las credenciales root / 12345 fue exitoso.
Panel de inicio de sesión de phpMyAdmin con campos de usuario y contraseña

Figura 4. Interfaz de autenticación de phpMyAdmin accedida por el puerto 8081, donde se observa el formulario de inicio de sesión con el usuario root y la conexión exitosa al servidor MySQL.

11.4 Resumen de Validaciones
Validación	Resultado	Evidencia
Descarga de imágenes Docker	✅ Exitoso	Figura 1 — Terminal
Creación de red Bridge	✅ Exitoso	root_red_practica creada
Creación de volúmenes	✅ Exitoso	root_datos_web y root_archivos_web
WordPress operativo	✅ Exitoso	Figura 3 — Pantalla de instalación
phpMyAdmin operativo	✅ Exitoso	Figura 4 — Panel de autenticación
Comunicación inter-contenedores	✅ Exitoso	WordPress → MySQL vía DNS
Persistencia de datos	✅ Exitoso	Volúmenes nombrados asignados
12. Conclusiones
Docker Compose demostró ser una herramienta indispensable para la gestión de arquitecturas multi-servicio, al permitir definir, configurar y desplegar tres servicios interdependientes con un único archivo YAML y un solo comando de ejecución.

La imagen oficial de WordPress simplifica enormemente el despliegue al integrar de forma nativa el intérprete PHP y el servidor web Apache, eliminando la necesidad de instalar y configurar estos componentes de forma individual.

Las redes Bridge personalizadas facilitan la comunicación segura entre contenedores, permitiendo que los servicios se descubran mutuamente mediante nombres de host (aliases DNS) en lugar de depender de direcciones IP dinámicas.

Los volúmenes con nombre constituyen un mecanismo esencial de persistencia, ya que aseguran que los datos almacenados en MySQL y los archivos del sitio WordPress sobrevivan a la eliminación y recreación de contenedores, lo cual es fundamental en entornos de desarrollo y producción.

La resolución de incidencias técnicas (compatibilidad V1/V2 de Docker Compose y fallos DNS) enriqueció la experiencia práctica al exponer problemáticas reales que un administrador de sistemas encontrará en entornos productivos.

La plataforma Killercoda resultó ser un entorno idóneo para el aprendizaje práctico, al proveer infraestructura preconfigurada con capacidades de redirección de puertos que permiten validar servicios web de forma inmediata.

13. Recomendaciones
En entornos de producción, nunca utilizar contraseñas simples como 12345. Se recomienda emplear secretos gestionados mediante Docker Secrets o variables de entorno cifradas.
Considerar el uso de Docker Compose V2 (docker compose sin guion) en instalaciones modernas, ya que V1 ha sido marcado como deprecated.
Implementar health checks en el archivo YAML para que Docker pueda verificar automáticamente la disponibilidad de cada servicio antes de iniciar los dependientes.
14. Bibliografía
Docker Inc. (2024). Compose file version 3 reference. Docker Documentation. https://docs.docker.com/compose/compose-file/compose-version3/

Docker Inc. (2024). Docker Hub Official Repository. https://hub.docker.com/

Docker Inc. (2024). Networking in Compose. Docker Documentation. https://docs.docker.com/compose/networking/

Méndez, A. R. (2021). Contenedores de software: una alternativa para el despliegue de aplicaciones web. Revista Cubana de Ciencias Informáticas.

Killercoda. (2024). Port-forwarding in Docker - Interactive Scenario. https://killercoda.com/
