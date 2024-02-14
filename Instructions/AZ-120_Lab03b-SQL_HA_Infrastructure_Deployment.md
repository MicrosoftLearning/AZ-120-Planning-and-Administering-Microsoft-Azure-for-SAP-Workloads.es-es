---
lab:
  title: '04b: Implementación de la arquitectura de SAP en máquinas virtuales de Azure que ejecutan Windows'
  module: Module 04 - Deploy SAP on Azure
---

# AZ 120 Módulo 4: Implementación de SAP en Azure
# Laboratorio 4b: Implementa la arquitectura de SAP en máquinas virtuales de Azure que ejecutan Windows

Tiempo estimado: 150 minutos

Todas las tareas de este laboratorio se realizan en Azure Portal (incluida una sesión de Cloud Shell de PowerShell)  

   > **Nota**: Cuando no se usa Cloud Shell, la máquina virtual de laboratorio debe tener el módulo Az de PowerShell instalado [**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Archivos de laboratorio: ninguno

## Escenario
  
Como preparación para la implementación de SAP NetWeaver en Azure, Adatum Corporation quiere implementar una demostración que ilustrará la implementación de alta disponibilidad de SAP NetWeaver en máquinas virtuales de Azure que ejecutan Windows Server 2016.

## Objetivos
  
Después de completar este laboratorio, podrá:

-   Aprovisionar los recursos de Azure necesarios para admitir una implementación de SAP NetWeaver de alta disponibilidad

-   Configurar el sistema operativo de máquinas virtuales de Azure que ejecutan Windows para admitir una implementación de SAP NetWeaver de alta disponibilidad

-   Configurar la agrupación en clústeres en máquinas virtuales de Azure que ejecutan Windows para admitir una implementación de SAP NetWeaver de alta disponibilidad

## Requisitos

-   Una suscripción a Microsoft Azure con el número suficiente de vCPU Dsv3 disponibles (cuatro máquinas virtuales Standard_D2s_v3 con 2 vCPU y seis máquinas virtuales Standard_D4s_v3 con 4 vCPU cada una) en una región de Azure que admita zonas de disponibilidad

-   Un equipo de laboratorio con un explorador web compatible con Azure Cloud Shell y acceso a Azure


## Ejercicio 1: Aprovisionamiento de los recursos de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad

Duración: 60 minutos

En este ejercicio, implementará los componentes de proceso de infraestructura de Azure necesarios para configurar la agrupación en clústeres de Windows. Esto implicará la creación de un par de máquinas virtuales de Azure que ejecutan Windows Server 2016 en el mismo conjunto de disponibilidad.

### Tarea 1: implementación de un par de máquinas virtuales de Azure que ejecutan controladores de dominio de Active Directory de alta disponibilidad mediante una plantilla de Azure Resource Manager

1. En el equipo de laboratorio, inicie un explorador web y vaya a Azure Portal en **https://portal.azure.com**.

1. Si se le solicita, inicie sesión con la cuenta profesional o educativa o personal de Microsoft con el rol de propietario o colaborador en la suscripción de Azure que usará para este laboratorio.

1. En Azure Portal, inicie una sesión de PowerShell en Cloud Shell. 

    > **Nota**: si es la primera vez que inicia Cloud Shell en la suscripción actual de Azure, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En el panel de Cloud Shell, ejecute los siguientes comandos para crear un clon superficial del repositorio que hospeda la plantilla de Bicep que usará para la implementación de un par de máquinas virtuales de Azure que ejecutan controladores de dominio de Active Directory de alta disponibilidad y establezca el directorio actual en la ubicación de esa plantilla y su archivo de parámetros:

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$rgName` en `az12001b-ad-RG`:

    ```
    $rgName = 'az12003b-ad-RG'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$location` en el nombre de las regiones de Azure que admite zonas de disponibilidad y dónde pretende implementar las máquinas virtuales del laboratorio (reemplace el marcador de posición `<Azure_region>` por el nombre de esa región):

    ```
    $location = '<Azure_region>'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear un grupo de recursos denominado **az12001b-ad-RG** en la región de Azure que eligió:

    ```
    New-AzResourceGroup -Name $rgName -Location $location
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$deploymentName`:

    ```
    $deploymentName = 'az1203b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
    ```

1. En el panel de Cloud Shell, ejecute los siguientes comandos para establecer el nombre de la cuenta de usuario administrativa y su contraseña (reemplace los marcadores de posición `<username>` y `<password>` por el nombre de la cuenta de usuario administrativa y el valor de su contraseña, respectivamente):

    ```
    $adminUsername = '<username>'
    $adminPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
    ```

    > **Nota**: asegúrese de que la contraseña cumple los requisitos de complejidad aplicables a la implementación de máquinas virtuales de Azure que ejecutan Windows (la longitud de al menos 12 caracteres que contiene letras minúsculas y mayúsculas, dígitos y caracteres especiales).

1. En el panel de Cloud Shell, ejecute el siguiente comando para ejecutar la implementación:

    ```
    New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $rgName -TemplateFile .\main.bicep -TemplateParameterFile .\azuredeploy.parameters.json -adminUsername $adminUsername -adminPassword $adminPassword -c
    ```

1. Revise la salida del comando y compruebe que no incluye errores ni advertencias. Cuando se le solicite, presione la tecla **Entrar** para continuar con la implementación.

    > **Nota**: La implementación tardará unos 30 minutos. Espere a que la implementación se complete antes de avanzar a la siguiente tarea.

    > **Nota**: si se produce un error en la implementación, incluida la instrucción `PowerShell DSC resource MSFT_xADDomainController failed to execute Set-TargetResource functionality with error message: Domain 'adatum.com' could not be found`, siga estos pasos para corregir este problema:

    - En Azure Portal, vaya a la hoja de la **máquina virtual adBDC**, en el menú de navegación vertical del lado izquierdo, en la sección **Configuración**, seleccione **Extensiones y aplicaciones**, en el panel **Extensiones y aplicaciones**, seleccione **Preparar BDC** y, en el panel **Preparar BDC**, seleccione **Desinstalar**. 

    - Vuelva a la hoja de máquina virtual **adBDC** y reinicie la máquina virtual de Azure.

    - Vaya a la hoja **az1203b-ad-RG**, en el menú de navegación vertical del lado izquierdo, en la **sección Configuración**, seleccione **Implementaciones**.

    - En la hoja **az1203b-ad-RG \| Implementaciones**, seleccione la implementación que comienza por el prefijo **az1203b** y, en la hoja de implementación, seleccione **Volver a implementar**.

    - En la hoja **Implementación personalizada**, en el cuadro de texto **Administración de contraseña**, escriba la misma contraseña que usó durante la implementación original, seleccione **Revisar y crear** y, a continuación, seleccione **Crear**.

    - No espere a que se complete la reimplementación, sino que continúe con la siguiente tarea. La reimplementación debe tardar unos 3 minutos.

### Tarea 2: Aprovisione subredes que hospedarán máquinas virtuales de Azure que ejecutan la implementación de SAP NetWeaver de alta disponibilidad y el clúster S2D.

1. En Azure Portal, vaya a la hoja del grupo de recursos **az12003b-ad-RG**.

1. En la hoja del grupo de recursos **az12003b-ad-RG**, busque la red virtual **adVNET** y haga clic en su entrada para mostrar la hoja **adVNET**.

1. En la hoja **adVNET**, vaya a la hoja **adVNET: Subredes**. 

1. En la hoja **adVNET: Subredes**, cree una subred con la siguiente configuración:

    -   Nombre: **sapSubnet**

    -   Intervalos de direcciones (bloque CIDR): **10.0.1.0/24**

1. En la hoja **adVNET: Subredes**, cree una subred con la siguiente configuración:

    -   Nombre: **s2dSubnet**

    -   Intervalos de direcciones (bloque CIDR): **10.0.2.0/24**

1. En Azure Portal, inicie una sesión de PowerShell en Cloud Shell. 

    > **Nota**: si es la primera vez que inicia Cloud Shell en la suscripción actual de Azure, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene los recursos que aprovisionó en la tarea anterior:

    ```
    $resourceGroupName = 'az12003b-ad-RG'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para identificar la red virtual creada en la tarea anterior:

    ```
    $vNetName = 'adVNet'

    $subnetName = 'sapSubnet'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para identificar el id. de recurso de la subred recién creada:

    ```
    $vNet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vNetName
    
    (Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vNet).Id
    ```

1. Copie el valor resultante en el Portapapeles. Lo necesitará en la próxima tarea.

### Tarea 3: Implementar la plantilla de Azure Resource Manager que aprovisiona las máquinas virtuales de Azure que ejecutan Windows Server 2016 y que hospedarán una implementación de SAP NetWeaver de alta disponibilidad

1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Template Deployment (implementar mediante plantilla personalizada).**

1. En la hoja **Implementación personalizada**, en la lista desplegable **Plantilla de inicio rápido (declinación de responsabilidades)**, escriba **application-workloads/sap/sap-3-tier-marketplace-image-md** y haga clic en **Seleccionar plantilla**.

    > **Nota**: Asegúrese de usar Microsoft Edge o un explorador de terceros. No use Internet Explorer.

1. En la hoja **SAP NetWeaver de 3 niveles (disco administrado)**, seleccione **Editar plantilla**.

1. En la hoja **Editar plantilla**, aplique el siguiente cambio y seleccione **Guardar**:

    -   en la línea **197**, reemplace `"dbVMSize": "Standard_E8s_v3",` por `"dbVMSize": "Standard_D4s_v3",`

1. De nuevo en la hoja **SAP NetWeaver de 3 niveles (disco administrado),** especifique la siguiente configuración, haga clic en **Revisar y crear** y, a continuación, haga clic en **Crear** para iniciar la implementación:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure*  |
    | **Grupo de recursos** | *el nombre de un nuevo grupo de recursos* **az12003b-sap-RG** |
    | **Ubicación** | *la misma región de Azure que especificó en la primera tarea de este ejercicio* |
    | **Identificador de sistema SAP** | **I20** |
    | **Tipo de pila** | **ABAP** |
    | **Tipo de SO** | **Windows Server 2016 Datacenter** |
    | **Dbtype** | **SQL** |
    | **Tamaño del sistema SAP** | **Demostración** |
    | **Disponibilidad del sistema** | **ALTA DISPONIBILIDAD** |
    | **Nombre de usuario administrador** | **Estudiante** |
    | **Tipo de autenticación** | **password** |
    | **Contraseña o clave de administrador** | *la misma contraseña que especificó anteriormente en este laboratorio* |
    | **Id. de subred** | *el valor que copió en el Portapapeles en la tarea anterior* |
    | **Zonas de disponibilidad** | **1, 2** |
    | **Ubicación** | **[resourceGroup().location]** |
    | **_artifacts Location** (Ubicación de _artefactos) | **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/sap/sap-3-tier-marketplace-image-md/** |
    | **_token de SaS de ubicación de artefactos** | *dejar en blanco* |

1. No espere a que se completen las implementaciones, sino que avance a la siguiente tarea. 

### Tarea 4: Implementar el clúster del Servidor de archivos de escalabilidad horizontal (SOFS)

En esta tarea, implementará el clúster del Servidor de archivos de escalabilidad horizontal (SOFS) que hospedará un recurso compartido de archivos para los servidores ASCS de SAP mediante una plantilla de inicio rápido de Azure Resource Manager de GitHub disponible en [](https://github.com/polichtm/301-storage-spaces-direct-md)**https://github.com/polichtm/301-storage-spaces-direct-md**. 

1. En el equipo de laboratorio, inicie un explorador y vaya a [**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md). 

    > **Nota**: Asegúrese de usar Microsoft Edge o un explorador de terceros. No use Internet Explorer.

1. En la página denominada **Usar discos administrados para crear un clúster de Servidor de archivos de escalabilidad horizontal (SOFS) de Espacios de almacenamiento directo (S2D) con Windows Server 2016**, haga clic en **Implementar en Azure**. Esto redirigirá automáticamente el explorador a Azure Portal y mostrará la hoja **Implementación personalizada**.

1. En la hoja **Implementación personalizada**, especifique la siguiente configuración, haga clic en **Revisar y crear** y, a continuación, haga clic en **Crear** para iniciar la implementación:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure*  |
    | **Grupo de recursos** | *el nombre de un nuevo grupo de recursos* **az12003b-s2d-RG** |
    | **Región** | *la misma región de Azure donde implementó máquinas virtuales de Azure en las tareas anteriores de este ejercicio* |
    | **Prefijo de nombre** | **i20** |
    | **Tamaño de máquina virtual** | **Standard D4s\_v3** |
    | **Habilitación de la redes aceleradas** | **true** |
    | **SKU de la imagen** | **2016-Datacenter-Server-Core** |
    | **Recuento de VM** | **2** |
    | **Tamaño de disco de máquina virtual** | **128** |
    | **Número de disco de máquina virtual** | **3** |
    | **Nombre de dominio existente** | **adatum.com** |
    | **Nombre de usuario administrador** | **Estudiante** |
    | **Contraseña del administrador** | *la misma contraseña que especificó anteriormente en este laboratorio* |
    | **Nombre de RG de la red virtual existente** | **az12003b-ad-RG** |
    | **Nombre de la red virtual existente** | **adVNet** |
    | **Nombre de subred existente** | **s2dSubnet** |
    | **Nombre de Sofs** | **sapglobalhost** |
    | **Nombre del recurso compartido** | **sapmnt** |
    | **Día de actualización programado** | **Domingo** |
    | **Hora de la actualización programada** | **3:00 a. m.** |
    | **Antimalware en tiempo real habilitado** | **false** |
    | **Antimalware programado habilitado** | **false** |
    | **Hora del antimalware programada** | **120** |
    | **_artifacts Location** (Ubicación de _artefactos) | **https://raw.githubusercontent.com/polichtm/301-storage-spaces-direct-md/master** |
    | **_token de SaS de ubicación de artefactos** | **Deje el valor predeterminado** |

1. La implementación puede tardar unos 20 minutos. No espere a que se completen las implementaciones, sino que avance a la siguiente tarea.

    > **Nota**: Si se produce un error en la implementación con el mensaje de error **Conflicto** durante la implementación del componente i20-s2d-1/s2dPrep o i20-s2d-0/s2dPrep, siga estos pasos para corregir este problema:

       - En Azure Portal, vaya a la máquina virtual **i20-s2d-0**, en el menú de navegación vertical, en la sección **Operaciones**, seleccione **Ejecutar comando**, en el panel **Ejecutar script de comando**, en el cuadro de texto **Script PowerShell**, escriba el siguiente script y seleccione el botón **Ejecutar** (asegúrese de reemplazar el marcador de posición `<password>` por la contraseña especificada anteriormente en este laboratorio):

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```

       - Vaya a la hoja de la máquina virtual **i20-s2d-1**, en el menú de navegación vertical, en la sección **Operaciones**, seleccione **Ejecutar comando**, en el panel **Ejecutar script de comando**, en el cuadro de texto **Script PowerShell**, escriba el siguiente script y seleccione el botón **Ejecutar** (asegúrese de reemplazar el marcador de posición `<password>` por la contraseña especificada anteriormente en este laboratorio):

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```
       
       - Vuelva a ejecutar los pasos de la tarea actual desde el principio

### Tarea 5: Implementación de un host de salto

   > **Nota**: Dado que las máquinas virtuales de Azure implementadas en la tarea anterior no son accesibles desde Internet, implementará una máquina virtual de Azure que ejecute Windows Server 2016 Datacenter que servirá como host de salto. 

1. En el equipo de laboratorio, en la interfaz de Azure Portal, haga clic en **+ Crear un recurso**.

1. En la hoja **Nuevo**, inicie la creación de una nueva máquina virtual de Azure basada en la imagen de **Windows Server 2019 Datacenter: Gen1**.

1. Aprovisione una máquina virtual de Azure con la siguiente configuración:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure*  |
    | **Grupo de recursos** | *el nombre de un nuevo grupo de recursos* **az12003b-dmz-RG** |
    | **Nombre de la máquina virtual** | **az12003b-vm0** |
    | **Región** | *la misma región de Azure donde implementó máquinas virtuales de Azure en las tareas anteriores de este ejercicio* |
    | **Opciones de disponibilidad** | **No se requiere redundancia de la infraestructura** |
    | **Imagen** | *seleccione* **Windows Server 2019 Datacenter - Gen2** |
    | **Tamaño** | **Estándar D2s_v3** |
    | **Nombre de usuario** | **Estudiante** |
    | **Contraseña** | *la misma contraseña que especificó anteriormente en este laboratorio* |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **¿Quiere usar una licencia de Windows Server existente?** | **No** |
    | **Tipo de disco del sistema operativo** | **HDD estándar** |
    | **Red virtual** | **adVNET** |
    | **Nombre de subred** | *una nueva subred denominada* **dmzSubnet** |
    | **Intervalo de direcciones de subred** | **10.0.255.0/24** |
    | **Dirección IP pública** | *una nueva dirección IP denominada* **az12003b-vm0-ip** |
    | **Grupo de seguridad de red de NIC** | **Basic**  |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **Habilitación de la redes aceleradas** | **Desactivado** |
    | **Opciones de equilibrio de carga** | **Ninguno** |
    | **Habilitación de una identidad administrada asignada por el sistema** | **Desactivado** |
    | **Inicio de sesión con Azure AD** | **Desactivado** |
    | **Habilitación del apagado automático** | **Desactivado** |
    | **Opciones de orquestación de revisiones** | **Actualizaciones manuales** |
    | **Diagnósticos de arranque** | **Deshabilitar** |
    | **Habilitación de diagnósticos del SO invitado** | **Desactivado** |
    | **Extensiones** | *Ninguno* |
    | **Etiquetas** | *Ninguno* |
   
1. Espere a que se complete el aprovisionamiento. Esta operación solo tardará unos minutos.

> **Result**: Después de completar este ejercicio, ha aprovisionado los recursos de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad


## Ejercicio 2: Configuración del sistema operativo de máquinas virtuales de Azure que ejecutan Windows para admitir una implementación de SAP NetWeaver de alta disponibilidad

Duración: 60 minutos

En este ejercicio, configurará el sistema operativo de las máquinas virtuales de Azure que ejecutan Windows Server para acoger una implementación de SAP NetWeaver de alta disponibilidad.

### Tarea 1: Unión de máquinas virtuales de Azure Windows Server 2016 al dominio de Active Directory.

   > **Nota**: Antes de iniciar esta tarea, asegúrese de que las implementaciones de plantilla iniciadas en el ejercicio anterior se hayan completado correctamente. 

1. En Azure Portal, vaya a la hoja de la red virtual denominada **adVNET**, que se aprovisionó automáticamente en el primer ejercicio de este laboratorio.

1. Muestre la hoja **adVNET: servidores DNS** y observe que la red virtual está configurada con las direcciones IP privadas asignadas a los controladores de dominio implementados en el primer ejercicio de este laboratorio como sus servidores DNS.

1. En Azure Portal, inicie una sesión de PowerShell en Cloud Shell. 

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene los recursos que aprovisionó en la tarea anterior:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para unir las máquinas virtuales de Azure de Windows Server que implementó en la tercera tarea del ejercicio anterior al dominio **adatum.com** de Active Directory (asegúrese de reemplazar el marcador de posición `<password>` por la contraseña que especificó anteriormente en este laboratorio):

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1','i20-di-0','i20-di-1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

### Tarea 2: Examine la configuración de almacenamiento de las máquinas virtuales de Azure del nivel de base de datos.

1. En el equipo de laboratorio, en Azure Portal, vaya a la hoja **az12003b-vm0**.

1. En la hoja **az12003b-vm0**, conéctese a la máquina virtual de Azure az12003b-vm0 a través del Escritorio remoto. Cuando se le solicite, proporcione las siguientes credenciales:

    -   Iniciar sesión como: **estudiante**

    -   Contraseña: *la misma contraseña que especificó anteriormente en este laboratorio*

1. Desde la sesión de RDP a az12003b-vm0, use Escritorio remoto para conectarse a la máquina virtual de Azure **i20-db-0.adatum.com**. Cuando se le solicite, proporcione las siguientes credenciales:

    -   Iniciar sesión como: **ADATUM\\Student**

    -   Contraseña: *la misma contraseña que especificó anteriormente en este laboratorio*

1. Use Escritorio remoto para conectarse a la máquina virtual de Azure **i20-db-1.adatum.com** con las mismas credenciales.

1. En la sesión de RDP para i20-db-0.adatum.com, use Servicios de archivos y almacenamiento en el Administrador del servidor para examinar la configuración del disco. Tenga en cuenta que se ha configurado un único disco de datos a través de montajes de volumen para proporcionar almacenamiento para los archivos de base de datos y de registro. 

1. En la sesión de RDP para i20-db-1.adatum.com, use Servicios de archivos y almacenamiento en el Administrador del servidor para examinar la configuración del disco. Tenga en cuenta que se ha configurado un único disco de datos a través de montajes de volumen para proporcionar almacenamiento para los archivos de base de datos y de registro. 


### Tarea 3: Prepárese para la configuración de clústeres de conmutación por error en máquinas virtuales de Azure que ejecutan Windows Server 2016 para admitir una instalación de SAP NetWeaver de alta disponibilidad.

1. En la sesión de RDP para i20-db-0.adatum.com, inicie una sesión de Windows PowerShell ISE e instale las características de clústeres de conmutación por error y las herramientas administrativas remotas ejecutando lo siguiente en el par de servidores ASCS y DB que se convertirán en nodos de los clústeres de ASCS y SQL Server, respectivamente:

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Nota**: Esto podría dar lugar a un reinicio del sistema operativo invitado de las cuatro máquinas virtuales de Azure.

1. En el equipo de laboratorio, en Azure Portal, haga clic en **+ Crear un recurso**.

1. En la hoja **Nuevo**, inicie la creación de una nueva **Cuenta de almacenamiento** con la siguiente configuración:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure* |
    | **Grupo de recursos** | *el nombre del grupo de recursos en el que implementó las máquinas virtuales de Azure que hospedarán la implementación de SAP NetWeaver de alta disponibilidad* |
    | **Nombre de cuenta de almacenamiento** | *cualquier nombre único que tenga entre 3 y 24 letras y dígitos* |
    | **Ubicación** | *la misma región de Azure donde implementó las máquinas virtuales de Azure en el ejercicio anterior* |
    | **Rendimiento** | **Estándar** |
    | **Redundancia** | **Almacenamiento con redundancia local (LRS)** |
    | **Método de conectividad** | **Punto de conexión público (todas las redes)** |
    | **Exigir transferencia segura para las operaciones de API de REST** | **Habilitado** |
    | **Recursos compartidos de archivos grandes** | **Deshabilitado** |
    | **Eliminación temporal de blobs, contenedores y archivos** | **Deshabilitado** |
    | **Espacio de nombres jerárquico** | **Deshabilitado** |


### Tarea 4: Configure clústeres de conmutación por error en máquinas virtuales de Azure que ejecutan Windows Server 2016 para admitir un nivel de base de datos de la instalación de SAP NetWeaver de alta disponibilidad.

1. Si es necesario, desde la sesión de RDP a az12003b-vm0, use Escritorio remoto para volver a conectarse a la máquina virtual de Azure **i20-db-0.adatum.com**. Cuando se le solicite, proporcione las siguientes credenciales:

    -   Iniciar sesión como: **ADATUM\\Student**

    -   Contraseña: *la misma contraseña que especificó anteriormente en este laboratorio*

1. En la sesión de RDP para i20-db-0.adatum.com, en el Administrador del servidor, vaya a la vista **Servidor local** y desactive la **Configuración de seguridad mejorada de IE**.

1. En la sesión de RDP a i20-db-0.adatum.com, en el menú **Herramientas** en Administrador del servidor, inicie el **centro administrativo de Active Directory**.

1. En el centro administrativo de Active Directory, cree una nueva unidad organizativa denominada **Clústeres** en la raíz del dominio de adatum.com.

1. En el centro administrativo de Active Directory, mueva las cuentas de equipo de i20-db-0 y i20-db-1 desde el contenedor **Equipos** a la unidad organizativa **Clústeres**.

1. En la sesión de RDP a i20-db-0, inicie una sesión de Windows PowerShell ISE y cree un clúster mediante la ejecución de lo siguiente:

    ```
    $nodes = @('i20-db-0','i20-db-1')

    New-Cluster -Name az12003b-db-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.15
    ```

1. En la sesión de RDP a i20-db-0.adatum.com, cambie a la consola del **centro administrativo de Active Directory**.

1. En el centro administrativo de Active Directory, vaya a la unidad organizativa **Clústeres** y muestre su ventana **Propiedades**. 

1. En la ventana **Propiedades** de la unidad organizativa **Clústeres**, vaya a la sección **Extensiones** y muestre la pestaña **Seguridad**. 

1. En la pestaña **Seguridad**, haga clic en el botón **Avanzada** para abrir la ventana **Configuración de seguridad avanzada para clústeres**. 

1. En la pestaña **Permisos** de la ventana **Configuración de seguridad avanzada para clústeres**, haga clic en **Agregar**.

1. En la ventana **Entrada de permiso para clústeres**, haga clic en **Seleccionar entidad de seguridad**

1. En el cuadro de diálogo **Seleccionar usuario, cuenta de servicio o de grupo**, haga clic en **Tipos de objeto**, habilite la casilla situada junto a la entrada **Equipos** y haga clic en **Aceptar**. 

1. De nuevo en el cuadro de diálogo **Seleccionar usuario, equipo, cuenta de servicio o grupo**, en el cuadro de diálogo **Escriba el nombre del objeto que desea seleccionar**, escriba **az12003b-db-cl0** y haga clic en **Aceptar**.

1. En la ventana **Entrada de permiso para clústeres**, asegúrese de que **Permitir** aparece en la lista desplegable **Tipo**. A continuación, en la lista desplegable **Se aplica a**, seleccione **Este objeto y todos los objetos descendientes**. En la lista **Permisos**, active las casillas **Crear objetos de equipo** y **Eliminar objetos de equipo** y haga clic en **Aceptar** dos veces.

1. En la sesión de Windows PowerShell ISE, instale el módulo Az PowerShell ejecutando lo siguiente:

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. En la sesión de Windows PowerShell ISE, autentíquese mediante las credenciales de Azure AD mediante la ejecución de lo siguiente:

    ```
    Add-AzAccount
    ```

    > **Nota**: Cuando se le solicite, inicie sesión con la cuenta profesional o educativa o personal de Microsoft con el rol de propietario o colaborador en la suscripción de Azure que usa para este laboratorio.

1. En la sesión de Windows PowerShell ISE, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene la cuenta de almacenamiento que aprovisionó en la tarea anterior:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. En la sesión de Windows PowerShell ISE, ejecute lo siguiente para establecer el cuórum testigo en la nube del nuevo clúster:

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Para comprobar la configuración resultante, en la sesión RDP a i20-db-0.adatum.com, en el menú **Herramientas** de Administrador del servidor, inicie el **Administrador de clústeres de conmutación por error**.

1. En la consola del **Administrador de clústeres de conmutación por error**, revise la configuración del clúster **az12003b-db-cl0**, incluidos sus nodos, así como la configuración de red y testigo. Tenga en cuenta que el clúster no tiene ningún almacenamiento compartido.


### Tarea 6: Configure clústeres de conmutación por error en máquinas virtuales de Azure que ejecutan Windows Server 2016 para admitir un nivel ASCS de alta disponibilidad de la instalación de SAP NetWeaver.

> **Nota**: Asegúrese de que la implementación del clúster S2D que inició en la tarea 4 del ejercicio 1 se ha completado correctamente antes de iniciar esta tarea.

1. Desde la sesión de RDP a az12003b-vm0, use Escritorio remoto para conectarse a la máquina virtual de Azure **i20-ascs-0.adatum.com**. Cuando se le solicite, proporcione las siguientes credenciales:

    -   Iniciar sesión como: **ADATUM\\Student**

    -   Contraseña: *la misma contraseña que especificó anteriormente en este laboratorio*

1. En la sesión de RDP para i20-ascs-0.adatum.com, en el Administrador del servidor, vaya a la vista **Servidor local** y desactive la **Configuración de seguridad mejorada de IE**.

1. En la sesión de RDP a i20-ascs-0.adatum.com, en el menú **Herramientas** en Administrador del servidor, inicie el **centro administrativo de Active Directory**.

1. En el Centro de administración de Active Directory, vaya al contenedor **Equipos**. 

1. En el centro administrativo de Active Directory, mueva las cuentas de equipo de i20-ascs-0 y i20-ascs-1 desde el contenedor **Equipos** a la unidad organizativa **Clústeres**.

1. En la sesión de RDP a i20-ascs-0.adatum.com, inicie una sesión de Windows PowerShell ISE y cree un clúster mediante la ejecución de lo siguiente:

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1')

    New-Cluster -Name az12003b-ascs-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.16
    ```

1. En la sesión de RDP a i20-ascs-0.adatum.com, cambie a la consola del **centro administrativo de Active Directory**.

1. En el centro administrativo de Active Directory, vaya a la unidad organizativa **Clústeres** y muestre su ventana **Propiedades**. 

1. En la ventana **Propiedades** de la unidad organizativa **Clústeres**, vaya a la sección **Extensiones** y muestre la pestaña **Seguridad**. 

1. En la pestaña **Seguridad**, haga clic en el botón **Avanzada** para abrir la ventana **Configuración de seguridad avanzada para clústeres**. 

1. En la pestaña **Permisos** de la ventana **Configuración de seguridad avanzada para equipos**, haga clic en **Agregar**.

1. En la ventana **Entrada de permiso para clústeres**, haga clic en **Seleccionar entidad de seguridad**

1. En el cuadro de diálogo **Seleccionar usuario, cuenta de servicio o de grupo**, haga clic en **Tipos de objeto**, habilite la casilla situada junto a la entrada **Equipos** y haga clic en **Aceptar**. 

1. De nuevo en el cuadro de diálogo **Seleccionar usuario, equipo, cuenta de servicio o grupo**, en el cuadro de diálogo **Escriba el nombre del objeto que desea seleccionar**, escriba **az12003b-ascs-cl0** y haga clic en **Aceptar**.

1. En la ventana **Entrada de permiso para clústeres**, asegúrese de que **Permitir** aparece en la lista desplegable **Tipo**. A continuación, en la lista desplegable **Se aplica a**, seleccione **Este objeto y todos los objetos descendientes**. En la lista **Permisos**, active las casillas **Crear objetos de equipo** y **Eliminar objetos de equipo** y haga clic en **Aceptar** dos veces.

1. En la sesión de Windows PowerShell ISE, instale el módulo Az PowerShell ejecutando lo siguiente:

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. En la sesión de Windows PowerShell ISE, autentíquese mediante las credenciales de Azure AD mediante la ejecución de lo siguiente:

    ```
    Add-AzAccount
    ```

    > **Nota**: Cuando se le solicite, inicie sesión con la cuenta profesional o educativa o personal de Microsoft con el rol de propietario o colaborador en la suscripción de Azure que usa para este laboratorio.

1. En la sesión de Windows PowerShell ISE, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene la cuenta de almacenamiento que aprovisionó anteriormente en este ejercicio:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. En la sesión de Windows PowerShell ISE, ejecute lo siguiente para establecer el cuórum testigo en la nube del clúster:

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Para comprobar la configuración resultante, en la sesión RDP a i20-ascs-0.adatum.com, en el menú **Herramientas** de Administrador del servidor, inicie el **Administrador de clústeres de conmutación por error**.

1. En la consola del **Administrador de clústeres de conmutación por error**, revise la configuración del clúster **az12003b-ascs-cl0**, incluidos sus nodos, así como la configuración de red y testigo. Tenga en cuenta que el clúster no tiene ningún almacenamiento compartido.


### Tarea 7: Establecer permisos en el recurso compartido \\\\GLOBALHOST\\sapmnt

En esta tarea, establecerá permisos de nivel de recurso compartido en el recurso compartido **\\\\GLOBALHOST\\sapmnt**.

> **Nota**: De forma predeterminada, los permisos de control total solo se conceden a la cuenta de ADATUM\Student. 

1. En la sesión de Escritorio remoto para i20-ascs-0.adatum.com, desde la ventana **Windows PowerShell ISE**, ejecute lo siguiente:

    ```
    $remoteSession = New-CimSession -ComputerName SAPGLOBALHOST

    Grant-SmbShareAccess -Name sapmnt -AccountName 'ADATUM\Domain Admins' -AccessRight Full -CimSession $remoteSession -Force
   
    ```

### Tarea 8: Configurar los requisitos previos del sistema operativo para instalar los componentes de base de datos y ASCS de SAP NetWeaver

1. En la sesión de Escritorio remoto para i20-ascs-0.adatum.com, desde la sesión de Windows PowerShell ISE, ejecute lo siguiente para configurar las entradas del Registro necesarias para facilitar la instalación de componentes de ASCS de SAP y el uso de nombres virtuales:

    ```
    $nodes = ('i20-db-0','i20-db-1')

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanworkstation\parameters'
        $registryEntry = 'DisableCARetryOnInitialConnect'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\LSA'
        $registryEntry = 'DisableLoopbackCheck'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters'
        $registryEntry = 'DisableStrictNameChecking'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }
    ```

> **Result**: Después de completar este ejercicio, ha configurado el sistema operativo de máquinas virtuales de Azure que ejecutan Windows para admitir una implementación de SAP NetWeaver de alta disponibilidad


## Ejercicio 3: Eliminación de recursos de laboratorio

Duración: 10 minutos

En este ejercicio, quitará los recursos aprovisionados en este laboratorio.

#### Tarea 1: Apertura de Cloud Shell

1. En la parte superior del portal, haga clic en el icono de **Cloud Shell** para abrir el panel de Cloud Shell y elija PowerShell como shell.

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene el par de **Windows Server 2019 Datacenter** máquinas virtuales de Azure que aprovisionó en el primer ejercicio de este laboratorio:

    ```
    $resourceGroupNamePrefix = 'az12003b-'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para enumerar todos los grupos de recursos que creó en este laboratorio:

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. Compruebe que la salida contiene solo los grupos de recursos que creó en este laboratorio. Estos grupos se eliminarán en la siguiente tarea.

#### Tarea 2: Eliminación de los grupos de recursos

1. En el panel de Cloud Shell, ejecute el siguiente comando para eliminar los grupos de recursos que creó en este laboratorio

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. Cierre el panel de Cloud Shell.

> **Result**: después de completar este ejercicio, ha quitado los recursos usados en este laboratorio.
