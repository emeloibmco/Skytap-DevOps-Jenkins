# Skytap-DevOps-Jenkins

## Entorno de integración :computer:

---

Se aprovisiona una máquina en skytap haciendo uso de la plantilla de _Jenkins v2.121.3 on Ubuntu 16.04.5 LTS Desktop-Firstboot_ la cual tiene instalada la versión _2.121.3_ de la herramienta de integración contínua _Jenkins_.

### 1. Creación del proyecto

Ingrese a la interfaz de jenkins mediante la IP o la URL provista por el DNS de la máquina por el puerto 8080, las credenciales las encontrará en skytap, en la sección VM settings -> credentials.<br/><br/>
Una vez ingrese sus credenciales seleccione **_crear nueva tarea_** para crear un **_proyecto libre_**.<br/>
![Creación del proyecto en UI de Jenkins](https://raw.githubusercontent.com/emeloibmco/Skytap-DevOps-Jenkins/master/.github/images/proyecto.png)

Es importante que configure el origen del código fuente, para esto, seleccionamos _git_ y en el campo **_Repository URL_** ingresamos la [dirección de nuestro repositorio](https://github.com/mayi29/js-jenkis). No olvide activar la opción _"GitHub hook trigger for GITScm polling"_ dentro de las configuraciones de los _build triggers_.

### 2. Agregar un Webhook de GitHub a su pipeline de Jenkins

Los webhooks de GitHub en Jenkins se usan para activar la compilación cada vez que un desarrollador confirma algo a la rama maestra. Para configurarlo tenga en cuenta los siguientes pasos:

- Vaya al repositorio de su proyecto.
- Vaya a "configuración" en la esquina derecha.
- Haga clic en "webhooks".
- Haga clic en "Agregar webhooks".
- Agregue la payload URL y agregue al final /github-webhook para decirle a GitHub que es un webhook. payloadURL:`http://happymontalcini.ibmlatin.skytapdns.com/github-webhook`<br/>
- En Content type seleccione la opción _application/json_

### 3. Plugin de SSH

A continuación, se creará la configuración del host dentro de la configuración principal de Jenkins. Para esto, ingrese a la página principal de Jenkins en donde dentro de la configuración del sistema encontrará la sección _Publish Over SSH_ aquí deberá pegar la clave privada no cifrada dentro del recuadro _key_, incluir los indicadores de la llave al inicio y al final de nuestra llave. Adicionalmente, se deberán llenar los siguientes recuadros para SSH Server.

- Passphrase: Frase de contraseña para la clave privada, si está cifrada
- Name: Nombre de identificación del servidor para Jenkins.
- Hostname: Nombre de host SSH o dirección IP.
- Username: Nombre de usuario con el que se conectará al PC
- Port: 22 (Este es el puerto predeterminado para SSH)
- Timeout: 300000 (Tiempo de espera en milisegundos para las conexiones SSH)

Una vez terminado guarde los cambios realizados.

#### Configuración del plugin en nuestro proyecto de Jenkins

En la pestaña **_Ejecutar_** agregamos los comandos correspondientes a la fase de compilación (Build) y pruebas (Test) del proyecto.

![Comandos de fase](https://raw.githubusercontent.com/emeloibmco/Skytap-DevOps-Jenkins/master/.github/images/fases.png)

En la imagen se muestran los pasos para un proyecto de Node.js, los comandos varían según la configuración personal de su proyecto.

##### Despliegue :rocket:

Vamos a la pestaña _**Acciones para ejecutar después**_, haciendo referencia al manejo que se le dará al artefacto (paquete) resultante de la fase anterior.

Editamos los campos según nuestra configuración de Jenkins, en este caso:

- Name: Nombre de identificación colocado en la configuración del plugin.
- Transfers:
  - Source files: definimos los archivos que queremos llevar a nuestro servidor de producción, en patrón de _fileset_ Ant.
  - Remove prefix: si queremos remover los prefijos de la ruta de nuestro artefacto. Útil para cuando se quiere organizar nuestro proyecto en la carpeta `/home` de nuestro servidor.
  - Remote directory: carpeta en la que queremos que se guarden los archivos.
  - Exec command: comando de despliegue, este se ejecuta en la máquina de producción desde la carpeta `/home`, se pueden utilizar variables de entorno.

![Despliegue](https://raw.githubusercontent.com/emeloibmco/Skytap-DevOps-Jenkins/master/.github/images/despliegue.png)

Guardamos los cambios realizados.

### 4. Conexión SSH con la máquina de producción

Se instala SSH Server en la terminal de Ubuntu

`sudo apt-get install ssh`

Se debe crear una clave SSH con el propósito de facilitar los inicios de sesión automatizados y sin contraseña entre ambas máquinas. Para generar la clave ssh utilice el sieguiente comando en el servidor Jenkins:

`ssh-keygen -t rsa -b 2048 -f ~/.ssh/mykey`

**_id_rsa_** es la clave privada, **_id_rsa.pub_** es la clave pública. Una vez generado el par de claves hay que copiar la clave pública a nuestro entorno de producción:

`ssh-copy-id -i ~/.ssh/mykey user@host`

Si el comando anterior no funciona correctamente, utilizar:

`cat ~/.ssh/id_rsa.pub | ssh username@server.address.com 'cat >> ~/.ssh/authorized_keys'`

En la máquina de producción, si la carpeta .ssh no existe, se debe crear junto con el archivo authorized_keys, usando los siguientes comandos:

```sh
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```

---

## Entorno de producción :computer:

Se aprovisiona una máquina en skytap haciendo uso de la plantilla cuyo sistema operativo es ubuntu. Para el despliegue de la aplicación se desarrolla el siguiente procedimiento:

### Instalación de prerequisitos

Necesitaremos actualizar el sistema operativo e instalar los paquetes necesarios para la ejecución.

- ssh
- Node y npm
- Git

```sh
sudo apt-get update && sudo apt-get upgrade \
sudo apt-get install ssh \
sudo apt-get install git
```

Para instalar Node utilizaremos el repositorio oficial de NodeSource.<br/>
**_Nota_**: para el tiempo de realización de está guía, se utiliza la última versión de Node, cambiar si es el caso 13.x por el necesario.

```sh
curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash - \
sudo apt install nodejs
```
