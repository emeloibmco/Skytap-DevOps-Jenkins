# Skytap-DevOps-Jenkins
## Entorno de integración :computer:
---
Se aprovisiona una máquina en skytap haciendo uso de la plantilla de _Jenkins v2.121.3 on Ubuntu 16.04.5 LTS Desktop-Firstboot_ la cual tiene instalada la versión _2.121.3_ de la herramienta de integración contínua _Jenkins_. 

### 1.Creación del proyecto
Ingrese a la interfaz de jenkins mediante la IP o el DNS de la máquina por el puerto 8080, las credenciales las encontrará en skytap, en la sección VM settings -> credentials.
Una vez ingrese sus credenciales seleccione _crear nueva tarea_ para crear un _proyecto libre_. Es importante que configure el origen del código fuente, para esto, seleccionamos _git_ y en el campo _Repository URL_ ingresamos la [dirección de nuestro repositorio](https://github.com/mayi29/js-jenkis). No olvide activar la opción _"GitHub hook trigger for GITScm polling"_ dentro de las configuraciones de los _build triggers_.

### 2.Agregar un Webhook de GitHub a su pipeline de Jenkins
Los webhooks de GitHub en Jenkins se usan para activar la compilación cada vez que un desarrollador confirma algo a la rama maestra. Para configurarlo tenga en cuenta los siguientes pasos:

- Vaya al repositorio de su proyecto.
- Vaya a "configuración" en la esquina derecha.
- Haga clic en "webhooks".
- Haga clic en "Agregar webhooks".
- Agregue la payload URL y agregue al final /github-webhook para decirle a GitHub que es un webhook. payloadURL:`http://happymontalcini.ibmlatin.skytapdns.com/github-webhook`<br/>
- En Content type seleccione la opción _application/json_

### 3.Plugin de SSH
A continuación, se creará la configuración del host dentro de la configuración principal de Jenkins. Para esto, ingrese a la página principal de Jenkins en donde dentro de la configuración del sistema encontrará la sección _Publish Over SSH_ aquí deberá pegar la clave privada no cifrada dentro del recuadro _key_. Adicionalmente, se deberán llenar los siguientes recuadros para SSH Server.

- Passphrase: Frase de contraseña para la clave privada, si está encriptada
- Name: Nombre del usuario que utilizará para conectarse al host.
- Hostname: Nombre de host SSH o dirección IP.
- Username: Nombre de usuario (mismo name)
- Port: 22 (Este es el puerto predeterminado para SSH)
- Timeout: 300000 (Tiempo de espera en milisegundos para las conexiones SSH)

Una vez terminado guarde los cambios realizados.

### 4.Conexión SSH con la máquina de producción
Se instala SSH Server en la terminal de Ubuntu
```
sudo apt-get install openssh-server
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
pass: ****
```
Una vez ya instalado el servidor OpenSSH se accede al repositorio, pero antes es importante otorgarle permisos permanentes a Jenkins de conectarse al repositorio gestionando las credenciales ssh de usuario. Es por esta razón que se creará la clave SSH para el usuario de Jenkins utilizando el siguiente comando:
```
sudo su-jenkins
ssh-keygen -t rsa -b 2048
```
Para poder acceder al repositorio de forma constante se deberá crear un usuario, tal como se describe en los comando a continuación
```
sudo adduser git #password
su git
mkdir .ssh
```
Se debe crear una clave SSH con el propósito de facilitar los inicios de sesión automatizados y sin contraseña entre ambas máquinas. Para generar la clave ssh utilice el sieguiente comando:
```sh
ssh-keygen
```
_id_rsa_ es la clave privada, _id_rsa.pub_ es la clave pública. Una vez generado el par de claves hay que copiar la clave pública a la máquina de producción:
```sh
 scp ~/.ssh/id_rsa.pub deploy@ip_produccion:/home/user/uploaded_key.pub
```
En la máquina de producción, si la carpeta .ssh no existe, se debe crear junto con el archivo authorized_keys, usando los siguientes comandos:
```sh
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```
Por último, en la máquina de producción copiamos la clave enviada por la máquina de integración y borramos el archivo copiado:
```sh
echo `cat ~/uploaded_key.pub` >> ~/.ssh/authorized_keys
rm /home/user/uploaded_key.pub
```
## Entorno de producción :computer: 
---
Se aprovisiona una máquina en skytap haciendo uso de la plantilla cuyo sistema operativo es ubuntu. Para el despliegue de la aplicación se desarrolla el siguiente procedimiento:
