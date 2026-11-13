# 📦 Repositorio de Asignatura: DAW_2_DAW
_Actualizado automáticamente: 2026-11-13 09:21 UTC_

## 📂 Índice de ejercicios incluidos
- `DAW_2_2_Hanouni_Otero_Karim-main/evidencias.md`
- `Tarea_2.1_DAW-master/documentacion.md`
- `documentacion-daw-main/README.md`
- `documentacion-daw-main/blog/authors.yml`
- `documentacion-daw-main/blog/tags.yml`
- `documentacion-daw-main/docs/analisis/requisitos.md`
- `documentacion-daw-main/docs/despliegue/github_pages.md`
- `documentacion-daw-main/docs/diseño/diagrama_clases.md`
- `documentacion-daw-main/docs/implementacion/estructura_proyecto.md`
- `documentacion-daw-main/docs/pruebas/analisis_caja_blanca.md`
- `documentacion-daw-main/docs/tutorial-basics/_category_.json`
- `documentacion-daw-main/docs/tutorial-extras/_category_.json`
- `documentacion-daw-main/package.json`
- `documentacion-daw-main/src/components/HomepageFeatures/styles.module.css`
- `documentacion-daw-main/src/css/custom.css`
- `documentacion-daw-main/src/pages/index.module.css`
- `documentacion-daw-main/tsconfig.json`


---
## 📄 Ejercicio / Archivo: DAW_2_2_Hanouni_Otero_Karim-main/evidencias.md
---

# EVIDENCIAS PRÁCTICAS DOCKER

## 1. Gestión de Redes Básica y DNS
Creo el contenedor inicial:

![img.png](img.png)

El contenedor temporal intenta conectarse a `cache` por su nombre, pero falla porque la red por defecto de Docker (`bridge`) no proporciona resolución DNS entre contenedores:

![img_1.png](img_1.png)

> **Nota sobre el comando ping:** Se utiliza para verificar la conectividad de red y el estado de salud del servicio. Al recibir la respuesta `PONG`, tenemos la confirmación empírica de que el contenedor responde y que la comunicación directa a través de la IP en la red `bridge` funciona correctamente, compensando la falta de resolución DNS.

---

*A partir de aquí, el ejercicio se realizó en entorno local utilizando Windows 11.*

## 2. Redes Personalizadas y Aislamiento
Creo la red personalizada y conecto el contenedor:

![img_5.png](img_5.png)

El valor se encuentra almacenado en la memoria RAM asignada al contenedor `cache2` y en su capa efímera de lectura y escritura:

![img_3.png](img_3.png)

> **¿Por qué en la configuración de una aplicación se usan nombres de contenedor y no direcciones IP, y por qué para todo esto no ha hecho falta publicar ningún puerto?**
> En las configuraciones de aplicaciones se utilizan los nombres de los contenedores porque sus direcciones IP son dinámicas y cambian cada vez que se reinician o se vuelven a crear, mientras que el nombre permanece estático. No es necesario publicar puertos (usando `-p`) porque todos los contenedores que comparten la misma red interna de Docker tienen acceso a todos sus puertos recíprocamente de forma predeterminada.

Al destruir los contenedores sin haber configurado previamente volúmenes persistentes, todos los datos almacenados en ellos se pierden de forma permanente:

![img_4.png](img_4.png)

### Segmentación de Redes
Creo ambas redes:

![img_6.png](img_6.png)

Inicialmente, no están conectadas:

![img_7.png](img_7.png)

Una vez conectadas, recibimos la respuesta de confirmación:

![img_8.png](img_8.png)

Y con un contenedor temporal comprobamos que no devuelve `PONG` si no está en la red correcta:

![img_9.png](img_9.png)

**Conclusión sobre Arquitectura de Red:**
Esta arquitectura de redes segmentadas garantiza que la base de datos (cache) permanezca oculta e inaccesible desde el exterior. Si un atacante logra comprometer un contenedor expuesto al público en la red frontal (como un servidor web), no podrá saltar directamente a la base de datos porque físicamente no hay ruta de red hacia ella. Solo los servicios explícitamente autorizados (como la API, conectada a ambas redes) actúan como puente seguro.

---

## 3. Gestión de Volúmenes y Persistencia de Datos
Creamos una red, su volumen asociado y levantamos el entorno:

![img_10.png](img_10.png)

Creamos la base de datos:

![img_11.png](img_11.png)

Borramos el contenedor, pero luego mostramos el volumen persistente de ese contenedor:

![img_12.png](img_12.png)

Nos conectamos a la red mediante un contenedor temporal para verificar la persistencia:

![img_13.png](img_13.png)

> **Funcionamiento de los Volúmenes:** Al usar el flag `-v pgdata:/var/lib/postgresql/data`, le indicamos a Docker que guarde todo lo que la base de datos escriba en esa carpeta interna directamente en un volumen gestionado en el disco del servidor anfitrión. Cuando se elimina el contenedor, Docker destruye su capa efímera, pero el volumen `pgdata` queda intacto. Al arrancar un contenedor nuevo y engancharle el mismo volumen, el nuevo motor de base de datos lee los archivos persistidos y recupera toda la información sin perder nada.

*(Nota: Esta actividad 1.4 la he realizado en mi equipo local con Docker Desktop sobre Windows, por lo que he publicado el puerto 80 para poder testear la conectividad desde el anfitrión).*

![img_14.png](img_14.png)

### Resolución de Permisos en Entorno Windows (Bind Mounts)
Tuve que hacer unos arreglos por temas de compatibilidad en Windows:

![img_15.png](img_15.png)
![img_16.png](img_16.png)

**Explicación técnica:** He realizado este ejercicio localmente en Windows con Docker Desktop. Al crear el archivo desde dentro del contenedor (usuario root), el bind mount lo refleja en mi carpeta local. Sin embargo, el comando `ls` (o `Get-ChildItem` en PowerShell) de Windows no muestra a 'root' como propietario, ya que Docker Desktop gestiona y traduce los permisos entre el sistema de archivos de Linux y el de Windows para permitir la edición. Si esto se ejecutara nativamente en Linux, el archivo pertenecería a root y requeriría privilegios para ser modificado por el usuario del anfitrión.

---

## 4. Docker Compose: WordPress y MariaDB

Creo los archivos `.env` y `compose.yaml`, y levanto los contenedores:

![img_17.png](img_17.png)
![img_18.png](img_18.png)

Instalo WordPress a través del navegador:

![img_19.png](img_19.png)
![img_20.png](img_20.png)
![img_21.png](img_21.png)

> **Comportamiento Declarativo:** Al ejecutar `docker compose up -d` sin modificar nada, Compose compara el estado actual con el archivo `compose.yaml` y, al ser idénticos, no hace nada (0.0s). Al añadir la etiqueta `labels` al servicio `wp`, Compose detecta el cambio en la configuración y, de forma inteligente, destruye y recrea únicamente el contenedor afectado (`wp`), dejando intacta la base de datos (`db`).

Elimino el contenedor de WordPress:

![img_22.png](img_22.png)

**Persistencia en Compose:** Al eliminar el contenedor `wp` y volver a recrearlo con `docker compose up -d`, la entrada del blog se conserva intacta. Esto ocurre porque los datos no se guardan dentro del ciclo de vida efímero del contenedor, sino que persisten gracias a que los archivos web y la base de datos están almacenados en los volúmenes nombrados (`wp_data` y `db_data`) que declaramos en el `compose.yaml`.

![img_23.png](img_23.png)
![img_24.png](img_24.png)

**Destrucción de Volúmenes (-v):** El comando `docker compose down` detiene y elimina los contenedores y la red, pero por seguridad conserva intactos los volúmenes nombrados, permitiendo que la entrada sobreviva al volver a levantar el proyecto. Sin embargo, al añadir la bandera `-v` (`docker compose down -v`), ordenamos a Compose destruir también los volúmenes asociados. Esto elimina físicamente los datos de la base de datos y los archivos web, provocando que al hacer `up -d` nuevamente, WordPress nos reciba con el asistente de instalación limpia porque ha perdido toda su información.

![img_25.png](img_25.png)
![img_26.png](img_26.png)

**Validación sin archivo .env:** Al renombrar el archivo `.env`, Docker Compose pierde el acceso a las variables de entorno que definimos para las contraseñas y usuarios. Cuando ejecutamos `docker compose config` para validar el proyecto, el orquestador detecta que en el `compose.yaml` se están llamando a variables (como `${DB_USER}`) que no tienen ningún valor asignado en el sistema, lanzando advertencias de que esas variables están vacías o no configuradas. Esto demuestra la importancia del archivo `.env` para inyectar datos sensibles sin escribirlos directamente en el código.

![img_27.png](img_27.png)
![img_28.png](img_28.png)
---
## 📄 Ejercicio / Archivo: Tarea_2.1_DAW-master/documentacion.md
---

# EVIDENCIAS DE LA PRÁCTICA 2.1 DAW

## Ciclo de vida de un contenedor

### Actividad 1.1: Explorar una imagen

Ejecutamos `docker search httpd` para localizar la imagen oficial en Docker Hub:

![img.png](img.png)

Descargamos y comparamos el tamaño de las dos imágenes (`httpd:2.4` y `httpd:2.4-alpine`). La variante Alpine es sustancialmente más ligera porque utiliza una distribución Linux minimalista basada en musl libc y BusyBox, optimizada para contenedores:

![img_1.png](img_1.png)

Inspeccionamos los metadatos de la imagen con `docker image inspect`:

![img_2.png](img_2.png)

---

### Actividad 1.2: Ejecutar y comprobar un servidor web

Creamos y comprobamos el contenedor `apache-aula`:

![img_3.png](img_3.png)

---

### Actividad 1.3: Seguir el ciclo de vida

Detenemos el contenedor y verificamos que `curl` falla con error de conexión, ya que el servicio no está en ejecución para recibir peticiones:

![img_4.png](img_4.png)

Al iniciarlo de nuevo con `docker start`, el contenedor pasa a estado `Up` y `curl` vuelve a responder correctamente:

![img_5.png](img_5.png)
![img_6.png](img_6.png)

Al ejecutar `docker restart`, el contenedor reinicia su proceso principal manteniéndose accesible.

Al intentar borrar el contenedor con `docker rm apache-aula`, Docker rechaza la orden con error de seguridad porque está en ejecución (`Up`). Para forzar la detención y el borrado en un único comando se utiliza la opción `-f`:

![img_7.png](img_7.png)

Al eliminar el contenedor, la imagen base (`httpd:2.4-alpine`) sigue existiendo en el host. Los contenedores y las imágenes son entidades independientes; borrar una instancia en ejecución no elimina la plantilla base de solo lectura:

![img_8.png](img_8.png)

| Operación | Estado de apache-aula (STATUS) | ¿Responde curl? | ¿Sigue disponible httpd:2.4-alpine? |
| :--- | :--- | :--- | :--- |
| Antes de empezar | Up | Sí | Sí |
| `docker stop apache-aula` | Exited | No | Sí |
| `docker start apache-aula` | Up | Sí | Sí |
| `docker restart apache-aula` | Up | Sí | Sí |
| `docker rm apache-aula` | (Falla porque está Up) | Sí | Sí |
| `docker rm -f apache-aula` | (Desaparece del registro) | No | Sí |

---

### Actividad 1.4: Cambios dentro de un contenedor

Creamos el contenedor `apache-cambios` y añadimos el archivo `equipo.html`:

![img_9.png](img_9.png)

Con `docker diff` comprobamos las modificaciones introducidas en la capa de lectura y escritura (`A /usr/local/apache2/htdocs/equipo.html`):

![img_10.png](img_10.png)

Al detener el contenedor, observamos que desaparece el archivo de PID del servidor web (`D /usr/local/apache2/logs/httpd.pid`):

![img_11.png](img_11.png)

Al eliminar el contenedor (`docker rm -f`) y volver a levantarlo con la misma imagen, `equipo.html` desaparece arrojando un error 404 en `curl`, ya que la capa de escritura del contenedor anterior se destruyó junto con él:

![img_12.png](img_12.png)

> [!IMPORTANT]
> **No es adecuado editar en marcha** porque los contenedores son efímeros e inmutables: cualquier modificación manual se destruye y se pierde en cuanto el contenedor se reinicia o se elimina (`docker rm`), rompiendo el control de versiones y la reproducibilidad del entorno. La **forma correcta** consiste en modificar el código en el host, compilar una nueva imagen con `docker build` y levantar un nuevo contenedor que reemplace al anterior (o, en desarrollo local, montar un volumen con `-v` para sincronizar los archivos sin alterar la imagen base).

---

### Actividad 1.5: Diagnosticar contenedores que se detienen

Lanzamos los tres contenedores de diagnóstico en segundo plano:

![img_13.png](img_13.png)

Consultamos los registros con `docker logs`:

![img_14.png](img_14.png)

#### 1. Estado y códigos de salida

- **`diag-1`:** `ExitCode: 0` (Finalizado con éxito)
- **`diag-2`:** `ExitCode: 1` (Finalizado con error)
- **`diag-3`:** `ExitCode: 0` (Finalizado con éxito)

#### 2. Diagnóstico de detención

- **`diag-1` (`ubuntu:24.04`):**
    - **Causa:** Arranca por defecto una shell Bash en segundo plano (`-d`) sin terminal interactiva asociada (`-it`). Al no recibir ninguna entrada estándar (STDIN) ni script que ejecutar, Bash finaliza de inmediato con código `0`. Al terminar el proceso principal (PID 1), Docker detiene el contenedor.

- **`diag-2` (`postgres:17-alpine`):**
    - **Causa:** El registro indica que la base de datos no está inicializada y no se ha especificado la variable obligatoria `POSTGRES_PASSWORD`. Por política de seguridad, el script de entrada aborta la ejecución con código de error `1`.

- **`diag-3` (`nginx:alpine nginx -g "daemon on;"`):**
    - **Comparación:** El comando por defecto (`CMD`) de la imagen original es `["nginx", "-g", "daemon off;"]`, mientras que la orden ejecutada fuerza `daemon on;`.
    - **Efecto de `daemon on;` y por qué se apaga:** Esta opción indica a Nginx que se ejecute en segundo plano como un demonio tradicional de Linux. Al activarse, el proceso principal (PID 1) delega el servicio a procesos secundarios en segundo plano y finaliza su propia tarea con código `0`. Dado que Docker únicamente supervisa el PID 1, al terminar este considera que la tarea ha concluido y apaga el contenedor.

![img_15.png](img_15.png)
![img_16.png](img_16.png)

---

## Actividad 2.1: Construir imágenes paso a paso

### Parte A: Web estática con Apache

Creamos la carpeta `web-apache` con los archivos `index.html`, `estilos.css` y el archivo `Dockerfile`:

![img_17.png](img_17.png)

> **¿Por qué este Dockerfile no necesita CMD ni EXPOSE?**  
> Porque la imagen base `httpd:2.4-alpine` ya los define previamente (`CMD ["httpd-foreground"]` y `EXPOSE 80`)[cite: 7]. Al basarnos en ella mediante `FROM`, nuestro contenedor hereda ambas directivas automáticamente sin necesidad de reescribirlas.

---

### Parte B: La aplicación Node.js

Creamos la carpeta `saludo` junto a `package.json`, `app.js` y el `Dockerfile`[cite: 9]. Construimos la imagen y arrancamos el contenedor `app-saludo`:

![img_18.png](img_18.png)

Comprobamos que las rutas responden adecuadamente mediante `curl`:

![img_19.png](img_19.png)

Verificamos que la aplicación corre sin privilegios (`node`) y dentro del directorio `/app`:

![img_20.png](img_20.png)

Lanzamos un segundo contenedor (`app-saludo-2`) modificando las variables de entorno `PUERTO` y `MENSAJE` mediante el parámetro `-e`:

![img_21.png](img_21.png)

> **¿Qué tienes que cambiar también en la opción -p y por qué?**  
> Es obligatorio cambiar el puerto del lado derecho de `-p` (`:4000`) para que coincida con el nuevo valor de `PUERTO` en el que escucha Express. Si mantuviéramos `:3000`, las peticiones llegarían a un puerto donde la aplicación no atiende. Además, se modifica el puerto del host (`8085`) para no colisionar con el contenedor anterior.

---

## Actividad 2.2: Publicar y versionar una imagen

### 1. Publicación de la imagen 1.0 en Docker Hub

Etiquetamos la imagen con nuestro usuario (`karimblackstar/saludo:1.0`), la subimos al registro con `docker push` y eliminamos los contenedores e imágenes locales con `docker rm -f` y `docker rmi` para preparar la prueba remota:

![img_22.png](img_22.png)
![img_23.png](img_23.png)

* **Capas subidas vs reutilizadas:** La capa marcada como `Mounted from...` ya existía en el registro remoto (pertenece a la imagen base oficial) y se reutiliza para evitar transferencias redundantes; las capas marcadas como `Pushed` son las capas nuevas generadas durante la compilación.

---

### 2. Comprobación desde Docker Hub

Ejecutamos el contenedor a partir de la imagen remota (`karimblackstar/saludo:1.0`), verificando que Docker descarga las capas del registro y que el servicio responde mediante `curl`:

![img_24.png](img_24.png)

---

### 3. Versionado con versión 1.1 y etiqueta latest

Modificamos la variable `MENSAJE` en el `Dockerfile` y compilamos la nueva versión asignando las etiquetas `1.1` y `latest`:

![img_25.png](img_25.png)

Publicamos ambas etiquetas en Docker Hub mediante `docker push`:

![img_26.png](img_26.png)

---

### 4. Verificación de etiquetas y arquitectura en Docker Hub

Accedemos a Docker Hub y verificamos en la pestaña **Tags** las versiones publicadas (`1.0`, `1.1` y `latest`) junto con su arquitectura (`linux/amd64`):

![img_27.png](img_27.png)
![img_28.png](img_28.png)
---
## 📄 Ejercicio / Archivo: documentacion-daw-main/README.md
---

# Website

This website is built using [Docusaurus](https://docusaurus.io/), a modern static website generator.

## Installation

```bash
npm install
```

**Note**: feel free to use the package manager of your choice.

## Local Development

```bash
npm run start
```

This command starts a local development server and opens up a browser window. Most changes are reflected live without having to restart the server.

## Build

```bash
npm run build
```

This command generates static content into the `build` directory and can be served using any static contents hosting service.

## Deployment

Using SSH:

```bash
USE_SSH=true npm run deploy
```

Not using SSH:

```bash
GIT_USER=<Your GitHub username> npm run deploy
```

If you are using GitHub Pages for hosting, this command is a convenient way to build the website and push to the `gh-pages` branch.

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/blog/authors.yml
---

```yml
yangshun:
  name: Yangshun Tay
  title: Ex-Meta Staff Engineer, Co-founder GreatFrontEnd
  url: https://linkedin.com/in/yangshun
  image_url: https://github.com/yangshun.png
  page: true
  socials:
    x: yangshunz
    linkedin: yangshun
    github: yangshun
    newsletter: https://www.greatfrontend.com

slorber:
  name: Sébastien Lorber
  title: Docusaurus maintainer
  url: https://sebastienlorber.com
  image_url: https://github.com/slorber.png
  page:
    # customize the url of the author page at /blog/authors/<permalink>
    permalink: '/all-sebastien-lorber-articles'
  socials:
    x: sebastienlorber
    linkedin: sebastienlorber
    github: slorber
    newsletter: https://thisweekinreact.com

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/blog/tags.yml
---

```yml
facebook:
  label: Facebook
  permalink: /facebook
  description: Facebook tag description

hello:
  label: Hello
  permalink: /hello
  description: Hello tag description

docusaurus:
  label: Docusaurus
  permalink: /docusaurus
  description: Docusaurus tag description

hola:
  label: Hola
  permalink: /hola
  description: Hola tag description

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/docs/analisis/requisitos.md
---

# Especificación de Requisitos

## Requisitos Funcionales (RF)
- **RF-01**: El usuario debe poder iniciar sesión con credenciales válidas y soporte de tokens JWT.
- **RF-02**: Permitir el registro, edición y baja lógica de productos del catálogo.
- **RF-03**: Generar reportes de ventas mensuales exportables en formato PDF.

## Requisitos No Funcionales (RNF)
- **RNF-01 (Rendimiento)**: El tiempo de respuesta de las consultas a la API no debe superar los 300 ms.
- **RNF-02 (Seguridad)**: Las contraseñas se almacenarán cifradas mediante el algoritmo BCrypt.

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/docs/despliegue/github_pages.md
---

# Despliegue en GitHub Pages

Instrucciones para publicar la documentación web mediante GitHub Actions.

## Configuración del Repositorio
1. Acceder a **Settings** > **Pages**.
2. En la sección **Build and deployment**, seleccionar la fuente **GitHub Actions**.

## Pipeline de Despliegue
El flujo `.github/workflows/deploy.yml` compila los archivos Markdown y publica el contenido estático generado en la rama `gh-pages` en cada `push` a la rama `main`.

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/docs/diseño/diagrama_clases.md
---

# Diagrama de Clases

Representación de las entidades principales del dominio del sistema.

```text
+-------------------+       1..* +-------------------+
|      Usuario      |<>--------->|      Pedido       |
+-------------------+            +-------------------+
| - id: Long        |            | - id: Long        |
| - email: String   |            | - fecha: LocalDate|
| - pass: String    |            | - total: Double   |
+-------------------+            +-------------------+
| + login(): Bool   |            | + calcularTotal() |
+-------------------+            +-------------------+

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/docs/implementacion/estructura_proyecto.md
---

# Estructura del Proyecto

Organización modular del código fuente siguiendo el patrón Modelo-Vista-Controlador (MVC):

```text
src/
├── controllers/    # Endpoints y gestión de peticiones HTTP
├── models/         # Entidades de datos y esquemas de BD
├── views/          # Componentes de interfaz y plantillas
├── services/       # Lógica de negocio de la aplicación
└── utils/          # Constantes y funciones auxiliares

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/docs/pruebas/analisis_caja_blanca.md
---

# Análisis de Caja Blanca

Pruebas enfocadas en la cobertura de rutas de ejecución del módulo de autenticación.

## Método Evaluado: `validarAcceso(Usuario u)`

### Caminos Identificados
1. **Camino 1**: Usuario no existe en la base de datos $\rightarrow$ Retorna error 404.
2. **Camino 2**: Contraseña incorrecta $\rightarrow$ Incrementa contador de intentos y retorna error 401.
3. **Camino 3**: Credenciales correctas y cuenta activa $\rightarrow$ Genera token de sesión.

## Métricas de Cobertura
- **Cobertura de ramas**: 100%
- **Complejidad ciclomática**: $V(G) = 3$

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/docs/tutorial-basics/_category_.json
---

```json
{
  "label": "Tutorial - Basics",
  "position": 2,
  "link": {
    "type": "generated-index",
    "description": "5 minutes to learn the most important Docusaurus concepts."
  }
}

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/docs/tutorial-extras/_category_.json
---

```json
{
  "label": "Tutorial - Extras",
  "position": 3,
  "link": {
    "type": "generated-index"
  }
}

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/package.json
---

```json
{
  "name": "documentacion",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "docusaurus": "docusaurus",
    "start": "docusaurus start",
    "build": "docusaurus build",
    "swizzle": "docusaurus swizzle",
    "deploy": "docusaurus deploy",
    "clear": "docusaurus clear",
    "serve": "docusaurus serve",
    "write-translations": "docusaurus write-translations",
    "write-heading-ids": "docusaurus write-heading-ids",
    "typecheck": "tsc"
  },
  "dependencies": {
    "@docusaurus/core": "3.10.2",
    "@docusaurus/faster": "3.10.2",
    "@docusaurus/preset-classic": "3.10.2",
    "@mdx-js/react": "^3.0.0",
    "clsx": "^2.0.0",
    "prism-react-renderer": "^2.3.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@docusaurus/module-type-aliases": "3.10.2",
    "@docusaurus/tsconfig": "3.10.2",
    "@docusaurus/types": "3.10.2",
    "@types/react": "^19.0.0",
    "typescript": "~6.0.2"
  },
  "browserslist": {
    "production": [
      ">0.5%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 3 chrome version",
      "last 3 firefox version",
      "last 5 safari version"
    ]
  },
  "engines": {
    "node": ">=20.0"
  }
}

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/src/components/HomepageFeatures/styles.module.css
---

```css
.features {
  display: flex;
  align-items: center;
  padding: 2rem 0;
  width: 100%;
}

.featureSvg {
  height: 200px;
  width: 200px;
}

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/src/css/custom.css
---

```css
/**
 * Any CSS included here will be global. The classic template
 * bundles Infima by default. Infima is a CSS framework designed to
 * work well for content-centric websites.
 */

/* You can override the default Infima variables here. */
:root {
  --ifm-color-primary: #2e8555;
  --ifm-color-primary-dark: #29784c;
  --ifm-color-primary-darker: #277148;
  --ifm-color-primary-darkest: #205d3b;
  --ifm-color-primary-light: #33925d;
  --ifm-color-primary-lighter: #359962;
  --ifm-color-primary-lightest: #3cad6e;
  --ifm-code-font-size: 95%;
  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.1);
}

/* For readability concerns, you should choose a lighter palette in dark mode. */
[data-theme='dark'] {
  --ifm-color-primary: #25c2a0;
  --ifm-color-primary-dark: #21af90;
  --ifm-color-primary-darker: #1fa588;
  --ifm-color-primary-darkest: #1a8870;
  --ifm-color-primary-light: #29d5b0;
  --ifm-color-primary-lighter: #32d8b4;
  --ifm-color-primary-lightest: #4fddbf;
  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.3);
}

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/src/pages/index.module.css
---

```css
/**
 * CSS files with the .module.css suffix will be treated as CSS modules
 * and scoped locally.
 */

.heroBanner {
  padding: 4rem 0;
  text-align: center;
  position: relative;
  overflow: hidden;
}

@media screen and (max-width: 996px) {
  .heroBanner {
    padding: 2rem;
  }
}

.buttons {
  display: flex;
  align-items: center;
  justify-content: center;
}

```

---
## 📄 Ejercicio / Archivo: documentacion-daw-main/tsconfig.json
---

```json
// This file is not used by "docusaurus start/build" commands.
// It is here to improve your IDE experience (type-checking, autocompletion...),
// and can also run the package.json "typecheck" script manually.
{
  "extends": "@docusaurus/tsconfig",
  "compilerOptions": {
    "baseUrl": ".",
    "ignoreDeprecations": "6.0",
    "strict": true
  },
  "exclude": [".docusaurus", "build"]
}

```
