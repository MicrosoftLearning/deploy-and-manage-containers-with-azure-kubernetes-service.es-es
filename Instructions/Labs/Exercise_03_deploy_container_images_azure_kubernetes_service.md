---
Exercise:
  title: 'Ejercicio: Implementar imágenes de contenedor en Azure Kubernetes Service'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Ejercicio 3: Implementar aplicaciones en Azure Kubernetes Service (AKS)

## Objetivos

Este proyecto guiado consta de los ejercicios siguientes:

+ Ejercicio 1: Aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS).
+ Ejercicio 2: Compilar imágenes de contenedor de Linux y Windows, y almacenarlas en Azure Container Registry.
+ **Ejercicio 3: Implementar imágenes de contenedor en Azure Kubernetes Service.**
+ Ejercicio 4: Revisar la implementación y desaprovisionar todos los recursos.

En este ejercicio implementa imágenes de contenedor en Azure Kubernetes Service.

## Ejercicio 3: Implementar imágenes de contenedor en AKS 
En este ejercicio implementa dos imágenes de contenedor que creó antes en el ejercicio en el clúster de AKS.
>**Nota:** Para realizar este ejercicio, necesita una [suscripción de Azure](https://azure.microsoft.com/free/).
> Para cualquier propiedad que no se especifique, use el valor predeterminado.
> **Nota:** Antes de continuar con este ejercicio, asegúrese de que se ha completado correctamente el aprovisionamiento del clúster de AKS.

### Tarea 1: Crear espacios de nombres de AKS personalizados
En esta tarea crea dos espacios de nombres en el clúster de AKS que creó anteriormente en este laboratorio.

1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Servicios de Kubernetes**.
1. En la página **Servicios de Kubernetes**, seleccione **aks-01**.
1. En la página **aks-01**, en el menú vertical, seleccione **Espacios de nombres**.
1. En la página **aks-01 \| Espacios de nombres**, seleccione **+ Crear** y, en el menú desplegable, seleccione **Espacio de nombres**.
1. En el panel **Crear un espacio de nombres**, en el cuadro de texto **Nombre**, escriba **dev-node** y seleccione **Crear**.
1. En la página **aks-01 \| Espacios de nombres**, seleccione **+ Crear** y, en el menú desplegable, seleccione **Espacio de nombres**.
1. En el panel **Crear un espacio de nombres**, en el cuadro de texto **Nombre**, escriba **dev-dotnet** y seleccione **Crear**.

### Tarea 2: Crear un manifiesto de Kubernetes para implementar la imagen de Linux
En esta tarea crea un manifiesto de Kubernetes para implementar la imagen de Linux en el grupo de nodos de Linux.

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

   > **Nota:** La implementación creará pods basados en la imagen de contenedor de Linux en el grupo de nodos de Linux en el clúster de AKS. Además, el manifiesto incluye un servicio que proporcionará acceso con equilibrio de carga a los pods de la implementación a través de una dirección IP pública en el puerto 80.

1. Guarde los cambios del archivo y ciérrelo para volver al símbolo del sistema de Bash.
1. En la sesión de Bash de Azure Cloud Shell, reemplace el marcador de posición **ACR_NAME** en el archivo aks-deployment-l01.yaml ejecutando los siguientes comandos:

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-l01.yaml
   ```

### Tarea 3: Crear un manifiesto de Kubernetes para implementar la imagen de Windows
En esta tarea, creará un manifiesto de Kubernetes para implementar la imagen de Windows en el grupo de nodos de Windows.

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

   > **Nota:** La implementación creará pods basados en la imagen de contenedor de Windows en el grupo de nodos de Windows en el clúster de AKS. Además, el manifiesto incluye un servicio que proporcionará acceso con equilibrio de carga a los pods de la implementación a través de una dirección IP pública en el puerto 80.

1. Guarde los cambios del archivo y ciérrelo para volver al símbolo del sistema de Bash.
1. En la sesión de Bash de Azure Cloud Shell, reemplace el marcador de posición **ACR_NAME** en el archivo aks-deployment-l01.yaml ejecutando el siguiente comando:

   ```azurecli
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-w01.yaml
   ```

### Tarea 4: Realizar implementaciones de AKS usando archivos de manifiesto de YAML
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

   > **Nota:** La salida del comando debe incluir la lista de todos los nodos de AKS (cuatro en este caso).

1. En la sesión de Bash de Azure Cloud Shell, cree la primera implementación definida en el archivo de manifiesto YAML correspondiente en el espacio de nombres **dev-node** ejecutando los siguientes comandos:

   ```kubectl
   cd ~/aks-l01
   kubectl apply -f aks-deployment-l01.yaml -n=dev-node
   ```

   > **Nota:** Continúe con el paso siguiente sin esperar a que se complete la implementación. El aprovisionamiento de todos los recursos puede tardar unos minutos.

1. En la sesión de Bash de Azure Cloud Shell, cree la segunda implementación definida en el archivo de manifiesto YAML correspondiente ejecutando el siguiente comando:

   ```kubectl
   cd ~/aks-w01
   kubectl apply -f aks-deployment-w01.yaml -n=dev-dotnet
   ```

   > **Nota:** Continúe con el paso siguiente sin esperar a que se complete la implementación. El aprovisionamiento de todos los recursos puede tardar unos minutos.

