---
Guided Exercise:
  title: 'Ejercicio guiado: implementar aplicaciones en AKS'
---
# Ejercicio guiado: implementar aplicaciones en Azure Kubernetes Service (AKS)

## Objetivos

Este ejercicio guiado consta de las siguientes actividades:

+ Ejercicio 1: aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS)
+ Ejercicio 2: compilar imágenes de contenedor de Linux y Windows, y almacenarlas en ACR
+ Ejercicio 3: implementar imágenes de contenedor en AKS 
+ Ejercicio 4: revisar la implementación y desaprovisionar todos los recursos

## Ejercicio 1: aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS)
En este ejercicio, creará una instancia de Azure Container Registry y un clúster de AKS.


>**Nota:** para realizar este ejercicio, necesita una [suscripción de Azure](https://azure.microsoft.com/free/).
> Para cualquier propiedad que no se especifique, use el valor predeterminado.


### Tarea 1: crear un Azure Container Registry
En esta tarea, creará una instancia de Azure Container Registry

1. En el equipo, abra una ventana del explorador web y vaya a Azure Portal en https://portal.azure.com.
1. Cuando se le solicite, inicie sesión con una cuenta de usuario que tenga el rol Propietario en la suscripción de Azure que usará en este ejercicio. 
1. Inicie sesión en Azure Portal. 
1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Registros de contenedor**.
1. En la página **Registros de contenedor**, seleccione **+ Crear** y especifique la siguiente configuración:

    |Configuración|Value|
    |---|---|
    |Subscription|Nombre de la suscripción de Azure que usará en este ejercicio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **acr-01-RG**|
    |Nombre de registro|Cualquier nombre válido y único a nivel global que conste de entre 5 y 50 caracteres alfanuméricos|
    |Region|Cualquier región de Azure en la que pueda crear una instancia de Azure Container Registry y un clúster de AKS|
    |Zonas de disponibilidad|**None**|
    |SKU|**Basic**|

1. En la página **Registros de contenedor**, seleccione **Revisar y crear** y, en la pestaña **Revisar y crear**, seleccione **Crear**.

   > **Nota:** continúe con el ejercicio siguiente sin esperar a que se complete el aprovisionamiento de Azure Container Registry.

### Tarea 2: crear una red virtual de Azure y un clúster de AKS
En esta tarea, creará una red virtual de Azure e implementará un clúster de AKS, incluido un grupo de nodos de Windows en esa red virtual.

> **Nota:** aunque podría crear una red virtual al aprovisionar un clúster de AKS, no es un escenario típico. Y lo que es más importante, la implementación de clústeres de AKS en una red virtual existente requiere algunas consideraciones adicionales con las que debe estar familiarizado.

1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Redes virtuales**.
1. En la página **Redes virtuales**, seleccione **+ Crear** y, a continuación, en la pestaña **Aspectos básicos** de la página **Crear red virtual**, especifique la siguiente configuración:

    |Configuración|Value|
    |---|---|
    |Subscription|Nombre de la suscripción de Azure seleccionada en el primer ejercicio|
    |Grupo de recursos|Nombre de un nuevo grupo de recursos **aks-01-RG**|
    |Nombre de la red virtual|**vnet-01**|
    |Region|La misma región de Azure que seleccionó en el primer ejercicio de este ejercicio|

1. En la pestaña **Aspectos básicos** de la página **Crear red virtual**, seleccione **Siguiente**.
1. En la pestaña **Seguridad** de la página **Crear red virtual**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la pestaña **Direcciones IP** de la página **Crear red virtual**, asegúrese de que el espacio de direcciones IP esté establecido en **10.0.0.0/16**, elimine la subred **predeterminada**, seleccione **Revisar y crear** y, en la pestaña **Revisar y crear**, seleccione **Crear**.

   > **Nota:** la creación de una red virtual solo debe tardar unos segundos, por lo que debería poder pasar directamente al siguiente paso.

1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Servicios de Kubernetes**.
1. En la página **Servicios de Kubernetes**, seleccione **+ Crear**, en la lista desplegable, seleccione **Crear un clúster de Kubernetes** y, a continuación, en la pestaña **Aspectos básicos** de la página **Crear clúster de Kubernetes**, especifique la siguiente configuración:

    |Configuración|Value|
    |---|---|
    |Subscription|Nombre de la suscripción de Azure seleccionada en el primer ejercicio|
    |Grupo de recursos|**aks-01-RG**|
    |Configuración preestablecida de clúster|**Desarrollo/pruebas**|
    |Nombre del clúster de Kubernetes|**aks-01**|
    |Region|La misma región de Azure que seleccionó en el primer ejercicio|
    |Zonas de disponibilidad|**None**|
    |Plan de tarifa de AKS|**Gratis**|
    |Versión de Kubernetes|Acepte el valor predeterminado|
    |Actualización automática|Disabled|
    |Tamaño del nodo|**B4ms estándar**|
    |Método de escala|**Manual**|
    |Recuento de nodos|**2**|

   > **Nota:** es posible que tenga que aumentar las cuotas de vCPU o cambiar la SKU de la máquina virtual para dar cabida a los valores de tamaño de nodo y número de nodos. Para obtener información sobre el procedimiento para aumentar las cuotas de vCPU, consulte el artículo de Microsoft Learn [Aumentar las cuotas de vCPU de una familia de máquinas virtuales](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests).

1. En la pestaña **Aspectos básicos** de la página **Crear clúster de Kubernetes**, seleccione **Siguiente: Grupos de nodos >** .
1. En la pestaña **Grupos de nodos** de la página **Crear clúster de Kubernetes**, seleccione **Siguiente: Acceso >** .
1. En la pestaña **Acceso** de la página **Crear clúster de Kubernetes**, seleccione **Siguiente: Redes >** .
1. En la pestaña **Redes** de la página **Crear clúster de Kubernetes**, seleccione la opción **Azure CNI**, en la lista desplegable **Red virtual**, seleccione **vnet-01** y, debajo del cuadro de texto **Subred de clúster**, seleccione **Configuración de subred administrada**.
1. En la página **vnet-01 \| Subredes**, seleccione **+ Subred**.
1. En la página **Agregar subredes**, especifique la siguiente configuración y seleccione **Guardar**:

    |Configuración|Value|
    |---|---|
    |Nombre|**aks-subnet**|
    |Intervalo de direcciones de subred|**10.0.0.0/20**|

1. De nuevo en la página **vnet-01 \| Subredes**, en la ruta de navegación de la parte superior izquierda de la página, seleccione **Crear clúster de Kubernetes**. 
1. De nuevo en la pestaña **Redes** de la página **Crear clúster de Kubernetes**, especifique la siguiente configuración:

    |Configuración|Value|
    |---|---|
    |Subred de clústeres|**aks-subnet (10.0.0.0/20)**|
    |Intervalo de direcciones del servicio de Kubernetes|**172.16.0.0/22**|
    |Dirección IP del servicio DNS de Kubernetes|**172.16.3.254**|
    |Prefijo del nombre DNS|**aks-01-dns**|
    |Habilitar clúster privado|Disabled|
    |Establecer intervalos IP autorizados|Disabled|
    |Directiva de red|**None**|

1. En la pestaña **Redes** de la página **Crear clúster de Kubernetes**, seleccione la pestaña **Grupos de nodos**.

   > **Nota:** agregará un grupo de nodos de Windows al clúster. Esto requiere cambiar la configuración de red a **Azure CNI** del **kubenet** predeterminado. La configuración de red de kubenet no admite grupos de nodos de Windows.

1. En la pestaña **Grupos de nodos** de la página **Crear clúster de Kubernetes**, seleccione **+Agregar grupo de nodos**.
1. En la página **Agregar grupo de nodos**, especifique la siguiente configuración:

    |Configuración|Value|
    |---|---|
    |Nombre del grupo de nodos|**w1pool**|
    |Modo|**Usuario**|
    |Tipo de SO|**Windows**|
    |Zona de disponibilidad|**None**|
    |Habilitar instancias de Azure Spot|Disabled|
    |Tamaño del nodo|**Standard B4s_v2**|
    |Método de escala|**Manual**|
    |Recuento de nodos|**2**|
    |Máximo de pods por nodo|**30**|
    |Habilitar IP pública por nodo|Disabled|

   > **Nota:** es posible que tenga que aumentar las cuotas de vCPU o cambiar la SKU de la máquina virtual para dar cabida a los valores de tamaño de nodo y número de nodos. Para obtener información sobre el procedimiento para aumentar las cuotas de vCPU, consulte el artículo de Microsoft Learn [Aumentar las cuotas de vCPU de una familia de máquinas virtuales](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests).

1. En la página **Agregar grupo de nodos**, seleccione **Agregar**.
1. De nuevo en la pestaña **Grupos de nodos** de la página **Crear clúster de Kubernetes**, seleccione la pestaña **Integraciones**.
1. En la pestaña **Integración** de la página **Crear clúster de Kubernetes**, en la lista desplegable **Registro de contenedor**, seleccione la entrada que representa la instancia de Azure Container Registry que creó en el ejercicio anterior, deshabilite la casilla **Habilitar reglas de alerta recomendadas**, asegúrese de que la opción **Azure Policy** está deshabilitada y seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear** de la página **Crear clúster de Kubernetes**, seleccione **Crear**.

   > **Nota:** continúe con el ejercicio siguiente sin esperar a que se complete el aprovisionamiento del clúster de AKS. El proceso de aprovisionamiento puede tardar unos cinco minutos.

## Ejercicio 2: compilar imágenes de contenedor de Linux y Windows, y almacenarlas en ACR
En este ejercicio, creará imágenes de Docker basadas en Linux y Windows y las insertará en la instancia de Azure Container Registry que creó anteriormente en este ejercicio.

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

## Ejercicio 3: implementar imágenes de contenedor en AKS 
En este ejercicio implementa dos imágenes de contenedor que creó antes en el ejercicio en el clúster de AKS.

> **Nota:** antes de continuar con este ejercicio, asegúrese de que se ha completado correctamente el aprovisionamiento del clúster de AKS.

### Tarea 1: crear espacios de nombres de AKS personalizados
En esta tarea crea dos espacios de nombres en el clúster de AKS que creó anteriormente en este ejercicio.

1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Servicios de Kubernetes**.
1. En la página **Servicios de Kubernetes**, seleccione **aks-01**.
1. En la página **aks-01**, en el menú vertical, seleccione **Espacios de nombres**.
1. En la página **aks-01 \| Espacios de nombres**, seleccione **+ Crear** y, en el menú desplegable, seleccione **Espacio de nombres**.
1. En el panel **Crear un espacio de nombres**, en el cuadro de texto **Nombre**, escriba **dev-node** y seleccione **Crear**.
1. En la página **aks-01 \| Espacios de nombres**, seleccione **+ Crear** y, en el menú desplegable, seleccione **Espacio de nombres**.
1. En el panel **Crear un espacio de nombres**, en el cuadro de texto **Nombre**, escriba **dev-dotnet** y seleccione **Crear**.

### Tarea 2: crear un manifiesto de Kubernetes para implementar la imagen de Linux
En esta tarea crea un manifiesto de Kubernetes para implementar la imagen de Linux en el grupo de nodos de Linux

1. En Azure Portal, seleccione el icono de **Cloud Shell**.
1. Asegúrese de que aparece **Bash** en el menú desplegable de la esquina superior izquierda del panel de Cloud Shell.
1. En la sesión de Bash de Azure Cloud Shell, cree un directorio para hospedar el manifiesto de implementación para aprovisionar pods basados en la imagen de Linux. Cambie a ese directorio desde el directorio actual ejecutando los siguientes comandos:

   ```bash
   mkdir ~/aks-l01
   cd ~/aks-l01
   ```

1. En la sesión de Bash de Azure Cloud Shell, use el editor integrado para crear un archivo denominado aks-deployment-l01.yaml en el directorio aks-l01 y copie en él el siguiente contenido:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromnode-deployment
     labels:
       environment: dev
       app: hellofromnode
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromnode
         labels:
           app: hellofromnode
       spec:
         nodeSelector:
           "kubernetes.io/os": linux
         containers:
         - name: hellofromnode
           image: ACR_NAME.azurecr.io/hellofromnode:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromnode
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromnode-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromnode
   ```

   > **Nota:** la implementación creará pods basados en la imagen de contenedor de Linux en el grupo de nodos de Linux en el clúster de AKS. Además, el manifiesto incluye un servicio que proporcionará acceso con equilibrio de carga a los pods de la implementación a través de una dirección IP pública en el puerto 80.

1. Guarde los cambios del archivo y ciérrelo para volver al símbolo del sistema de Bash.
1. En la sesión de Bash de Azure Cloud Shell, reemplace el marcador de posición **ACR_NAME** en el archivo aks-deployment-l01.yaml ejecutando los siguientes comandos:

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-l01.yaml
   ```

### Tarea 3: crear un manifiesto de Kubernetes para implementar la imagen de Windows
En esta tarea, creará un manifiesto de Kubernetes para implementar la imagen de Windows en el grupo de nodos de Windows

1. En la sesión de Bash de Azure Cloud Shell, cree un directorio para hospedar el manifiesto de implementación para aprovisionar pods basados en la imagen de Windows. Cambie a ese directorio desde el directorio actual ejecutando los siguientes comandos:

   ```bash
   mkdir ~/aks-w01
   cd ~/aks-w01
   ```

1. En la sesión de Bash de Azure Cloud Shell, use el editor integrado para crear un archivo denominado aks-deployment-w01.yaml en el directorio aks-w01 y copie en él el siguiente contenido:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromdotnet-deployment
     labels:
       environment: dev
       app: hellofromdotnet
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromdotnet
         labels:
           app: hellofromdotnet
       spec:
         nodeSelector:
           "kubernetes.io/os": windows
         containers:
         - name: hellofromdotnet
           image: ACR_NAME.azurecr.io/hellofromdotnet:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromdotnet
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromdotnet-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromdotnet
   ```

   > **Nota:** la implementación creará pods basados en la imagen de contenedor de Windows en el grupo de nodos de Windows en el clúster de AKS. Además, el manifiesto incluye un servicio que proporcionará acceso con equilibrio de carga a los pods de la implementación a través de una dirección IP pública en el puerto 80.

1. Guarde los cambios del archivo y ciérrelo para volver al símbolo del sistema de Bash.
1. En la sesión de Bash de Azure Cloud Shell, reemplace el marcador de posición **ACR_NAME** en el archivo aks-deployment-l01.yaml ejecutando el siguiente comando:

   ```azurecli
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-w01.yaml
   ```

### Tarea 4: realizar implementaciones de AKS usando archivos de manifiesto de YAML
En esta tarea implementa imágenes de contenedor en sus respectivos espacios de nombres y grupos de nodos en el clúster de AKS de destino.

1. En la sesión de Bash de Azure Cloud Shell, conéctese al clúster de AKS ejecutando los siguientes comandos:

   ```azurecli
   AKSRG='aks-01-RG'
   AKSNAME='aks-01'
   az aks get-credentials --resource-group $AKSRG --name $AKSNAME
   ```

1. En la sesión de Bash de Azure Cloud Shell, ejecute el siguiente comando para comprobar que la conexión se ha establecido correctamente:

   ```kubectl
   kubectl get nodes
   ```

   > **Nota:** la salida del comando debe incluir la lista de todos los nodos de AKS (cuatro en este caso).

1. En la sesión de Bash de Azure Cloud Shell, cree la primera implementación definida en el archivo de manifiesto YAML correspondiente en el espacio de nombres **dev-node** ejecutando los siguientes comandos:

   ```kubectl
   cd ~/aks-l01
   kubectl apply -f aks-deployment-l01.yaml -n=dev-node
   ```

   > **Nota:** continúe con el paso siguiente sin esperar a que se complete la implementación. El aprovisionamiento de todos los recursos puede tardar unos minutos.

1. En la sesión de Bash de Azure Cloud Shell, cree la segunda implementación definida en el archivo de manifiesto YAML correspondiente ejecutando el siguiente comando:

   ```kubectl
   cd ~/aks-w01
   kubectl apply -f aks-deployment-w01.yaml -n=dev-dotnet
   ```

   > **Nota:** continúe con el paso siguiente sin esperar a que se complete la implementación. El aprovisionamiento de todos los recursos puede tardar unos minutos.

# Ejercicio 4: revisar la implementación y desaprovisionar todos los recursos
En este ejercicio, revisará los resultados de las implementaciones y desaprovisionará todos los recursos.

### Tarea 1: revisar las implementaciones y servicios de AKS
En esta tarea, revisará los resultados de ambas implementaciones, incluidos los objetos de implementaciones y servicios.

1. En la sesión Bash de Azure Cloud Shell, muestre el estado de ambas implementaciones mediante la ejecución de los siguientes comandos:

   ```kubectl
   kubectl get deployments -n=dev-node
   kubectl get deployments -n=dev-dotnet
   ```

   > **Nota:** antes de continuar con el paso siguiente, compruebe que ambas implementaciones se muestran con el estado listo. Si no es así, espere otro minuto, vuelva a ejecutar los dos comandos mencionados anteriormente y vuelva a comprobar el estado de la implementación.

1. En la sesión de Bash de Azure Cloud Shell, muestre el estado de los dos servicios que se incluyeron en los archivos de manifiesto mediante la ejecución de los siguientes comandos:

   ```kubectl
   kubectl get services -n=dev-node
   kubectl get services -n=dev-dotnet
   ```

1. Compruebe que la lista de cada servicio incluye un valor en la columna **EXTERNAL-IP**. 
1. Use un explorador web para navegar a las direcciones IP que identificó en el paso anterior y compruebe que las páginas web resultantes muestran los mensajes **Hello World from Node** y **Hello World from .Net 7**, respectivamente.

### Tarea 2: eliminar todos los recursos
En esta tarea, eliminará todos los recursos aprovisionados en este ejercicio.

1. En la sesión de Bash de Azure Cloud Shell, muestre la lista de recursos en los dos grupos de recursos aprovisionados en este ejercicio mediante la ejecución de los siguientes comandos:

   ```azurecli
   az resource list --resource-group 'acr-01-RG' --query "[].name" --output tsv
   az resource list --resource-group 'aks-01-RG' --query "[].name" --output tsv
   ```

   > **Nota:** compruebe que son los recursos que desea eliminar. Si es así, continúe con el siguiente paso.

1. En la sesión de Bash de Azure Cloud Shell, ejecute los siguientes comandos para eliminar todos los recursos aprovisionados en este ejercicio:

   ```azurecli
   az group delete --name 'acr-01-RG' --no-wait --yes
   az group delete --name 'aks-01-RG' --no-wait --yes
   ```

   > **Nota:** el comando se ejecuta de forma asíncrona (tal y como impone el parámetro --nowait), por lo que aunque ambos comandos vuelvan inmediatamente a la línea de comandos de Bash, pasarán unos minutos antes de que se eliminen los grupos de recursos y sus recursos.

1. Cierre el panel de Azure Cloud Shell.
