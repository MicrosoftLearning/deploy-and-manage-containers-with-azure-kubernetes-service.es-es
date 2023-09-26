---
Exercise:
  title: 'Ejercicio: Revisar la implementación y desaprovisionar todos los recursos'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Ejercicio 4: Implementar aplicaciones en Azure Kubernetes Service (AKS)

## Objetivos

Este proyecto guiado consta de los ejercicios siguientes:

+ Ejercicio 1: Aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS)
+ Ejercicio 2: Compilar imágenes de contenedor de Linux y Windows y almacenarlas en ACR
+ Ejercicio 3: Implementar imágenes de contenedor en AKS
+ **Ejercicio 4: Revisar la implementación y desaprovisionar todos los recursos**

En este ejercicio, revisará la implementación de los servicios en los ejercicios anteriores y desaprovisionará todos los recursos.

# Ejercicio 4: Revisar la implementación y desaprovisionar todos los recursos
En este ejercicio, revisará los resultados de las implementaciones y desaprovisionará todos los recursos.

>**Nota:** Para realizar este ejercicio, necesita una [suscripción de Azure](https://azure.microsoft.com/free/).
> Para cualquier propiedad que no se especifique, use el valor predeterminado.

### Tarea 1: Revisar las implementaciones y servicios de AKS
En esta tarea, revisará los resultados de ambas implementaciones, incluidos los objetos de implementaciones y servicios.

1. En la sesión Bash de Azure Cloud Shell, muestre el estado de ambas implementaciones mediante la ejecución de los siguientes comandos:

   ```kubectl
   kubectl get deployments -n=dev-node
   kubectl get deployments -n=dev-dotnet
   ```

   > **Nota:** Antes de continuar con el paso siguiente, compruebe que ambas implementaciones se muestran con el estado listo. Si no es así, espere otro minuto, vuelva a ejecutar los dos comandos mencionados anteriormente y vuelva a comprobar el estado de la implementación.

1. En la sesión de Bash de Azure Cloud Shell, muestre el estado de los dos servicios que se incluyeron en los archivos de manifiesto mediante la ejecución de los siguientes comandos:

   ```kubectl
   kubectl get services -n=dev-node
   kubectl get services -n=dev-dotnet
   ```

1. Compruebe que la lista de cada servicio incluye un valor en la columna **EXTERNAL-IP**. 
1. Use un explorador web para navegar a las direcciones IP que identificó en el paso anterior y compruebe que las páginas web resultantes muestran los mensajes **Hello World from Node** y **Hello World from .Net 7**, respectivamente.

### Tarea 2: Eliminar todos los recursos
En esta tarea, eliminará todos los recursos aprovisionados en este laboratorio.

1. En la sesión de Bash de Azure Cloud Shell, muestre la lista de recursos en los dos grupos de recursos aprovisionados en este ejercicio mediante la ejecución de los siguientes comandos:

   ```azurecli
   az resource list --resource-group 'acr-01-RG' --query "[].name" --output tsv
   az resource list --resource-group 'aks-01-RG' --query "[].name" --output tsv
   ```

   > **Nota:** Compruebe que son los recursos que desea eliminar. Si es así, continúe con el siguiente paso.

1. En la sesión de Bash de Azure Cloud Shell, ejecute los siguientes comandos para eliminar todos los recursos aprovisionados en este ejercicio:

   ```azurecli
   az group delete --name 'acr-01-RG' --no-wait --yes
   az group delete --name 'aks-01-RG' --no-wait --yes
   ```

   > **Nota:** El comando se ejecuta de forma asíncrona (tal y como impone el parámetro --nowait), por lo que, aunque ambos comandos vuelvan inmediatamente a la línea de comandos de Bash, pasarán unos minutos antes de que se eliminen los grupos de recursos y sus recursos.

1. Cierre el panel de Azure Cloud Shell.
