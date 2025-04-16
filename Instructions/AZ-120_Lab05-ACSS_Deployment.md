---
lab:
  title: 05 - Automatizar la implementación mediante Azure Center for SAP solutions
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Módulo AZ 120: Diseño e implementación de una infraestructura para admitir cargas de trabajo de SAP en Azure
# Laboratorio: Automatizar la implementación mediante Azure Center for SAP solutions

Tiempo estimado: 100 minutos

Todas las tareas de este laboratorio se realizan desde Azure Portal

## Objetivos

Después de completar este laboratorio, podrá:

- Implementación de los requisitos previos para implementar cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions
- Implementación de la infraestructura que hospedará las cargas de trabajo de SAP en Azure usando Azure Center for SAP solutions

## Ejercicio 1: Implementación de los requisitos previos para implementar cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions

Duración: 60 minutos

En este ejercicio, implementará los requisitos previos para implementar cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions. Esto incluirá las siguientes tareas:
- Abordar los requisitos de vCPU en la suscripción de Azure de destino
- Configuración de las asignaciones de roles de control de acceso basado en rol (RBAC) de Azure para la cuenta de usuario de Microsoft Entra ID que se usa para realizar la implementación
- Creación de una cuenta de almacenamiento asociada a la instancia de Azure Center for SAP solutions usada para la implementación
- Creación de una identidad administrada asignada por el usuario para ser usada por Azure Center for SAP solutions para la autenticación y autorización de su implementación automatizada
- Creación de un grupo de seguridad de red (NSG) que se usará en subredes de la red virtual que hospedará la implementación
- Creación de tablas de rutas que se usarán dentro de las subredes de la red virtual que hospedará la implementación
- Creación y configuración de la red virtual que hospedará la implementación
- Implementación de Azure Firewall en la red virtual que hospedará la implementación
- Implementación de Azure Bastion en la red virtual que hospedará la implementación

El ejercicio consta de las tareas siguientes:

- Tarea 1: Aborde los requisitos de vCPU en la suscripción de Azure de destino
- Tarea 2: Configure las asignaciones de roles de control de acceso basado en rol (RBAC) de Azure para la cuenta de usuario de Microsoft Entra ID que se usará para realizar la implementación
- Tarea 3: Cree una cuenta de almacenamiento asociada a la instancia de Azure Center for SAP solutions usada para la implementación
- Tarea 4: Cree y configure una identidad administrada asignada por el usuario para ser usada por Azure Center for SAP solutions para la autenticación y autorización de su implementación automatizada
- Tarea 5: Cree un grupo de seguridad de red (NSG) que se usará en subredes de la red virtual que hospeda la implementación
- Tarea 6: Cree tablas de rutas que se usarán dentro de las subredes de la red virtual que hospeda la implementación
- Tarea 7: Cree y configure la red virtual que hospedará la implementación
- Tarea 8: Implemente Azure Firewall en la red virtual que hospedará la implementación
- Tarea 9: Implemente Azure Bastion en la red virtual que hospedará la implementación

### Tarea 1: Aborde los requisitos de vCPU en la suscripción de Azure de destino

>**Nota**: No es necesario completar esta tarea si ya ha implementado todos los [requisitos previos del laboratorio AZ-120](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab00_Prerequisites.md).

>**Nota**: Para completar este laboratorio (como se describe), necesitará una suscripción de Microsoft Azure con las cuotas de vCPU que admiten la implementación de las siguientes máquinas virtuales:

- 2 máquinas virtuales Standard_E4ds_v4 (4 vCPU y 32 GiB de memoria cada una) o 2 Standard_D4ds_v4 (4 vCPU y 16 GiB de memoria cada una) para el nivel ASCS
- 2 máquinas virtuales Standard_E4ds_v4 (4 vCPU y 32 GiB de memoria cada una) o 2 Standard_D4ds_v4 (4 vCPU y 16 GiB de memoria cada una) para la capa de aplicación 
- 2 máquinas virtuales Standard_M64ms (64 vCPU y 1750 GiB de memoria cada una) para el nivel de base de datos

>**Nota**: Para minimizar los requisitos de memoria y vCPU para las máquinas virtuales de base de datos, puede cambiar la SKU de máquina virtual a Standard_M32ts (32 vCPU y 192 GiB de memoria cada una).

1. En el equipo de laboratorio, inicie un explorador web y vaya a Azure Portal en `https://portal.azure.com`.
1. En Azure Portal, seleccione el icono **Cloud Shell** e inicie una sesión de PowerShell en Cloud Shell. 

    > **Nota**: Si es la primera vez que inicia Cloud Shell en la suscripción de Azure que va a usar en este laboratorio, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En Azure Portal, en el panel de **Cloud Shell**, en la solicitud de PowerShell, ejecute lo siguiente (si es necesario, reemplace `eastus` por el nombre de la región de Azure en la que pretende implementar los recursos de este laboratorio):

    > **Nota**: Para identificar los nombres de las regiones de Azure, en **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute `(Get-AzLocation).Location`
     
    ```powershell
    Set-Variable -Name "Azure_region" -Value ('eastus') -Option constant -Scope global -Description "All processes" -PassThru

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}
    
    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'cores'}
    ```

1. Revise la salida para identificar el uso actual y el límite de vCPU. Asegúrese de que la diferencia entre ellas es suficiente para dar cabida a las vCPU de las máquinas virtuales de Azure que se implementarán en este laboratorio. Tenga en cuenta los números de vCPU regionales y específicos de la familia de máquinas virtuales. 
1. Si el número de vCPU no es suficiente, cierre el panel de Cloud Shell, en Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Cuotas**.
1. En la página **Cuotas**, seleccione **Proceso**.
1. En la página **Cuotas \| Proceso**, use el filtro **Región** para seleccionar la región de Azure a la que va a implementar recursos en este laboratorio.
1. En la columna **Nombre de cuota**, busque y seleccione el nombre de la SKU de máquina virtual que requiere un aumento de cuota. 
1. En la misma fila, compruebe la entrada en la columna **Ajustable**. El siguiente paso depende de si la columna contiene la entrada **Sí** o **No**.

   - Si la entrada está establecida en **Sí**, seleccione el icono **Solicitud de ajuste**, en la **Nueva solicitud de cuota**, en el cuadro de texto **Nuevo límite**, escriba el nuevo límite de cuota y, a continuación, seleccione **Enviar**.
   - Si la entrada está establecida en **No**, seleccione el icono **Solicitar acceso u obtener recomendaciones** y, en el panel **Recomendaciones de cuota**, seleccione la opción **Ponerse en contacto con el soporte técnico** y, a continuación, seleccione **Siguiente**. 
1. En la pestaña **Descripción del problema** de la página **Nueva solicitud de soporte técnico**, especifique la siguiente configuración y, a continuación, seleccione **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |¿Con qué está relacionado su problema?|**Servicios de Azure**|
    |Tipo de problema|**Límites de servicio y suscripción (cuotas)**|
    |Subscription|El nombre de la suscripción de Azure que usa en este laboratorio|
    |Tipo de cuota|**El límite de suscripción de proceso o máquina virtual (núcleos o vCPU) aumenta**|

1. En la pestaña **Detalles adicionales**, seleccione **Especificar detalles**.
1. En la pestaña **Detalles de cuota**, en la lista desplegable **Modelo de implementación**, seleccione **Resource Manager**, en la lista desplegable **Ubicaciones**, seleccione la región de Azure de destino, en la lista desplegable **Cuotas**, seleccione la serie de máquinas virtuales de Azure para la que necesita aumentar los límites de cuota, en el cuadro de texto **Nuevo límite**, escriba el nuevo límite de cuota y, a continuación, seleccione **Guardar y continuar**.
1. De nuevo en la pestaña **Detalles adicionales**, en la pestaña **Información de diagnóstico avanzada**, seleccione **Sí (recomendado)**.
1. En la sección **Método de soporte técnico**, seleccione **Correo electrónico** o **Teléfono** como método de contacto preferido y, a continuación, seleccione **Siguiente**.
1. En la pestaña **Revisar y crear**, seleccione **Crear**.

    > **Nota**: Espere hasta que la solicitud para aumentar los límites de cuota se complete correctamente antes de continuar con la siguiente tarea.

### Tarea 2: Configure las asignaciones de roles de control de acceso basado en rol (RBAC) de Azure para la cuenta de usuario de Microsoft Entra ID que se usará para realizar la implementación

1. En el equipo de laboratorio, inicie Microsoft Edge y vaya a Azure Portal en `https://portal.azure.com`.
1. Cuando se le pida que se autentique, inicie sesión con las credenciales de Microsoft Entra ID con el rol Propietario en la suscripción de Azure que usará para este laboratorio. 
1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Suscripciones**.
1. En la página **Suscripciones**, seleccione la entrada que representa la suscripción de Azure que va a usar para este laboratorio. 
1. En la página que muestra las propiedades de la suscripción de Azure, seleccione **Control de acceso (IAM)**.
1. En la página **Control de acceso (IAM)**, seleccione **+ Agregar** y, después, en el menú desplegable, seleccione **Agregar asignación de roles**.
1. En la pestaña **Rol** de la página **Agregar asignación de roles**, en la lista de **roles de función de trabajo**, busque y seleccione la entrada del **administrador de Azure Center for SAP solutions** y seleccione **Siguiente**.
1. En la pestaña **Miembros** de la página **Agregar asignación de roles**, haga clic en **+ Seleccionar miembros**. 
1. En el panel **Seleccionar miembros**, en el cuadro de texto **Seleccionar**, escriba el nombre de la cuenta de usuario de Microsoft Entra ID que usó para acceder a la suscripción de Azure que usa para este laboratorio, selecciónela en la lista de resultados que coincidan con la entrada y, a continuación, haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Miembros**, seleccione **Revisar y asignar**.
1. En la pestaña **Revisar y asignar**, seleccione **Revisar y asignar**.
1. Repita los seis pasos anteriores para asignar el rol **Operador identidad administrada** a la cuenta de usuario que usa para este laboratorio.

### Tarea 3: Cree una cuenta de almacenamiento asociada a la instancia de Azure Center for SAP solutions usada para la implementación

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Cuentas de almacenamiento**.
1. En la página **Cuentas de almacenamiento**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear una cuenta de almacenamiento**, especifique la siguiente configuración y seleccione **Siguiente: Opciones avanzadas >**.

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|el nombre de un **nuevo** grupo de recursos **ACSS-DEMO**|
    |Nombre de la cuenta de almacenamiento|Cualquier nombre globalmente único con una longitud de 3 a 24 caracteres, que consta de letras y dígitos|
    |Region|el nombre de la región de Azure en la que tiene suficientes cuotas de vCPU para ejecutar este laboratorio|
    |Rendimiento|**Estándar**|
    |Redundancia|**Almacenamiento con redundancia geográfica (GRS)**|
    |Habilite el acceso de lectura a los datos en el caso de que la región esté disponible|Deshabilitado|

1. En la pestaña **Avanzado**, revise las opciones disponibles, acepte los valores predeterminados y seleccione **Siguiente: Redes >**.
1. En la pestaña **Redes**, revise las opciones disponibles, asegúrese de que la opción **Habilitar el acceso público desde todas las redes** está habilitada y seleccione **Revisar**.
1. En la pestaña **Revisar**, espere a que se complete el proceso de validación y seleccione **Crear**.

    >**Nota**: No espere a que se complete el aprovisionamiento de la cuenta de Azure Storage. En su lugar, continúe con la siguiente tarea. El aprovisionamiento puede tardar aproximadamente 2 minutos.

### Tarea 4: Cree y configure una identidad administrada asignada por el usuario para ser usada por Azure Center for SAP solutions para la autenticación y autorización de su implementación automatizada

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Identidades administradas**.
1. En la página **Identidades administradas**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear identidad administrada asignada por el usuario**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|**ACSS-DEMO**|
    |Region|el nombre de la región de Azure en la que aprovisionó la cuenta de almacenamiento anteriormente en este laboratorio|
    |Nombre|**Contoso-MSI**|

1. En la pestaña **Revisar**, espere a que se complete el proceso de validación y seleccione **Crear**.

    >**Nota**: Espere a que se complete el aprovisionamiento de la identidad administrada asignada por el usuario. Esto tardará unos segundos.

1. En Azure Portal, vaya a la página **Identidades administradas** y seleccione la entrada **Contoso-MSI**.
1. En la página **Contoso-MSI**, seleccione **Asignaciones de roles de Azure**.
1. En la página **Asignaciones de roles de Azure**, seleccione **+ Agregar asignación de roles (versión preliminar)**.
1. En el panel **+ Agregar asignación de roles (versión preliminar)**, especifique la siguiente configuración y seleccione **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Ámbito|**Suscripción**|
    |Subscription|El nombre de la suscripción de Azure que usa en este laboratorio|
    |Role|**Rol de servicio de Azure Center for SAP solutions**|

2. De nuevo en la página **Asignaciones de roles de Azure**, seleccione **+ Agregar asignación de roles (versión preliminar)**.
3. En el panel **+ Agregar asignación de roles (versión preliminar)**, especifique la siguiente configuración y seleccione **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Ámbito|**Storage**|
    |Subscription|El nombre de la suscripción de Azure que usa en este laboratorio|
    |Resource|Nombre de la cuenta de Azure Storage que creó en la tarea anterior|
    |Role|**Lector y acceso a los datos**|

### Tarea 5: Cree un grupo de seguridad de red (NSG) que se usará en subredes de la red virtual que hospeda la implementación

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Grupo de seguridad de red**.
1. En la página **Grupos de seguridad de red**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear grupo de seguridad de red**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|Nombre de un **nuevo** grupo de recursos **CONTOSO-VNET-RG**|
    |Nombre|**ACSS-DEMO-NSG**|
    |Region|el nombre de la región de Azure en la que aprovisionó la cuenta de almacenamiento anteriormente en este laboratorio|

1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

    >**Nota**: De manera predeterminada, las reglas integradas de los grupos de seguridad de red permiten todo el tráfico saliente, todo el tráfico dentro de la misma red virtual, así como todo el tráfico entre redes virtuales emparejadas. Esto es suficiente para completar correctamente el laboratorio. En función de los requisitos de seguridad, puede considerar la posibilidad de bloquear parte de ese tráfico. Si es así, consulte las instrucciones incluidas en la documentación de [Preparación de la red para la implementación de la infraestructura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network) de Microsoft Learn.

### Tarea 6: Cree tablas de rutas que se usarán dentro de las subredes de la red virtual que hospeda la implementación

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Tabla de rutas**.
1. En la página **Tabla de rutas**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear tabla de rutas**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|**CONTOSO-VNET-RG**|
    |Region|el nombre de la región de Azure en la que aprovisionó recursos anteriormente en este laboratorio|
    |Nombre|**ACSS-ROUTE**|
    |Propagar las rutas de la puerta de enlace|**No**|

1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

### Tarea 7: Cree y configure la red virtual que hospedará la implementación

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Redes virtuales**. 
1. En la página **Redes virtuales**, seleccione **y Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear red virtual**, especifique la siguiente configuración y seleccione **Siguiente**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nombre de la red virtual|**CONTOSO-VNET**|
    |Region|el nombre de la región de Azure en la que aprovisionó recursos anteriormente en este laboratorio|

1. En la pestaña **Seguridad**, acepte la configuración predeterminada y seleccione **Siguiente**.

    >**Nota**: En este momento, puede aprovisionar tanto Azure Bastion como Azure Firewall, pero los aprovisionará por separado una vez creada la red virtual.

1. En la pestaña **Direcciones IP**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Espacio de direcciones IP|**10.5.0.0/16 (65.536 direcciones)**|

    >**Nota**: Elimine las entradas de subred creadas previamente. Agregará subredes después de crear la red virtual.

1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.
1. Vuelva a la página **Redes virtuales**, seleccione la entrada **CONTOSO-VNET**. 
1. En la página **CONTOSO-VNET**, en la barra de menús vertical del lado izquierdo de la página, seleccione **Subredes**.
1. En la página **CONTOSO-VNET \| Subredes**, seleccione **+ Subred**. 
1. En el panel **Agregar subredes**, especifique la siguiente configuración y seleccione **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|**app**|
    |Intervalo de direcciones de subred|**10.5.0.0/24**|
    |Grupo de seguridad de red|**ACSS-DEMO-NSG**|
    |Tabla de rutas|**ACSS-ROUTE**|

1. De nuevo en la página **CONTOSO-VNET \| Subredes**, seleccione **+ Subred**. 
1. En el panel **Agregar subredes**, especifique la siguiente configuración y seleccione **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|**AzureBastionSubnet**|
    |Intervalo de direcciones de subred|**10.5.1.0/26**|

1. De nuevo en la página **CONTOSO-VNET \| Subredes**, seleccione **+ Subred**. 
1. En el panel **Agregar subredes**, especifique la siguiente configuración y seleccione **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|**db**|
    |Intervalo de direcciones de subred|**10.5.2.0/24**|
    |Grupo de seguridad de red|**ACSS-DEMO-NSG**|
    |Tabla de rutas|**ACSS-ROUTE**|

1. De nuevo en la página **CONTOSO-VNET \| Subredes**, seleccione **+ Subred**. 
1. En el panel **Agregar subredes**, especifique la siguiente configuración y seleccione **Guardar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|**AzureFirewallSubnet**|
    |Intervalo de direcciones de subred|**10.5.3.0/24**|

### Tarea 8: Implemente Azure Firewall en la red virtual que hospedará la implementación

>**Nota**: Antes de implementar una instancia de Azure Firewall, primero creará una directiva de firewall y una dirección IP pública que usará la instancia.

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Directivas de firewall**.
1. En la página **Directivas de firewall**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear una directiva de Azure Firewall**, especifique la siguiente configuración y seleccione **Siguiente: Configuración de DNS >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nombre|**FirewallPolicy_contoso-firewall**|
    |Region|el nombre de la región de Azure en la que aprovisionó recursos anteriormente en este laboratorio|
    |Nivel de directiva|**Estándar**|
    |Directiva principal|**Ninguno**|

1. En la pestaña **Configuración de DNS**, acepte la opción predeterminada **Deshabilitado** y seleccione **Siguiente: Inspección de TLS >**.
1. En la pestaña **Inspección de TLS**, seleccione **Siguiente: Reglas >**.
1. En la pestaña **Reglas**, seleccione **+ Agregar una colección de reglas**.
1. En el panel **Agregar una colección de reglas**, especifique la siguiente configuración:

    |Configuración|Valor|
    |---|---|
    |Nombre|**AllowOutbound**|
    |Tipo de colección de reglas|**Network**|
    |Prioridad|**101**|
    |Acción de colección de reglas|**Permitir**|
    |Grupo de colección de reglas|**DefaultNetworkRuleCollectionGroup**|

1. En el panel **Agregar una colección de reglas**, en la sección **Reglas**, agregue una regla con la siguiente configuración:

    |Configuración|Valor|
    |---|---|
    |Nombre|**RHEL**|
    |Tipo de origen|**Dirección IP**|
    |Origen|*|
    |Protocolo|**Cualquiera**|
    |Puertos de destino|*|
    |Tipo de destino|**Dirección IP**|
    |Destino|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|

    >**Nota**: Para identificar las direcciones IP que se van a usar para RHEL, consulte [Preparación de la red para la implementación de la infraestructura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Configuración|Valor|
    |---|---|
    |Nombre|**ServiceTags**|
    |Tipo de origen|**Dirección IP**|
    |Origen|*|
    |Protocolo|**Cualquiera**|
    |Puertos de destino|*|
    |Tipo de destino|**Etiqueta de servicio**|
    |Destino|**AzureActiveDirectory,AzureKeyVault,Storage**|

    >**Nota**: Si se prefiere, es posible usar etiquetas de servicio con ámbitos regionales. 

    |Configuración|Valor|
    |---|---|
    |Nombre|**SUSE**|
    |Tipo de origen|**Dirección IP**|
    |Origen|*|
    |Protocolo|**Cualquiera**|
    |Puertos de destino|*|
    |Tipo de destino|**Dirección IP**|
    |Destino|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|

    >**Nota**: Para identificar las direcciones IP que se van a usar para SUSE, consulte [Preparación de la red para la implementación de la infraestructura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Configuración|Valor|
    |---|---|
    |Nombre|**AllowOutbound**|
    |Tipo de origen|**Dirección IP**|
    |Origen|*|
    |Protocolo|**TCP,UDP,ICMP,Any**|
    |Puertos de destino|*|
    |Tipo de destino|**Dirección IP**|
    |Destino|*|

1. Seleccione el botón **Agregar** para guardar todas las reglas.
1. De nuevo en la pestaña **Reglas**, seleccione **Siguiente: IDPS >**.
1. En la pestaña **IDPS**, seleccione **Siguiente: Inteligencia sobre amenazas >**.

    >**Nota**: La funcionalidad IDPS requiere la SKU Premium.

1. En la pestaña **Inteligencia sobre amenazas**, revise la configuración disponible sin realizar ningún cambio y, a continuación, seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

    >**Nota**: Espere a que se complete el aprovisionamiento de la directiva de firewall. El aprovisionamiento tardará alrededor de 1 minuto.

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Direcciones IP públicas**.
1. En la página **Direcciones IP públicas**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear una dirección IP pública**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|**CONTOSO-VNET-RG**|
    |Region|el nombre de la región de Azure en la que aprovisionó recursos anteriormente en este laboratorio|
    |Nombre|**contoso-firewal-pip**|
    |Versión de la dirección IP|**IPv4**|
    |SKU|**Estándar**|
    |Zona de disponibilidad|**Sin zona**|
    |Nivel|**Regional**|
    |Preferencia de enrutamiento|**Microsoft Network**|
    |Tiempo de espera de inactividad (minutos)|**4**|
    |Etiqueta de nombre DNS|Sin establecer|

1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

    >**Nota**: Espere a que se complete el aprovisionamiento de la dirección IP pública. El aprovisionamiento debe tardar unos segundos.

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Firewalls**.
1. En la página **Firewalls**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear un firewall**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nombre|**contoso-firewall**|
    |Region|el nombre de la región de Azure en la que aprovisionó recursos anteriormente en este laboratorio|
    |Zona de disponibilidad|**Ninguno**|
    |SKU del firewall|**Estándar**|
    |Administración del firewall|**Uso de una directiva de firewall para administrar este firewall**|
    |Directiva de firewall|**FirewallPolicy_contoso-firewall**|
    |Elegir una red virtual|**Utilizar existente**|
    |Red virtual|**CONTOSO-VNET**|
    |Dirección IP pública|**contoso-firewall-pip**|
    |Tunelización forzada|**Deshabilitado**|

    >**Nota**: Espere a que se complete el aprovisionamiento de Azure Firewall. El aprovisionamiento puede tardar unos 3 minutos.

1. En Azure Portal, vuelva a la página **Firewalls**.
1. En la página **Firewalls**, seleccione la entrada **contoso-firewall**.
1. En la página **contoso-firewall**, tenga en cuenta la entrada **IP privada** establecida en **10.5.3.4** que representa la dirección IP privada de la instancia de Azure Firewall.

    >**Nota**: Para que el tráfico de red se enrute a través de Azure Firewall, debe agregar rutas definidas por el usuario a las tablas de rutas asociadas a las subredes de aplicación y base de datos de la red virtual que hospedarán la implementación de SAP.

1. En Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Tablas de rutas**.
1. En la página **Tablas de rutas**, seleccione la entrada **ACSS-ROUTE**.
1. En la página **ACSS-ROUTE**, seleccione **Rutas**.
1. En la página **ACSS-ROUTE \| Rutas**, seleccione **+ Agregar**.
1. En el panel **Agregar ruta**, especifique la siguiente configuración y seleccione **Agregar**:

    |Configuración|Value|
    |---|---|
    |Nombre de ruta|**Firewall**|
    |Tipo de destino|**Direcciones IP**|
    |Direcciones IP de destino/intervalos CIDR|**0.0.0.0/0**|
    |Tipo de próximo salto|**Aplicación virtual**|
    |Siguiente dirección de salto|**10.5.3.4**|

### Tarea 9: Implemente Azure Bastion en la red virtual que hospedará la implementación

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Bastions**. 
1. En la página **Bastions**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Bastions**, especifique la siguiente configuración y seleccione **Siguiente: Etiquetas >**:

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nombre|**ACSS-BASTION**|
    |Region|el nombre de la región de Azure en la que aprovisionó recursos anteriormente en este laboratorio|
    |Nivel|**Basic**|
    |Recuento de instancias|**2**|
    |Red virtual|**CONTOSO-VNET**|
    |Subnet|**AzureBastionSubnet**|
    |Dirección IP pública|**Crear nuevo**|
    |Nombre de la dirección IP pública|**ACSS-BASTION-PIP**|

1. En la pestaña **Etiquetas**, seleccione **Siguiente: Avanzado >**
1. En la pestaña **Avanzado**, revise la configuración disponible sin realizar ningún cambio y, a continuación, seleccione **Siguiente: Revisar y crear >**
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

    >**Nota**: No espere a que se complete el aprovisionamiento del host de Bastion. En su lugar, continúe con la siguiente tarea. El aprovisionamiento puede tardar unos 15 minutos.

## Ejercicio 2: Implementación de la infraestructura que hospedará las cargas de trabajo de SAP en Azure usando Azure Center for SAP solutions

Duración: 40 minutos

En este ejercicio, usará Azure Center for SAP solutions para implementar la infraestructura que hospedará cargas de trabajo de SAP en la suscripción de Azure que usó en el ejercicio anterior. Después de implementar correctamente el software de SAP, puede proceder a instalarlo usando Azure Center for SAP solutions o eliminando los recursos de Azure aprovisionados en este laboratorio.

>**Nota**: Para obtener información sobre la instalación del software de SAP mediante el uso de Azure Center for SAP solutions, consulte la documentación de Microsoft Learn que describe cómo [Obtener los medios de instalación de SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) e [Instalar el software de SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software). Las instrucciones para eliminar los recursos de Azure aprovisionados en este laboratorio se incluyen en la segunda tarea de este ejercicio.

El ejercicio consta de la siguiente tarea:

- Tarea 1: Creación de Virtual Instance for SAP solutions
- Tarea 2: Eliminación de los recursos de Azure aprovisionados en este laboratorio

### Tarea 1: Creación de Virtual Instance for SAP solutions

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Azure Center for SAP Solutions**. 
1. En la página **Azure Center for SAP Solutions \| Información general**, seleccione **Crear un nuevo sistema SAP**.
1. En la pestaña **Aspectos básicos** de la página **Crear Virtual Instance for SAP solutions**, especifique la siguiente configuración y seleccione **Siguiente: Máquinas virtuales**

    |Configuración|Valor|
    |---|---|
    |Suscripción|Nombre de la suscripción a Azure que usas en este laboratorio|
    |Resource group|Nombre de un **nuevo** grupo de recursos **Contoso-SAP-C1S**|
    |Nombre (SID)|**C1S**|
    |Region|el nombre de la región de Azure en la que aprovisionó recursos anteriormente en este laboratorio|
    |Tipo de entorno|**Producción**
          |
    |Producto de SAP|**S/4HANA**|
    |Base de datos|**HANA**|
    |Método de escalado de HANA|**Escalado vertical (recomendado)**|
    |Tipo de implementación|**Distribuido con alta disponibilidad (HA)**|
    |Disponibilidad de proceso|**99.95 (conjunto de disponibilidad)**|
    |Red virtual|**CONTOSO-VNET**|
    |Subred de aplicación|**aplicación (10.5.0.0/24)**|
    |Subred de base de datos|**base de datos (10.5.2.0/24)**|
    |Imagen del sistema operativo de la aplicación|**Red Hat Enterprise Linux 8.4 para aplicaciones SAP: x64 Gen2 más reciente**|
    |Imagen del sistema operativo de la base de datos|**Red Hat Enterprise Linux 8.4 para aplicaciones SAP: x64 Gen2 más reciente**|
    |Opción de transporte de SAP|**Crear un nuevo directorio de transporte de SAP**|
    |Grupo de recursos de transporte|**ACSS-DEMO**|
    |Nombre de la cuenta de almacenamiento|sin entrada|
    |Tipo de autenticación|**SSH público**|
    |Nombre de usuario|**contososapadmin**|
    |Origen de la clave pública SSH|**Generar par de claves nuevo**|
    |Nombre del par de claves|**contosoc1skey**|
    |SQP FQDN|**sap.contoso.com**|
    |Origen de identidad administrada|**Uso de una identidad administrada asignada por el usuario existente**|
    |Nombre de la identidad administrada|**Contoso-MSI**|

1. En la pestaña **Máquinas virtuales**, especifique la configuración siguiente:

    |Configuración|Valor|
    |---|---|
    |Generación de recomendaciones basadas en|**SAP Application Performance Standard (SAPS): seleccione esta opción para proporcionar SAPS para el tamaño de la memoria de base de datos y el nivel de aplicación y haga clic en Generación de recomendaciones**|
    |SAPS para el nivel de aplicación|**10000**|
    |Tamaño de memoria de la base de datos (GiB)|**1024**|

1. Seleccione **Generar recomendación**.
1. Revise el tamaño y el número de máquinas virtuales para las máquinas virtuales de ASCS, aplicación y base de datos. 

    >**Nota**: Si es necesario, ajuste los tamaños recomendados seleccionando el vínculo **Ver todos los tamaños** de cada conjunto de máquinas virtuales y eligiendo un tamaño alternativo. De forma predeterminada, el tipo de implementación distribuida con alta disponibilidad, así como el tamaño de memoria de base de datos y SAPS del nivel de aplicación especificado anteriormente, da como resultado las siguientes recomendaciones de SKU de máquina virtual:
    - 2 Standard_E4ds_v4 para las máquinas virtuales ASCS (4 vCPU y 32 GiB de memoria cada una)
    - 2 máquinas virtuales Standard_E4ds_v4 para las máquinas virtuales de la aplicación (4 vCPU y 32 GiB de memoria cada una)
    - 2 Standard_M64ms para las máquinas virtuales de base de datos (64 vCPU y 1750 GiB de memoria cada una)

    >**Nota**: Para minimizar los requisitos de memoria y vCPU para las máquinas virtuales de base de datos, considere la posibilidad de cambiar la SKU de máquina virtual a Standard_M32ts (32 vCPU y 192 GiB de memoria cada una).

    >**Nota**: Si es necesario, puede solicitar el aumento de cuota seleccionando el vínculo **Solicitar cuota** para una SKU específica de máquinas virtuales y enviando una solicitud de aumento de cuota. El procesamiento de una solicitud suele tardar unos minutos.

    >**Nota**: Azure Center for SAP solutions aplica el uso de las SKU de máquina virtual admitidas por SAP durante la implementación.

1. En la pestaña **Máquinas virtuales**, en la sección **Discos de datos**, seleccione el vínculo **Ver y personalizar la configuración**.
1. En la página **Configuración del disco de base de datos**, revise la configuración recomendada sin realizar ningún cambio y seleccione **Cerrar**.
1. De nuevo en la pestaña **Máquinas virtuales**, seleccione **Siguiente: Visualizar arquitectura**.
1. En la pestaña **Visualizar arquitectura**, revise el diagrama que ilustra la arquitectura recomendada y seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación, active la casilla de confirmación que tiene una cuota amplia disponible en la región de implementación para evitar que se produzca un error de "Cuota insuficiente" y seleccione **Crear**.
1. Cuando se le solicite, en la ventana **Generar nuevo par de claves**, seleccione **Descargar clave privada y crear recursos**.

    >**Nota**: La clave privada necesaria para conectarse a las máquinas virtuales de Azure incluidas en la implementación se descargará en el equipo desde el que ejecuta este laboratorio.

    >**Nota**: Espere a que la implementación se complete. Esto puede tardar unos 25 minutos.

>**Nota**: Después de la implementación, continúe con la instalación del software de SAP mediante Azure Center for SAP solutions o elimine los recursos de laboratorio siguiendo las instrucciones de la siguiente tarea.

### Tarea 2: Eliminación de los recursos de Azure aprovisionados en este laboratorio

>**Importante**: El costo de los recursos implementados es significativo, por lo que debe asegurarse de desaprovisionar el laboratorio si no pretende usarlo más allá de este punto. La eliminación de la instancia virtual para soluciones de SAP no eliminará los recursos de infraestructura subyacentes. Para eliminar los recursos, debe usar el procedimiento descrito en esta tarea, que tiene como destino los recursos de tres grupos de recursos:

- **Contoso-SAP-C1S**
- **CONTOSO-VNET-RG**
- **ACSS-DEMO**

1. En el equipo del laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, seleccione el icono **Cloud Shell** e inicie una sesión de PowerShell en Cloud Shell. 

    > **Nota**: Si es la primera vez que inicia Cloud Shell en la suscripción de Azure que va a usar en este laboratorio, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En Azure Portal, en el panel de **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute los siguientes comandos para detener y desasignar todas las máquinas virtuales de Azure implementadas en este laboratorio:

    ```powershell
    $resourceGroupName = 'Contoso-SAP-C1S'
    $vms = Get-AzVM -ResourceGroupName $resourceGroupName
    foreach ($vm in $vms) {
       Stop-AzVM -ResourceGroupName $resourceGroupName -Name $vm.Name -Force 
    }
    ```

1. En el símbolo del sistema de PowerShell, ejecute los siguientes comandos para desasociar todos los discos de datos de todas las máquinas virtuales de Azure implementadas en este laboratorio:

    ```powershell
    foreach ($vm in $vms) {  
       $vmDisks = $vm.StorageProfile.DataDisks
       foreach ($vmDisk in $vmDisks) {
          Remove-AzVMDataDisk -VM $vm -Name $vmDisk.Name
       }
       Update-AzVM -ResourceGroupName $resourceGroupName -VM $vm -ErrorAction SilentlyContinue
    }
    ```

1. En el símbolo del sistema de PowerShell, ejecute los siguientes comandos para habilitar la opción de eliminación para las interfaces de red y los discos conectados a todas las máquinas virtuales de Azure implementadas en este laboratorio:

    ```powershell
    foreach ($vm in $vms) {
       $vmConfig = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vm.Name
       $vmConfig.StorageProfile.OsDisk.DeleteOption = 'Delete'
       $vmConfig.StorageProfile.DataDisks | ForEach-Object { $_.DeleteOption = 'Delete' }
       $vmConfig.NetworkProfile.NetworkInterfaces | ForEach-Object { $_.DeleteOption = 'Delete' }
       $vmConfig | Update-AzVM
    }   
    ```

1. En el símbolo del sistema de PowerShell, ejecute los siguientes comandos para eliminar todas las máquinas virtuales de Azure implementadas en este laboratorio:

    ```powershell
    foreach ($vm in $vms) {
       Remove-AzVm -ResourceGroupName $resourceGroupName -Name $vm.Name -ForceDeletion $true -Force
    }
    ```

1. En el símbolo del sistema de PowerShell, ejecute los siguientes comandos para eliminar el grupo de recursos **Contoso-SAP-C1S** y todos sus recursos restantes:

    ```powershell
    Remove-AzResourceGroup -Name 'Contoso-SAP-C1S' -Force -AsJob
    ```

1. En el símbolo del sistema de PowerShell, ejecute los siguientes comandos para eliminar el grupo de recursos **CONTOSO-VNET-RG** y todos sus recursos restantes:

    ```powershell
    Remove-AzResourceGroup -Name 'CONTOSO-VNET-RG' -Force -AsJob
    ```

1. En el símbolo del sistema de PowerShell, ejecute los siguientes comandos para eliminar el grupo de recursos **ACSS-DEMO** y todos sus recursos restantes:

    ```powershell
    Remove-AzResourceGroup -Name 'ACSS-DEMO' -Force -AsJob
    ```

    >**Nota**: Los tres últimos comandos se ejecutan de forma asíncrona (tal y como determina el parámetro -AsJob), por lo que, aunque podrá ejecutar el siguiente comando de PowerShell inmediatamente después dentro de la misma sesión de PowerShell, pasarán unos minutos antes de que se eliminen realmente los grupos de recursos y sus recursos.
