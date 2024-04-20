---
Exercise:
  title: 'Ejercicio: aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS)'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---

# Ejercicio: implementar aplicaciones en Azure Kubernetes Service (AKS)

## Objetivos

Este proyecto guiado consta de los ejercicios siguientes:

+ **Ejercicio 1: aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS).**
+ Ejercicio 2: Compilar imágenes de contenedor de Linux y Windows, y almacenarlas en Azure Container Registry.
+ Ejercicio 3: Implementar imágenes de contenedor en Azure Kubernetes Service.
+ Ejercicio 4: revisar la implementación y desaprovisionar todos los recursos.

En este ejercicio, aprovisiona recursos de Azure Container Registry y Azure Kubernetes Service.
## Ejercicio 1: aprovisionar Azure Container Registry (ACR) y Azure Kubernetes Service (AKS)
En este ejercicio, creará una instancia de Azure Container Registry y un clúster de AKS.

>**Nota:** para realizar este ejercicio, necesita una [suscripción de Azure](https://azure.microsoft.com/free/).
> Para cualquier propiedad que no se especifique, use el valor predeterminado.

### Tarea 1: crear un Azure Container Registry
En esta tarea, creará una instancia de Azure Container Registry

1. En el equipo, abra una ventana del explorador web y vaya a Azure Portal en https://portal.azure.com.
1. Cuando se le solicite, inicie sesión con una cuenta de usuario que tenga el rol Propietario en la suscripción de Azure que usará en este laboratorio. 
1. Inicie sesión en Azure Portal. 
1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Registros de contenedor**.
1. En la página **Registros de contenedor**, seleccione **+ Crear** y especifique la siguiente configuración:

   |Configuración|Valor|
   |---|---|
   |Subscription|Nombre de la suscripción de Azure que usará en este laboratorio|
   |Grupo de recursos|Nombre de un nuevo grupo de recursos **acr-01-RG**|
   |Nombre de registro|Cualquier nombre válido y único a nivel global que conste de entre 5 y 50 caracteres alfanuméricos|
   |Location|Cualquier región de Azure en la que pueda crear una instancia de Azure Container Registry y un clúster de AKS|
   |Uso de zonas de disponibilidad|Deshabilitado|
   |Plan de precios|**Basic**|

1. En la página **Registros de contenedor**, seleccione **Revisar y crear** y, en la pestaña **Revisar y crear**, seleccione **Crear**.

   > **Nota:** continúe con el ejercicio siguiente sin esperar a que se complete el aprovisionamiento de Azure Container Registry.

### Tarea 2: crear una red virtual de Azure y un clúster de AKS
En esta tarea, creará una red virtual de Azure e implementará en ella un clúster de AKS que incluye un grupo de nodos de Windows.

> **Nota:** Aunque es posible crear una red virtual al aprovisionar un clúster de AKS, la implementación de clústeres de AKS en una red virtual existente requiere algunas consideraciones adicionales con las que debe estar familiarizado.

1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Redes virtuales**.
1. En la página **Redes virtuales**, seleccione **+ Crear** y, a continuación, en la pestaña **Aspectos básicos** de la página **Crear red virtual**, especifique la siguiente configuración:

   |Configuración|Valor|
   |---|---|
   |Subscription|Nombre de la suscripción de Azure seleccionada en el primer ejercicio de este laboratorio|
   |Grupo de recursos|Nombre de un nuevo grupo de recursos **aks-01-RG**|
   |Nombre de la red virtual|**vnet-01**|
   |Region|La misma región de Azure que seleccionó en el primer ejercicio de este laboratorio|

1. En la pestaña **Aspectos básicos** de la página **Crear red virtual**, seleccione **Siguiente**.
1. En la pestaña **Seguridad** de la página **Crear red virtual**, acepte la configuración predeterminada y seleccione **Siguiente**.
1. En la pestaña **Direcciones IP** de la página **Crear red virtual**, asegúrese de que el espacio de direcciones IP esté establecido en **10.0.0.0/16**, elimine la subred **predeterminada**, seleccione **Revisar y crear** y, en la pestaña **Revisar y crear**, seleccione **Crear**.

   > **Nota:** la creación de una red virtual solo debe tardar unos segundos, por lo que debería poder pasar directamente al siguiente paso.

1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Servicios de Kubernetes**.
1. En la página **Servicios de Kubernetes**, seleccione **+ Crear**, en la lista desplegable, elija **Crear clúster de Kubernetes**, posteriormente en la pestaña **Aspectos básico** de la página **Crear clúster de Kubernetes**, especifique la siguiente configuración y, a continuación, seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Subscription|Nombre de la suscripción de Azure seleccionada en el primer ejercicio de este laboratorio|
   |Grupo de recursos|**aks-01-RG**|
   |Configuración preestablecida de clúster|**Desarrollo/pruebas**|
   |Nombre del clúster de Kubernetes|**aks-01**|
   |Region|La misma región de Azure que seleccionó en el primer ejercicio de este laboratorio|
   |Zonas de disponibilidad|**Ninguno**|
   |Plan de tarifa de AKS|**Gratis**|
   |Versión de Kubernetes|Acepte el valor predeterminado|
   |Actualización automática|Deshabilitado|
   |Tipo de canal de seguridad del nodo|**Ninguno**|
   |Autenticación y autorización|**Cuentas locales con RBAC de Kubernetes**|

1. En la pestaña **Grupos de nodo** de la página **Crear clúster de Kubernetes**, realice las tareas siguientes:

   - En la sección **Grupos de nodo**, seleccione el vínculo **agentpool**.
   - En la página **Actualizar grupo de nodo**, en la sección **Tamaño del nodo**, seleccione el vínculo **Elegir el tamaño**.
   - En la página **Seleccionar el tamaño de máquina virtual**, en la lista de tamaños de máquina virtual, seleccione **D2s_v3** y haga clic en **Seleccionar**.
   - De nuevo en la página **Actualizar grupo de nodo**, establezca el **Método de escala** en **Manual** y el **Recuento de nodos** en **2**.
   - En la página **Actualizar grupo de nodo**, seleccione **Actualizar**.

   > **Nota:** es posible que tenga que aumentar las cuotas de vCPU o cambiar la SKU de la máquina virtual para dar cabida a los valores de tamaño de nodo y número de nodos. Para obtener información sobre el procedimiento para aumentar las cuotas de vCPU, consulte el artículo de Microsoft Learn [Aumentar las cuotas de vCPU de una familia de máquinas virtuales](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests).

   > **Nota:** agregará un grupo de nodos de Windows al clúster. Esto requiere cambiar la configuración de red a **Azure CNI** del **kubenet** predeterminado. La configuración de red de kubenet no admite grupos de nodos de Windows.

1. De nuevo en la pestaña **Grupos de nodo** de la página **Crear clúster de Kubernetes**, seleccione **Siguiente**.
1. En la pestaña **Redes** de la página **Crear clúster de Kubernetes**, asegúrese de que la opción **Azure CNI** esté seleccionada y, a continuación, elija la casilla **Traiga su propia red virtual**. Después, en la lista desplegable **Red virtual**, seleccione **vnet-01** y, debajo del cuadro de texto **Subred de clúster**, seleccione **Configuración de subred administrada**.
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
   |Virtual network|**vnet-01**|
   |Subred de clústeres|**aks-subnet (10.0.0.0/20)**|
   |Intervalo de direcciones del servicio de Kubernetes|**172.16.0.0/22**|
   |Dirección IP del servicio DNS de Kubernetes|**172.16.3.254**|
   |Prefijo del nombre DNS|**aks-01-dns**|
   |Directiva de red|**Ninguno**|

1. En la pestaña **Redes** de la página **Crear clúster de Kubernetes**, seleccione **Anterior**.
1. De nuevo en la pestaña **Grupos de nodo** de la página **Crear clúster de Kubernetes**, seleccione **+ Agregar grupo de nodo**.
1. En la página **Agregar grupo de nodos**, especifique la siguiente configuración:

   |Configuración|Valor|
   |---|---|
   |Nombre del grupo de nodos|**w1pool**|
   |Modo|**User**|
   |Tipo de SO|**Windows 2022**|
   |Zona de disponibilidad|**Ninguno**|
   |Habilitar instancias de Azure Spot|Deshabilitado|
   |Tamaño del nodo|**D2s_v3**|
   |Método de escala|**Manual**|
   |Recuento de nodos|**2**|
   |Máximo de pods por nodo|**30**|
   |Habilitar IP pública por nodo|Deshabilitado|

   > **Nota:** Aquí también es posible que deba aumentar las cuotas de vCPU o cambiar la SKU de la máquina virtual para adaptarla al tamaño del nodo y a los valores de recuento de nodos.

1. En la página **Agregar grupo de nodos**, seleccione **Agregar**.
1. De nuevo en la pestaña **Grupos de nodo** de la página **Crear clúster de Kubernetes**, seleccione **Siguiente**.
1. En la pestaña **Redes** de la página **Crear clúster de Kubernetes**, seleccione **Siguiente**.
1. En la pestaña **Integración** de la página **Crear clúster de Kubernetes**, en la lista desplegable **Registro de contenedor**, seleccione la entrada que representa la instancia de Azure Container Registry que creó en el ejercicio anterior, asegúrese de que la opción **Azure Policy** esté deshabilitada y seleccione **Siguiente**.
1. En la pestaña **Supervisión** de la página **Crear clúster de Kubernetes**, desactive la casilla **Habilitar métricas de Prometheus**, desactive la casilla **Habilitar reglas de alerta recomendadas** y, a continuación, seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear** de la página **Crear clúster de Kubernetes**, seleccione **Crear**.

   > **Nota:** continúe con el ejercicio siguiente sin esperar a que se complete el aprovisionamiento del clúster de AKS. El proceso de aprovisionamiento puede tardar unos cinco minutos.
