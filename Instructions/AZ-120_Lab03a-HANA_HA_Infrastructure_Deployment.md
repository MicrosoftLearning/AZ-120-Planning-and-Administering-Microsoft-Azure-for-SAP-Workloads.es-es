---
lab:
  title: '04a: Implementación de la arquitectura de SAP en máquinas virtuales de Azure que ejecutan Linux'
  module: Module 04 - Deploy SAP on Azure
---

# AZ 120 Módulo 4: Implementación de SAP en Azure
# Laboratorio 4a: Implementación de la arquitectura de SAP en máquinas virtuales de Azure que ejecutan Linux

Tiempo estimado: 100 minutos

Todas las tareas de este laboratorio se realizan desde Azure Portal (incluida la sesión de Cloud Shell de Bash)  

   > **Nota**: Cuando no se usa Cloud Shell, la máquina virtual de laboratorio debe tener la CLI de Azure instalada [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows).

Archivos de laboratorio: ninguno

## Escenario
  
Como preparación para la implementación de SAP NetWeaver en Azure, Adatum Corporation quiere implementar una demostración que ilustrará la implementación de alta disponibilidad de SAP NetWeaver en máquinas virtuales de Azure que ejecutan la distribución de SUSE de Linux.

## Objetivos
  
Después de completar este laboratorio, podrá:

-   Aprovisionar los recursos de Azure necesarios para admitir una implementación de SAP NetWeaver de alta disponibilidad

-   Configure el sistema operativo de máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP NetWeaver de alta disponibilidad

-   Configure la agrupación en clústeres en máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP NetWeaver de alta disponibilidad

## Requisitos

-   Una suscripción a Microsoft Azure con el número suficiente de vCPU Dsv3 disponibles (cuatro máquinas virtuales Standard_D2s_v3 con 2 vCPU cada una y dos máquinas virtuales Standard_D4s_v3 con 4 vCPU cada una) en una región de Azure que admita zonas de disponibilidad

-   Un equipo de laboratorio con un explorador web compatible con Azure Cloud Shell y acceso a Azure


## Ejercicio 1: Aprovisionamiento de los recursos de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad

Duración: 30 minutos

En este ejercicio, implementará los componentes de proceso de infraestructura de Azure necesarios para configurar la agrupación en clústeres de Linux. Esto implicará la creación de un par de máquinas virtuales de Azure que ejecutan SUSE de Linux en el mismo conjunto de disponibilidad.

### Tarea 1: Cree una red virtual que hospedará una implementación de SAP NetWeaver de alta disponibilidad.

1. En el equipo de laboratorio, inicie un explorador web y vaya a Azure Portal en **https://portal.azure.com**.

1. Si se le solicita, inicie sesión con la cuenta de Microsoft profesional, educativa o personal con el rol de propietario o colaborador de la suscripción a Azure que vaya a usar para este laboratorio y el rol de Administrador global en el inquilino de Azure AD asociado a su suscripción.

1. En Azure Portal, inicie una sesión de Bash en Cloud Shell. 

    > **Nota**: Si es la primera vez que inicia Cloud Shell en la suscripción actual de Azure, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En el panel de Cloud Shell, ejecute el siguiente comando para especificar la región de Azure que es compatible con las zonas de disponibilidad y en la que quiere crear recursos para este laboratorio (sustituya `<region>` por el nombre de la región de Azure que es compatible con las zonas de disponibilidad):

    ```cli
    LOCATION='<region>'
    ```

    > **Nota**: Considere la posibilidad de usar las regiones **Este de EE. UU.** o **Este de EE. UU. 2** para la implementación de los recursos. 

    > **Nota**: Asegúrese de usar la notación adecuada para la región de Azure (nombre corto que no incluye un espacio, por ejemplo, **eastus** en lugar de **Este de EE. UU.**)

    > **Nota**: Para identificar las regiones de Azure que admiten zonas de disponibilidad, consulte [https://docs.microsoft.com/en-us/azure/availability-zones/az-region](https://docs.microsoft.com/en-us/azure/availability-zones/az-region)

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `RESOURCE_GROUP_NAME` en el nombre del grupo de recursos que contiene los recursos que aprovisionó en la tarea anterior:

    ```cli
    RESOURCE_GROUP_NAME='az12003a-sap-RG'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear un grupo de recursos en la región especificada:

    ```cli
    az group create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear una red virtual con una sola subred en el grupo de recursos que creó:

    ```cli
    VNET_NAME='az12003a-sap-vnet'

    VNET_PREFIX='10.3.0.0/16'

    SUBNET_NAME='sapSubnet'

    SUBNET_PREFIX='10.3.0.0/24'

    az network vnet create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION --name $VNET_NAME --address-prefixes $VNET_PREFIX --subnet-name $SUBNET_NAME --subnet-prefixes $SUBNET_PREFIX
    ```

1.  En el panel de Cloud Shell, ejecute el siguiente comando para identificar el identificador de recurso de la subred de la red virtual recién creada y almacenarlo en la variable SUBNET_ID:

    ```cli
    SUBNET_ID=$(az network vnet subnet list --resource-group $RESOURCE_GROUP_NAME --vnet-name $VNET_NAME --query "[?name == '$SUBNET_NAME'].id" --output tsv)
    ```

### Tarea 2: Implementación de una plantilla de Bicep que aprovisiona máquinas virtuales de Azure que ejecutan SUSE de Linux que hospedarán una implementación de SAP NetWeaver de alta disponibilidad

1.  En el equipo de laboratorio, en el panel de Cloud Shell, ejecute los siguientes comandos para crear un clon superficial del repositorio que hospeda la plantilla de Bicep que usará para la implementación de un par de máquinas virtuales de Azure que hospedarán una instalación de alta disponibilidad de SAP HANA y establezca el directorio actual en la ubicación de esa plantilla y su archivo de parámetros:

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/sap/sap-3-tier-marketplace-image-md/
    ```

1.  En el panel de Cloud Shell, ejecute los siguientes comandos para establecer el nombre de la cuenta de usuario administrativa y su contraseña:

    ```
    ADMINUSERNAME='student'
    ADMINPASSWORD='Pa55w.rd1234'
    ```

1.  En el panel de Cloud Shell, ejecute el siguiente comando para identificar los recursos que se incluirán en la próxima implementación:

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group what-if --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters03a.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Revise la salida del comando y compruebe que no incluye ningún error (ignore las advertencias). A continuación, en el panel de Cloud Shell, ejecute el siguiente comando para iniciar la implementación:

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group create --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Revise la salida del comando y compruebe que no incluye errores ni advertencias. Cuando se le solicite, presione la tecla **Entrar** para continuar con la implementación.

1.  No espere a que la implementación finalice, continúe con la siguiente tarea. 

    > **Nota**: Si la implementación falla con el mensaje de error **Conflict** durante la implementación del componente CustomScriptExtension, use los siguientes pasos para solucionar este problema:

       - en Azure Portal, en la hoja **Implementación**, revise los detalles de la implementación e identifique la(s) máquina(s) virtual(es) en la(s) que falló la instalación de CustomScriptExtension

       - en Azure Portal, navegue hasta la hoja de la(s) máquina(s) virtual(es) que identificó en el paso anterior, seleccione **Extensiones** y, desde la hoja **Extensiones**, elimine la extensión CustomScript

       - Vuelva a ejecutar el paso anterior de esta tarea.

### Tarea 3: Implementación de un host de salto

   > **Nota**: Dado que las máquinas virtuales de Azure implementadas en la tarea anterior no son accesibles desde Internet, implementará una máquina virtual de Azure que ejecute Windows Server 2019 Datacenter que servirá como host de salto. 

1. En el equipo de laboratorio, en Azure Portal, haga clic en **+ Crear un recurso**.

1. En la hoja **Nuevo**, inicie la creación de una nueva máquina virtual de Azure basada en la imagen de **Windows Server 2019 Datacenter**.

1. Aprovisione una máquina virtual Azure con la siguiente configuración (deje todas las demás opciones con sus valores predeterminados):

    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure*  |
    | **Grupo de recursos** | *el nombre de un nuevo grupo de recursos* **az12003a-dmz-RG** |
    | **Nombre de la máquina virtual** | **az12003a-vm0** |
    | **Región** | *la misma región de Azure donde implementó máquinas virtuales de Azure en las tareas anteriores de este ejercicio* |
    | **Opciones de disponibilidad** | **No se requiere redundancia de la infraestructura** |
    | **Imagen** | *seleccione* **Windows Server 2019 Datacenter - Gen2** |
    | **Tamaño** | **Standard D2s_v3** o similar |
    | **Nombre de usuario** | **Estudiante** |
    | **Contraseña** | cualquier contraseña compleja de su elección |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **¿Quiere usar una licencia de Windows Server existente?** | **No** |
    | **Tipo de disco del sistema operativo** | **HDD estándar** |
    | **Red virtual** | **az12003a-sap-vnet** |
    | **Nombre de subred** | *una nueva subred denominada* **bastionSubnet** |
    | **Intervalo de direcciones de subred** | **10.3.255.0/24** |
    | **Dirección IP pública** | *una nueva dirección IP denominada***az12003a-vm0-ip** |
    | **Grupo de seguridad de red de NIC** | **Basic**  |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **Habilitación de la redes aceleradas** | **Activado** |
    | **Opciones de equilibrio de carga** | **Ninguno** |
    | **Habilitación de una identidad administrada asignada por el sistema** | **Desactivado** |
    | **Inicio de sesión con Azure AD** | **Desactivado** |
    | **Habilitación del apagado automático** | **Desactivado** |
    | **Opciones de orquestación de revisiones** | **Actualizaciones manuales** |
    | **Diagnósticos de arranque** | **Deshabilitar** |
    | **Habilitación de diagnósticos del SO invitado** | **Desactivado** |
    | **Extensiones** | *Ninguno* |
    | **Etiquetas** | *Ninguno* |

   > **Nota**: Asegúrese de recordar la contraseña que especificó durante la implementación. Lo necesitará más adelante en este laboratorio.

1. Espere a que se complete el aprovisionamiento. Esta operación solo tardará unos minutos.

> **Result**: Después de completar este ejercicio, ha aprovisionado los recursos de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad


## Ejercicio 2: Configuración de máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP NetWeaver de alta disponibilidad

Duración: 30 minutos

En este ejercicio, configurará máquinas virtuales de Azure que ejecutan SUSE Linux Enterprise Server para dar cabida a una implementación de SAP NetWeaver de alta disponibilidad.

### Tarea 1: Configure las redes del nivel de base de datos de máquinas virtuales de Azure.

   > **Nota**: Antes de iniciar esta tarea, asegúrese de que las implementaciones de plantilla iniciadas en el ejercicio anterior se hayan completado correctamente. 

1. En el equipo de laboratorio, en Azure Portal, vaya a la hoja de la máquina virtual de Azure **i20-db-0**.

1. En la hoja **i20-db-0**, vaya a su hoja **Redes**. 

1. Desde la hoja **i20-db-0 - Redes**, navegue hasta la interfaz de red de i20-db-0. 

1. En la hoja de la interfaz de red de i20-db-0, vaya a su hoja configuraciones IP y, desde allí, muestre su hoja **ipconfig1**.

1. En la hoja **ipconfig1**, establezca la dirección IP privada en **10.3.0.20**, cambie su asignación a **Estática** y guarde el cambio.

1. En Azure Portal, navegue hasta la hoja de la máquina virtual de Azure **i20-db-1**.

1. En la hoja **i20-db-1**, vaya a su hoja **Redes**. 

1. Desde la hoja **i20-db-1 - Redes**, navegue hasta la interfaz de red de i20-db-1. 

1. En la hoja de la interfaz de red de i20-db-1, vaya a su hoja configuraciones IP y, desde allí, muestre su hoja **ipconfig1**.

1. En la hoja **ipconfig1**, establezca la dirección IP privada en **10.3.0.21**, cambie su asignación a **Estática** y guarde el cambio.


### Tarea 2: Conéctese a las máquinas virtuales de Azure del nivel de base de datos.

1. En el equipo de laboratorio, en Azure Portal, vaya a la hoja **az12003a-vm0**.

1. En la hoja **az12003a-vm0**, conéctese a la máquina virtual de Azure az12003a-vm0 a través del Escritorio remoto. Cuando se le pida que se autentique, escriba el nombre de usuario y la contraseña que estableció durante la implementación de esta máquina virtual.

1. En la sesión de RDP para az12003a-vm0, en el Administrador del servidor, vaya a la vista **Servidor local** y desactive la **Configuración de seguridad mejorada de IE**.

1. Dentro de la sesión de RDP para az12003a-vm0, descargue e instale PuTTY desde [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

1. Use PuTTY para conectarse mediante SSH a la máquina virtual de Azure **i20-db-0**. Confirme la alerta de seguridad y, cuando se le solicite, proporcione las siguientes credenciales:

    -   Inicio de sesión como: **estudiante**

    -   Contraseña: **Pa55w.rd1234**

1. Use PuTTY para conectarse mediante SSH a la máquina virtual de Azure **i20-db-1** con las mismas credenciales.


### Tarea 3: Examine la configuración de almacenamiento de las máquinas virtuales de Azure del nivel de base de datos.

1. Desde la sesión de SSH de PuTTY a la máquina virtual de Azure i20-db-0, ejecute el siguiente comando para elevar privilegios: 

    ```
    sudo su -
    ```

1. Si se le solicita la contraseña, escriba **Pa55w.rd1234** y presione la tecla **Entrar**. 

1. En la sesión SSH a i20-db-0, compruebe que todos los volúmenes relacionados con SAP HANA (incluidos **/usr/sap**, **/hana/shared**, **/hana/backup**, **/hana/data** y **/hana/logs**) se montan correctamente mediante la ejecución de:

    ```
    df -h
    ```

1. Repita los pasos anteriores en la máquina virtual de Azure i20-db-1.


### Tarea 4: Habilitación del acceso SSH sin contraseña entre nodos

1. En la sesión SSH a i20-db-0, genere la clave SSH sin frase de contraseña mediante la ejecución de:

    ```
    ssh-keygen
    ```

1. Cuando se le solicite, presione **Entrar** tres veces y, a continuación, muestre la clave ejecutando: 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Copie el valor de la clave en el portapapeles.

1. En la sesión SSH a i20-db-1, cree el archivo **/root/.ssh/authorized\_keys** en el editor de vi mediante la ejecución de:

    ```
    vi /root/.ssh/authorized_keys
    ```

1. En el editor de vi, pegue la clave que generó en i20-db-0.

1. Guarde los cambios y cierre el editor.

1. En la sesión SSH a i20-db-1, genere una clave SSH sin frase de contraseña mediante la ejecución de:

    ```
    ssh-keygen
    ```

1. Cuando se le solicite, presione **Entrar** tres veces y, a continuación, muestre la clave ejecutando: 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Copie el valor de la clave en el portapapeles.

1. En la sesión SSH a i20-db-0, cree el archivo **/root/.ssh/authorized\_keys** en el editor de vi mediante la ejecución de:

    ```
    vi /root/.ssh/authorized_keys
    ```

1. En el editor de vi, pegue la clave que generó en i20-db-1 a partir de una nueva línea.

1. Guarde los cambios y cierre el editor.

1. Para comprobar que la configuración se ha realizado correctamente, en la sesión SSH a i20-db-0, establezca una sesión SSH como **raíz** desde i20-db-0 a i20-db-1 ejecutando: 

    ```
    ssh root@i20-db-1
    ```

1. Cuando se le pida si está seguro de continuar con la conexión, escriba `yes` y presione la tecla **Entrar**. 

1. Asegúrese de que no se le solicite la contraseña.

1. Cierre la sesión SSH de i20-db-0 a i20-db-1 mediante la ejecución de: 

    ```
    exit
    ```

1. En la sesión SSH a i20-db-1, establezca una sesión SSH como **raíz** de i20-db-1 a i20-db-0 mediante la ejecución de: 

    ```
    ssh root@i20-db-0
    ```

1. Cuando se le pida si está seguro de continuar con la conexión, escriba `yes` y presione la tecla **Entrar**. 

1. Asegúrese de que no se le solicite la contraseña.

1. Cierre la sesión SSH de i20-db-1 a i20-db-0 mediante la ejecución de: 

    ```
    exit
    ```

### Tarea 5: Agregar paquetes YaST, actualizar el sistema operativo Linux e instalar extensiones de alta disponibilidad

1. En la sesión SSH a i20-db-0, ejecute lo siguiente para iniciar YaST:

    ```
    yast
    ```

1. En el **Centro de control de **, seleccione **Software -\> Productos complementarios** y pulse **Entrar**. Esto cargará el **Administrador de paquetes**.

1. En la pantalla **Productos complementarios instalados**, compruebe que **Módulo de nube pública** ya está instalado. A continuación, presione **F9** dos veces para volver a la solicitud del shell.

1. En la sesión SSH a i20-db-0, ejecute lo siguiente para actualizar el sistema operativo (cuando se le solicite, escriba **y** y presione la tecla **Entrar**):

    ```
    zypper update
    ```

1. En la sesión SSH a i20-db-0, ejecute lo siguiente para instalar los paquetes necesarios para los recursos del clúster (cuando se le solicite, escriba **y** y presione la tecla **Entrar**):

    ```
    zypper in socat
    ```

1. En la sesión SSH a i20-db-0, ejecute lo siguiente para instalar el componente azure-lb requerido por los recursos del clúster:

    ```
    zypper in resource-agents
    ```

1. En la sesión SSH a i20-db-0, abra el archivo **/etc/systemd/system.conf** en el editor de vi ejecutando:

    ```
    vi /etc/systemd/system.conf
    ```

1. En el editor de vi, reemplace `#DefaultTasksMax=512` por `DefaultTasksMax=4096`. 

    > **Nota**: En algunos casos, Pacemaker puede crear muchos procesos, alcanzando el límite predeterminado impuesto en su número y desencadenando una conmutación por error. Este cambio aumenta el número máximo de procesos permitidos.

1. Guarde los cambios y cierre el editor.

1. En la sesión SSH a i20-db-0, ejecute lo siguiente para activar el cambio de configuración:

    ```
    systemctl daemon-reload
    ```

1. En la sesión SSH a i20-db-0, ejecute lo siguiente para instalar el paquete de agentes de barrera:

    ```
    zypper install fence-agents
    ```

1. En la sesión SSH a i20-db-0, ejecute lo siguiente para instalar el SDK de Python de Azure requerido por el agente de barrera (cuando se le solicite, escriba **y** y presione la tecla **Entrar**):

    ```
    SUSEConnect -p sle-module-public-cloud/12/x86_64
    zypper install python-azure-mgmt-compute
    ```

1. Repita los pasos anteriores de esta tarea en i20-db-1.

> **Result**: Después de completar este ejercicio, ha configurado el sistema operativo de máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP NetWeaver de alta disponibilidad

## Ejercicio 3: Configuración de la agrupación en clústeres en máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP NetWeaver de alta disponibilidad

Duración: 30 minutos

En este ejercicio, configurará la agrupación en clústeres en máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP NetWeaver de alta disponibilidad.

### Tarea 1: Configuración de la agrupación en clústeres

1. En la sesión RDP a az12003a-vm0, en la sesión SSH basada en PuTTY en i20-db-0, ejecute lo siguiente para iniciar la configuración de un clúster de alta disponibilidad en i20-db-0:

    ```
    ha-cluster-init -u
    ```

1. Cuando se le solicite, proporcione las siguientes respuestas:

    -   ¿Desea continuar de todos modos (y/n)?: **y**

    -   Dirección de ring0 [10.3.0.20]: **ENTRAR**

    -   Puerto para ring0 [5405]: **ENTRAR**

    -   ¿Desea usar SBD (y/n)?: **n**

    -   ¿Desea configurar una dirección IP virtual (y/n)?: **n**

    > **Nota**: La configuración de la agrupación en clústeres genera una cuenta de **hacluster** con su contraseña establecida en **linux**. La cambiará más adelante en esta tarea.

1. En la sesión rdP a az12003a-vm0, en la sesión SSH basada en PuTTY en i20-db-1, ejecute lo siguiente para unir el clúster de alta disponibilidad en i20-db-0 desde i20-db-1:

    ```
    ha-cluster-join
    ```

1. Cuando se le solicite, proporcione las siguientes respuestas:

    -   ¿Desea continuar de todos modos (y/n)? **y**

    -   Dirección IP o nombre de host del nodo existente (por ejemplo: 192.168.1.1) \[\]: **i20-db-0**

    -   Dirección de ring0 [10.3.0.21]: **ENTRAR**

1. En la sesión SSH basada en PuTTY en i20-db-0, ejecute lo siguiente para establecer la contraseña de la cuenta de **hacluster** en **Pa55w.rd1234** (escriba la nueva contraseña cuando se le solicite): 

    ```
    passwd hacluster
    ```

1. Repita el paso anterior en i20-db-1.

### Tarea 2: Revisión de la configuración de corosync

1. En la sesión RDP a az12003a-vm0, en la sesión SSH basada en PuTTY en i20-db-0, abra el archivo **/etc/corosync/corosync/corosync.conf** ejecutando:

    ```
    vi /etc/corosync/corosync.conf
    ```

1. En el editor de vi, observe la entrada `transport: udpu` y la sección `nodelist`:
    ```
    [...]
       interface { 
           [...] 
       }
       transport:      udpu
    } 
    nodelist {
       node {
         ring0_addr:     10.3.0.20
         nodeid:     1
       }
       node {
         ring0_addr:     10.3.0.21
         nodeid:     2
       } 
    }
    logging {
        [...]
    ```

1. En el editor de vi, reemplace la entrada `token: 5000` por `token: 30000`.

    > **Nota**: Este cambio permite mantener la memoria conservando el mantenimiento. Para más información, consulte la [Documentación de Microsoft sobre el mantenimiento de máquinas virtuales en Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-and-updates#maintenance-that-doesnt-require-a-reboot)

1. Guarde los cambios y cierre el editor.

1. Repita los pasos anteriores en i20-db-1.


### Tarea 3: Identificar el valor del identificador de suscripción de Azure y el identificador de inquilino de Azure AD

1. Desde el equipo del laboratorio, en la ventana del navegador, en Azure Portal en **https://portal.azure.com**, asegúrese de que ha iniciado sesión con la cuenta de usuario que tiene el rol de Administrador global en el inquilino de Azure AD asociado a su suscripción.

1. En Azure Portal, inicie una sesión de Bash en Cloud Shell. 

1. En el panel de Cloud Shell, ejecute el siguiente comando para identificar el id. de su suscripción de Azure y el id. del inquilino de Azure AD correspondiente:

    ```cli
    az account show --query '{id:id, tenantId:tenantId}' --output json
    ```

1. Copie los valores resultantes en el Bloc de notas. Lo necesitará en la próxima tarea.


### Tarea 4: Creación de una aplicación de Azure AD para el dispositivo STONITH

1. Acceda a Azure Portal y vaya a la hoja Azure Active Directory.

1. Desde la hoja de Azure Active Directory, vaya a la hoja **Registros de aplicaciones** y después haga clic en **+ Nuevo registro**:

1. En la hoja **Registrar una aplicación**, especifique la siguiente configuración y haga clic en **Registrar**:

    -   Nombre: **Aplicación Stonith**

    -   Tipo de cuenta admitido: **Solo las cuentas de este directorio organizativo**

1. En la hoja de **Aplicación Stonith**, copie el valor de **Id. de aplicación (cliente)** en el Bloc de notas. Esto se denominará **login_id** más adelante en este ejercicio:

1. En la hoja de **Aplicación Stonith**, haga clic en **Certificados y secretos**.

1. En la hoja **Aplicación Stonith - Certificados y secretos**, haga clic en **+ Nuevo secreto de cliente**.

1. En el panel **Agregar un secreto de cliente**, en el cuadro de texto **Descripción**, escriba **STONITH app key**, en la sección **Expira**, deje el valor predeterminado **Recomendado: 6 meses** y, a continuación, haga clic en **Agregar**.

1. Copie el **Valor** resultante en el Bloc de notas (esta entrada solo aparece una vez, después de hacer clic en **Agregar**). Esto se denominará **contraseña** más adelante en este ejercicio:


### Tarea 5: Concesión de permisos a máquinas virtuales de Azure a la entidad de servicio de la aplicación STONITH 

1. En Azure Portal, vaya a la hoja de la máquina virtual de Azure **i20-db-0**

1. Desde la hoja **i20-db-0**, visualice la hoja **i20-db-0 - Control de acceso (IAM)**.

1. Desde la hoja **i20-db-0 - Control de acceso (IAM)**, agregue una asignación de rol con la siguiente configuración:

    -   Rol: **Colaborador de la máquina virtual**

    -   Asignar acceso a: **Usuario, grupo o entidad de servicio de Azure AD**

    -   Seleccione: **Aplicación Stonith**

1. Repita los pasos anteriores para asignar a la aplicación Stonith el rol de Colaborador de máquinas virtuales a la máquina virtual de Azure **i20-db-1**


### Tarea 6: Configuración del dispositivo de clúster STONITH 

1. Dentro de la sesión RDP a az12003a-vm0, cambie a la sesión SSH basada en PuTTY a i20-db-0.

1. Dentro de la sesión RDP a az12003a-vm0, en la sesión SSH basada en PuTTY a i20-db-0, ejecute los siguientes comandos (asegúrese de reemplazar los marcadores de posición `subscription_id`, `tenant_id`, `login_id,` y `password` por los valores que identificó en la tarea 4 del Ejercicio 3:

    ```
    crm configure property stonith-enabled=true

    crm configure property concurrent-fencing=true

    crm configure primitive rsc_st_azure stonith:fence_azure_arm \
      params subscriptionId="subscription_id" resourceGroup="az12003a-sap-RG" tenantId="tenant_id" login="login_id" passwd="password" \
      pcmk_monitor_retries=4 pcmk_action_limit=3 power_timeout=240 pcmk_reboot_timeout=900 \
      op monitor interval=3600 timeout=120

    sudo crm configure property stonith-timeout=900
    ```

### Tarea 7: Revisión de la configuración de agrupación en clústeres en máquinas virtuales de Azure que ejecutan Linux mediante Hawk

1. Dentro de la sesión RDP a az12003a-vm0, inicie Internet Explorer y vaya a **https://i20-db-0:7630**. Esto debería mostrar la página de inicio de sesión de SUSE Hawk.

   > **Nota**: Ignore el mensaje **Este sitio no es seguro**.

1. En la página de inicio de sesión de SUSE Hawk, inicie sesión con las siguientes credenciales:

    -   Nombre de usuario: **hacluster**

    -   Contraseña: **Pa55w.rd1234**

1. Compruebe que el estado del clúster es correcto. Si ve un mensaje que indica que uno de los dos nodos de clúster no está limpio, reinicie ese nodo desde Azure Portal.

> **Result**: Después de completar este ejercicio, ha configurado la agrupación en clústeres en máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP NetWeaver de alta disponibilidad


## Ejercicio 4: Eliminación de recursos de laboratorio

Duración: 10 minutos

En este ejercicio, quitará los recursos aprovisionados en este laboratorio.

#### Tarea 1: Apertura de Cloud Shell

1. En la parte superior del portal, haga clic en el icono de **Cloud Shell** para abrir el panel de Cloud Shell y elija Bash como shell.

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `RESOURCE_GROUP_PREFIX` en el prefijo del nombre del grupo de recursos que contiene los recursos que aprovisionó en este laboratorio:

    ```cli
    RESOURCE_GROUP_PREFIX='az12003a-'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para enumerar todos los grupos de recursos que creó en este laboratorio:

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv
    ```

1. Compruebe que la salida contiene solo el grupo de recursos que creó en este laboratorio. Este grupo de recursos con todos sus recursos se eliminará en la siguiente tarea.

#### Tarea 2: Eliminación de los grupos de recursos

1. En el panel de Cloud Shell, ejecute el siguiente comando para eliminar el grupo de recursos y sus recursos.

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. Cierre el panel de Cloud Shell.

> **Result**: después de completar este ejercicio, ha quitado los recursos usados en este laboratorio.
