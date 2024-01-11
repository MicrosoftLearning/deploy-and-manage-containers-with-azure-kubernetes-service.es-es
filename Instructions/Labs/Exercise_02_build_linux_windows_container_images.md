---
Exercise:
  title: 'Ejercicio: compilar imágenes de contenedor de Linux y Windows, y almacenarlas en Azure Container Registry'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Ejercicio 2: implementar aplicaciones en Azure Kubernetes Service (AKS)

## Objetivos

Este proyecto guiado consta de los ejercicios siguientes:

+ Ejercicio 1: Aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS).
+ **Ejercicio 2: compilar imágenes de contenedor de Linux y Windows, y almacenarlas en Azure Container Registry.**
+ Ejercicio 3: Implementar imágenes de contenedor en Azure Kubernetes Service.
+ Ejercicio 4: revisar la implementación y desaprovisionar todos los recursos.

En este ejercicio compila imágenes de contenedor de Linux y Windows, y almacenarlas en Azure Container Registry.

## Ejercicio 2: compilar imágenes de contenedor de Linux y Windows y almacenarlas en ACR
En este ejercicio, creará imágenes de Docker basadas en Linux y Windows y las insertará en la instancia de Azure Container Registry que creó anteriormente en este laboratorio.


>**Nota:** para realizar este ejercicio, necesita una [suscripción de Azure](https://azure.microsoft.com/free/).
> Para cualquier propiedad que no se especifique, use el valor predeterminado.


### Tarea 1: compilar una imagen de contenedor de Linux y almacenarla en ACR
En esta tarea, usará una tarea de ACR para compilar una imagen de contenedor de Linux e insertarla automáticamente en el ACR.

1. En Azure Portal, seleccione el icono de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**. 
1. Si se le solicite, seleccione **Crear almacenamiento** y espere hasta que aparezca Azure Cloud Shell. 
1. Asegúrese de que aparece **Bash** en el menú desplegable de la esquina superior izquierda del panel de Cloud Shell.
1. Desde la sesión de Bash en Cloud Shell, cree un directorio que hospede el Dockerfile para la imagen de Linux y cámbielo desde el directorio actual mediante la ejecución de los siguientes comandos:

   ```bash
   mkdir ~/image-l01
   cd ~/image-l01
   ```

1. En la sesión de Bash de Azure Cloud Shell, use el editor integrado para crear un archivo denominado server.js en el directorio image-l01 y copie en él el siguiente contenido:

   ```js
   const http = require('http')
   const port = 80
   const server = http.createServer((request, response) => {
     response.writeHead(200, {'Content-Type': 'text/plain'})
     response.write('Hello World from Node\n')
     response.end('Version: ' + process.env.NODE_VERSION + '\n')
   })
   server.listen(port)
   console.log(`Server running at http://localhost: ${port}`)
   ```

   > **Nota:** una vez en ejecución, el código de Node.js resultante mostrará el mensaje **Hello World from Node**.

1. En la sesión de Bash de Azure Cloud Shell, vuelva a usar el editor integrado para crear un archivo denominado package.json en el directorio image-l01 y copie en él el siguiente contenido:

   ```json
   {
     "name": "helloworld",
     "version": "1.0.0",
     "description": "Sample app for ACR Build",
     "main": "server.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "start": "node server.js"
     },
     "license": "MIT"
   }
   ```

1. Guarde los cambios del archivo y ciérrelo para volver al símbolo del sistema de Bash.
1. En la sesión de Bash de Azure Cloud Shell, vuelva a usar el editor integrado para crear un archivo denominado Dockerfile en el directorio image-l01 y copie en él el siguiente contenido:

   ```Dockerfile
   FROM node:20.2-alpine
   COPY . /src
   RUN cd /src && npm install
   EXPOSE 80
   CMD ["node", "/src/server.js"]
   ```

1. Guarde los cambios del archivo y ciérrelo para volver al símbolo del sistema de Bash.
1. En la sesión de Bash de Azure Cloud Shell, identifique el nombre de Azure Container Registry que creó anteriormente en este ejercicio y almacénelo en una variable denominada **$ACRNAME** mediante la ejecución de los siguientes comandos:

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   ```

1. En la sesión de Bash de Azure Cloud Shell, cree una imagen de Docker basada en el Dockerfile almacenado en el directorio actual e insértela automáticamente en Azure Container Registry cuyo nombre está almacenado en la variable **$ACRNAME** mediante la ejecución del siguiente comando (asegúrese de incluir el punto final):

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromnode:v1.0 .
   ```

   > **Nota:** realice un seguimiento del progreso de la compilación y asegúrese de que se completa correctamente. Debería tardar menos de un minuto.

### Tarea 2: compilar una imagen de contenedor de Windows y almacenarla en ACR
En esta tarea, usará una tarea de ACR para compilar una imagen de contenedor de Windows e insertarla automáticamente en el ACR.

1. Desde la sesión de Bash en Cloud Shell, cree un directorio que hospede el Dockerfile para la imagen de Windows y cámbielo desde el directorio actual mediante la ejecución de los siguientes comandos:

   ```bash
   mkdir ~/image-w01
   cd ~/image-w01
   ```

1. Desde la sesión de Bash de Azure Cloud Shell, clone un repositorio público de GitHub que hospede los archivos que usará para compilar la imagen de Windows y cambie el directorio actual al repositorio clonado mediante la ejecución de los siguientes comandos:

   ```git
   git clone https://github.com/Azure-Samples/dotnetcore-docs-hello-world.git
   cd ~/image-w01/dotnetcore-docs-hello-world
   ```

   > **Nota:** el repositorio contiene el código fuente de una aplicación web de .NET 7 que muestra el mensaje **Hello World from .NET 7**.

1. En la sesión de Bash de Azure Cloud Shell, cree una imagen de Docker basada en el Dockerfile almacenado en el directorio actual e insértela automáticamente en Azure Container Registry cuyo nombre está almacenado en la variable **$ACRNAME** mediante la ejecución del siguiente comando (asegúrese de incluir el punto final):

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromdotnet:v1.0 --platform windows --file Dockerfile.windows .
   ```

   > **Nota:** el nombre del Dockerfile que define la compilación de la imagen de Windows es **Dockerfile.windows**.

   > **Nota:** realice un seguimiento del progreso de la compilación y asegúrese de que se completa correctamente. Debería tardar menos de 3 minutos.

1. Cierre el panel de Azure Cloud Shell.
1. En Azure Portal, vaya a la página **Registros** de contenedor y seleccione la entrada que representa el registro de contenedor en el que insertó ambas imágenes.
1. En la página Registro de contenedor, en el menú del hub vertical, seleccione **Repositorios** y compruebe que **hellofromnode** y **hellofromdotnet** aparecen en la lista de repositorios.
