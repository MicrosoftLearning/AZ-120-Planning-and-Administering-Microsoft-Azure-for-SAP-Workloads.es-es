---
lab:
  title: '01b: Implementación de clústeres de Windows en máquinas virtuales de Azure'
  module: Module 01 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 Módulo 1: Fundamentos de IaaS para SAP en Azure
# Laboratorio 1b: Implementación de clústeres de Windows en máquinas virtuales de Azure

Tiempo estimado: 120 minutos

Todas las tareas de este laboratorio se realizan desde Azure Portal (incluida una sesión de Cloud Shell de PowerShell)  

   > **Nota**: Cuando no se usa Cloud Shell, la máquina virtual de laboratorio debe tener el módulo Az de PowerShell instalado [**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Archivos de laboratorio: ninguno

## Escenario
  
Como preparación para la implementación de SAP NetWeaver en Azure, con SQL Server como sistema de administración de bases de datos, Adatum Corporation quiere explorar el proceso de implementación de la agrupación en clústeres en máquinas virtuales de Azure que ejecutan Windows Server 2022.

## Objetivos
  
Después de completar este laboratorio, podrá:

-   Aprovisionar los recursos de proceso de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad

-   Configurar el sistema operativo de máquinas virtuales de Azure que ejecutan Windows Server 2022 para admitir una implementación de SAP NetWeaver de alta disponibilidad.

-   Aprovisionar los recursos de red de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad

## Requisitos

-   Una suscripción a Microsoft Azure con el número suficiente de unidades vCPU DSv2 y Dsv3 disponibles (una máquina virtual Standard_DS1_v2 con una vCPU y cuatro máquinas virtuales Standard_D4s_v3 con cuatro vCPU cada una) en la región de Azure que tenga previsto usar para este laboratorio

-   Un equipo de laboratorio con un explorador web compatible con Azure Cloud Shell y acceso a Azure

> **Nota**: asegúrese de que la región de Azure que elija para la implementación de los recursos admite zonas de disponibilidad. Para obtener la lista de estas regiones, consulte (https://docs.microsoft.com/en-us/azure/availability-zones/az-overview). Considere la posibilidad de usar **Este de EE. UU.** o **Este de EE. UU. 2**.

## Ejercicio 1: Aprovisionar los recursos de proceso de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad

Duración: 50 minutes

En este ejercicio, implementará los componentes de proceso de infraestructura de Azure necesarios para configurar clústeres de conmutación por error en máquinas virtuales de Azure que ejecutan Windows Server 2022. Esto implicará la implementación de un par de controladores de dominio de Active Directory, seguido de un par de máquinas virtuales de Azure que ejecutan Windows Server 2022. Cada par de máquinas virtuales se colocará en zonas de disponibilidad independientes dentro de la misma red virtual. Para automatizar la implementación de controladores de dominio, usará una plantilla de inicio rápido de Azure Resource Manager disponible en <https://aka.ms/az120-1bdeploy>

### Tarea 1: implementación de un par de máquinas virtuales de Azure que ejecutan controladores de dominio de Active Directory de alta disponibilidad mediante una plantilla de Bicep

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
    $rgName = 'az12001b-ad-RG'
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
    $deploymentName = 'az1201b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
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

    - Vaya a la hoja **az1201b-ad-RG**, en el menú de navegación vertical del lado izquierdo, en la sección **Configuración**, seleccione **Implementaciones**.

    - En la hoja **az1201b-ad-RG \| Implementaciones**, seleccione la implementación que comienza por el prefijo **az1201b** y, en la hoja de implementación, seleccione **Volver a implementar**.

    - En la hoja **Implementación personalizada**, en el cuadro de texto **Administración de contraseña**, escriba la misma contraseña que usó durante la implementación original, seleccione **Revisar y crear** y, a continuación, seleccione **Crear**.

    - No espere a que se complete la reimplementación, sino que continúe con la siguiente tarea. La reimplementación debe tardar unos 3 minutos.

### Tarea 2: implementación de un par de máquinas virtuales de Azure que ejecutan Windows Server 2022 en diferentes zonas de disponibilidad

1. En el equipo de laboratorio, en Azure Portal, vaya a la hoja **Máquinas virtuales**, haga clic en **+ Crear**y, en el menú desplegable, seleccione **máquina virtual de Azure**.

1. En la hoja **Crear una máquina virtual**, inicie el aprovisionamiento de un **centro de datos de Windows Server 2022: Azure Edition: Gen2** máquina virtual de Azure con la siguiente configuración (deje todos los demás con sus valores predeterminados):
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure*  |
    | **Grupo de recursos** | *el nombre de un nuevo grupo de recursos* **az12001b-cl-RG** |
    | **Nombre de la máquina virtual** | **az12001b-cl-vm0** |
    | **Región** | *la misma región de Azure donde implementó las máquinas virtuales de Azure en la tarea anterior* |
    | **Opciones de disponibilidad** | **Zona de disponibilidad** |
    | **Zona de disponibilidad** | **Zona 1** |
    | **Imagen** | *seleccione* **Windows Server 2022 Datacenter: Azure Edition: Gen2** |
    | **Tamaño** | **Estándar D4s v3** |
    | **Nombre de usuario** | *el mismo nombre de usuario que especificó al implementar la plantilla de Bicep anteriormente en este ejercicio* |
    | **Contraseña** | *la misma contraseña que especificó al implementar la plantilla de Bicep anteriormente en este ejercicio* |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **¿Quiere usar una licencia de Windows Server existente?** | **No** |
    | **Tipo de disco del sistema operativo** | **SSD Premium** |
    | **Red virtual** | **adVNET** |
    | **Nombre de subred** | *una nueva subred denominada* **clSubnet** |
    | **Intervalo de direcciones de subred** | **10.0.1.0/24** |
    | **Dirección IP pública** | *una nueva dirección IP denominada* **az12001b-cl-vm0-ip** |
    | **Grupo de seguridad de red de NIC** | **Basic**  |
    | **Habilitación de la redes aceleradas** | **Activado** |
    | **Opciones de equilibrio de carga** | **Ninguno** |
    | **Habilitación de una identidad administrada asignada por el sistema** | **Desactivado** |
    | **Inicio de sesión con Azure AD** | **Desactivado** |
    | **Habilitación del apagado automático** | **Desactivado** |
    | **Opción de orquestación de revisiones** | **Actualizaciones manuales** |
    | **Diagnósticos de arranque** | **Deshabilitar** |
    | **Extensiones** | *Ninguno* |
    | **Etiquetas** | *Ninguno* |

1. No espere a que se complete el aprovisionamiento; continúe con el paso siguiente.

1. Aprovisione otro **Centro de datos de Windows Server 2022: Azure Edition: Gen2** máquina virtual de Azure con la siguiente configuración:
     
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure*  |
    | **Grupo de recursos** | *el nombre del grupo de recursos que usó al implementar la primera **Windows Server 2022 Datacenter: Edición de Azure: Gen2** máquina virtual de Azure en esta tarea* |
    | **Nombre de la máquina virtual** | **az12001b-cl-vm1** |
    | **Región** | *la misma región de Azure donde implementó la primera **Windows Server 2022 Datacenter: Edición de Azure: Gen2** máquina virtual de Azure en esta tarea* |
    | **Opciones de disponibilidad** | **Zona de disponibilidad** |
    | **Zona de disponibilidad** | **Zona 2** |
    | **Imagen** | *seleccione* **Windows Server 2022 Datacenter: Azure Edition: Gen2** |
    | **Tamaño** | **Estándar D4s v3** |
    | **Nombre de usuario** | *el mismo nombre de usuario que especificó al implementar la plantilla de Bicep anteriormente en este ejercicio* |
    | **Contraseña** | *la misma contraseña que especificó al implementar la plantilla de Bicep anteriormente en este ejercicio* |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **¿Quiere usar una licencia de Windows Server existente?** | **No** |
    | **Tipo de disco del sistema operativo** | **SSD Premium** |
    | **Red virtual** | **adVNET** |
    | **Nombre de subred** | **clSubnet** |
    | **Dirección IP pública** | *una nueva dirección IP denominada* **az12001b-cl-vm1-ip** |
    | **Grupo de seguridad de red de NIC** | **Basic**  |
    | **Habilitación de la redes aceleradas** | **Activado** |
    | **Opciones de equilibrio de carga** | **Ninguno** |
    | **Inicio de sesión con Azure AD** | **Desactivado** |
    | **Habilitación del apagado automático** | **Desactivado** |
    | **Opción de orquestación de revisiones** | **Actualizaciones manuales** |
    | **Diagnósticos de arranque** | **Deshabilitar** |
    | **Extensiones** | *Ninguno* |
    | **Etiquetas** | *Ninguno* |

1. Espere a que se complete el aprovisionamiento. Esta operación solo tardará unos minutos.

### Tarea 3: Creación y configuración de una instancia de discos de máquinas virtuales de Azure

1. En Azure Portal, inicie una sesión de PowerShell en Cloud Shell. 

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene los recursos que aprovisionó en la tarea anterior:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear el primer conjunto de cuatro discos administrados que asociará a la primera máquina virtual de Azure que implementó en la tarea anterior:

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location
    
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm0').Zones

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear el segundo conjunto de cuatro discos administrados que asociará a la segunda máquina virtual de Azure que implementó en la tarea anterior:

    ```
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm1').Zones
    
    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone
        
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1. En Azure Portal, vaya a la hoja de la primera máquina virtual de Azure que aprovisionó en la tarea anterior (**az12001b-cl-vm0**).

1. En la hoja **az12001b-cl-vm0**, vaya a la hoja **az12001b-cl-vm0: discos**.

1. En la hoja **az12001b-cl-vm0: discos**, conecte discos de datos con la siguiente configuración a az12001b-cl-vm0:
   
   | Configuración | Value |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nombre del disco** | **az12001b-cl-vm0-DataDisk0** |
   | **Grupo de recursos** | *el nombre del grupo de recursos que usó al implementar el par de máquinas virtuales de Azure de **Windows Server 2022 Datacenter** en la tarea anterior* |
   | **ALMACENAMIENTO EN CACHÉ DE HOST** | **Solo lectura** |

1. Repita el paso anterior para conectar los tres discos restantes con el prefijo **az12001b-cl-vm0-DataDisk** (para el total de cuatro). Asigne el número LUN que coincida con el último carácter del nombre del disco. Para el último disco (LUN **3**), establezca ALMACENAMIENTO EN CACHÉ DE HOST en **Ninguno**.

1. Guarde los cambios. 

1. En Azure Portal, vaya a la hoja de la segunda máquina virtual de Azure que aprovisionó en la tarea anterior (**az12001b-cl-vm1**).

1. En la hoja **az12001b-cl-vm1**, vaya a la hoja **az12001b-cl-vm1: discos**.

1. En la hoja **az12001b-cl-vm1: discos**, conecte discos de datos con la siguiente configuración a az12001b-cl-vm1:
     
   | Configuración | Value |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nombre del disco** | **az12001b-cl-vm1-DataDisk0** |
   | **Grupo de recursos** | *el nombre del grupo de recursos que usó al implementar el par de máquinas virtuales de Azure de **Windows Server 2022 Datacenter** en la tarea anterior* |
   | **ALMACENAMIENTO EN CACHÉ DE HOST** | **Solo lectura** |

1. Repita el paso anterior para conectar los tres discos restantes con el prefijo **az12001b-cl-vm1-DataDisk** (para el total de cuatro). Asigne el número LUN que coincida con el último carácter del nombre del disco. Para el último disco (LUN **3**), establezca ALMACENAMIENTO EN CACHÉ DE HOST en **Ninguno**.

1. Guarde los cambios. 

> **Result**: Después de completar este ejercicio, ha aprovisionado los recursos de proceso de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad.


## Ejercicio 2: configuración del sistema operativo de máquinas virtuales de Azure que ejecutan Windows Server 2022 Datacenter para admitir una instalación de SAP NetWeaver de alta disponibilidad

Duración: 40 minutos

### Tarea 1: unión de máquinas virtuales de Windows Server 2022 Datacenter al dominio de Active Directory.

   > **Nota**: antes de iniciar esta tarea, asegúrese de que la implementación de plantilla que inició en la última tarea del ejercicio anterior se ha completado correctamente. 

1. En Azure Portal, vaya a la hoja de la red virtual **adVNET**, que se aprovisionó automáticamente en el primer ejercicio de este laboratorio.

1. Muestre la hoja **adVNET: servidores DNS** y observe que la red virtual está configurada con las direcciones IP privadas asignadas a los controladores de dominio implementados en el primer ejercicio de este laboratorio como sus servidores DNS.

1. En Azure Portal, inicie una sesión de PowerShell en Cloud Shell. 

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene el par de **Windows Server 2022 Datacenter: Azure Edition: Gen2** máquinas virtuales de Azure que aprovisionó en el ejercicio anterior:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para unir las máquinas virtuales de Azure de Windows Server 2022 que implementó en la segunda tarea del ejercicio anterior al dominio **adatum.com** de Active Directory (reemplace los marcadores de posición `<username>` y `<password>` por el nombre y la contraseña de la cuenta de usuario administrativa que especificó al implementar la plantilla de Bicep en el primer ejercicio de este laboratorio):

    ```
    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\<username>", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1. Espere a que se complete el script antes de continuar con la siguiente tarea.


### Tarea 2: Configure el almacenamiento en máquinas virtuales de Azure que ejecutan Windows Server 2022 para admitir una instalación de SAP NetWeaver de alta disponibilidad.

1. En Azure Portal, vaya a la hoja de la máquina virtual **az12001b-cl-vm0**, que aprovisionó en el primer ejercicio de este laboratorio.

1. En la hoja **az12001b-cl-vm0**, conéctese al sistema operativo invitado de máquina virtual mediante Escritorio remoto. Cuando se le pida que se autentique, proporcione las credenciales de la cuenta de usuario administrativa que especificó al implementar la plantilla de Bicep en el primer ejercicio de este laboratorio. 

    > **Nota**: asegúrese de iniciar sesión con la cuenta de dominio **ADATUM**, en lugar de la cuenta de nivel de sistema operativo (es decir, asegúrese de que el nombre de usuario esté precedido por el prefijo **ADATUM\\**.

1. En la sesión de RDP a az12001b-cl-vm0, en el Administrador del servidor, vaya al nodo **Servicios de archivos y almacenamiento** -> **Servidores**. 

1. Vaya a la vista **Grupos de almacenamiento** y compruebe que ve todos los discos conectados a la máquina virtual de Azure en el ejercicio anterior.

1. Use el **Asistente para nuevos grupos de almacenamiento** para crear un nuevo grupo de almacenamiento con la siguiente configuración:
     
    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **Grupo de almacenamiento de datos** |
    | **Discos físicos** | *seleccione los tres discos con números de disco correspondientes a los tres primeros números de LUN (0-2) y establezca su asignación en* **Automático** |

    > **Nota**: use la entrada de la columna **Chasis** para identificar el número de **LUN**.

1. Use el **Asistente para nuevo disco virtual** para crear un nuevo disco virtual con la siguiente configuración:
     
    | Configuración | Valor |
    |   --    |  --   |
    | **Nombre de disco virtual** | **Disco virtual de datos** |
    | **Diseño de almacenamiento** | **Sencillo** |
    | **Aprovisionamiento** | **Fija** |
    | **Tamaño** | **Tamaño máximo** |

1. Use el **Asistente para nuevo volumen** para crear un nuevo volumen con la siguiente configuración:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Servidor y disco** | *acepte los valores predeterminados* |
    | **Tamaño** | *acepte los valores predeterminados* |
    | **Unidad** | **M** |
    | **Sistema de archivos** | **ReFS** |
    | **tamaño de unidad de asignación** | **Valor predeterminado** |
    | **Etiqueta de volumen** | **Data** |

1. De nuevo en la vista **Grupos de almacenamiento**, use el **Asistente para nuevos grupos de almacenamiento** para crear un nuevo grupo de almacenamiento con la siguiente configuración:
    
    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **Grupo de almacenamiento de registro** |
    | **Discos físicos** | *seleccione los últimos cuatro discos y establezca su asignación en* **Automático** |

1. Use el **Asistente para nuevo disco virtual** para crear un nuevo disco virtual con la siguiente configuración:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Nombre de disco virtual** | **Disco virtual de registro** |
    | **Diseño de almacenamiento** | **Sencillo** |
    | **Aprovisionamiento** | **Fija** |
    | **Tamaño** | **Tamaño máximo** |

1. Use el **Asistente para nuevo volumen** para crear un nuevo volumen con la siguiente configuración:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Servidor y disco** | *acepte los valores predeterminados* |
    | **Tamaño** | *acepte los valores predeterminados* |
    | **Unidad** | **L** |
    | **Sistema de archivos** | **ReFS** |
    | **tamaño de unidad de asignación** | **Valor predeterminado** |
    | **Etiqueta de volumen** | **Registro** |

1. Repita el paso anterior de esta tarea para configurar el almacenamiento en az12001b-cl-vm1.

### Tarea 3: Prepárese para la configuración de clústeres de conmutación por error en máquinas virtuales de Azure que ejecutan Windows Server 2022 para admitir una instalación de SAP NetWeaver de alta disponibilidad.

1. En la sesión de RDP a az12001b-cl-vm0, inicie una sesión de Windows PowerShell ISE e instale las características de Clústeres de conmutación por error y Herramientas remotas administrativas en az12001b-cl-vm0 y az12001b-cl-vm1 ejecutando lo siguiente:

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Nota**: esto provocará el reinicio del sistema operativo invitado de ambas máquinas virtuales de Azure.

1. En el equipo de laboratorio, en Azure Portal, haga clic en **+ Crear un recurso**.

1. En la hoja **Nuevo**, inicie la creación de una nueva **Cuenta de almacenamiento** con la siguiente configuración:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure* |
    | **Grupo de recursos** | *el nombre del grupo de recursos que contiene el par de máquinas virtuales de Azure de **Windows Server 2022 Datacenter** que aprovisionó en el ejercicio anterior* |
    | **Nombre de cuenta de almacenamiento** | *cualquier nombre único que tenga entre 3 y 24 letras y dígitos* |
    | **Ubicación** | *la misma región de Azure donde implementó las máquinas virtuales de Azure en el ejercicio anterior* |
    | **Rendimiento** | **Estándar** |
    | **Redundancia** | **Almacenamiento con redundancia local (LRS)** |
    | **Método de conectividad** | **Punto de conexión público (todas las redes)** |
    | **Exigir transferencia segura para las operaciones de API de REST** | **Habilitado** |
    | **Recursos compartidos de archivos grandes** | **Deshabilitado** |
    | **Eliminación temporal de blobs, contenedores y archivos** | **Deshabilitado** |
    | **Espacio de nombres jerárquico** | **Deshabilitado** |

### Tarea 4: Configure clústeres de conmutación por error en máquinas virtuales de Azure que ejecutan Windows Server 2022 para admitir una instalación de SAP NetWeaver de alta disponibilidad.

1. En Azure Portal, vaya a la hoja de la máquina virtual **az12001b-cl-vm0**, que aprovisionó en el primer ejercicio de este laboratorio.

1. En la hoja **az12001b-cl-vm0**, conéctese al sistema operativo invitado de máquina virtual mediante Escritorio remoto. Cuando se le pida que se autentique, proporcione las credenciales de la cuenta de usuario administrativa que especificó al implementar la plantilla de Bicep en el primer ejercicio de este laboratorio.

1. En la sesión de RDP a az12001b-cl-vm0, en el menú **Herramientas** en Administrador del servidor, inicie el **centro administrativo de Active Directory**.

1. En el centro administrativo de Active Directory, cree una nueva unidad organizativa denominada **Clústeres** en la raíz del dominio de adatum.com.

1. En el centro administrativo de Active Directory, mueva las cuentas de equipo de **az12001b-cl-vm0** y **az12001b-cl-vm1** desde el contenedor **Equipos** a la unidad organizativa **Clústeres**.

1. En la sesión de RDP a az12001b-cl-vm0, inicie una sesión de Windows PowerShell ISE y cree un clúster mediante la ejecución de lo siguiente:

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1. En la sesión de RDP a az12001b-cl-vm0, cambie a la consola del **centro administrativo de Active Directory**.

1. En el centro administrativo de Active Directory, vaya a la unidad organizativa **Clústeres** y muestre su ventana **Propiedades**. 

1. En la ventana **Propiedades** de la unidad organizativa **Clústeres**, vaya a la sección **Extensiones** y muestre la pestaña **Seguridad**. 

1. En la pestaña **Seguridad**, haga clic en el botón **Avanzada** para abrir la ventana **Configuración de seguridad avanzada para clústeres**. 

1. En la pestaña **Permisos** de la ventana **Configuración de seguridad avanzada para clústeres**, haga clic en **Agregar**.

1. En la ventana **Entrada de permiso para clústeres**, haga clic en **Seleccionar entidad de seguridad**

1. En el cuadro de diálogo **Seleccionar usuario, cuenta de servicio o de grupo**, haga clic en **Tipos de objeto**, habilite la casilla situada junto a la entrada **Equipos** y haga clic en **Aceptar**. 

1. De nuevo en el cuadro de diálogo **Seleccionar usuario, equipo, cuenta de servicio o grupo**, en el cuadro de diálogo **Escriba el nombre del objeto que desea seleccionar**, escriba **az12001b-cl-cl0** y haga clic en **Aceptar**.

1. En la ventana **Entrada de permiso para clústeres**, asegúrese de que **Permitir** aparece en la lista desplegable **Tipo**. A continuación, en la lista desplegable **Se aplica a**, seleccione **Este objeto y todos los objetos descendientes**. En la lista **Permisos**, active las casillas **Crear objetos de equipo** y **Eliminar objetos de equipo** y haga clic en **Aceptar** dos veces.

1. En la sesión de Windows PowerShell ISE, instale el módulo Az PowerShell ejecutando lo siguiente:

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. En la sesión de Windows PowerShell ISE, autentíquese mediante las credenciales de Azure AD mediante la ejecución de lo siguiente:

    ```
    Add-AzAccount
    ```

    > **Nota**: Cuando se le solicite, inicie sesión con la cuenta profesional o educativa o personal de Microsoft con el rol de propietario o colaborador en la suscripción de Azure que usa para este laboratorio.

1. En la sesión de Windows PowerShell ISE, establezca el cuórum testigo en la nube del nuevo clúster ejecutando lo siguiente:

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Para comprobar la configuración resultante, en la sesión RDP a az12001b-cl-vm0, en el menú **Herramientas** de Administrador del servidor, inicie el **Administrador de clústeres de conmutación por error**.

1. En la consola del **Administrador de clústeres de conmutación por error**, revise la configuración del clúster **az12001b-cl-cl0**, incluidos sus nodos, así como la configuración de red y testigo. Tenga en cuenta que el clúster no tiene ningún almacenamiento compartido.

1. Finalice la sesión RDP en az12001b-cl-vm0.

> **Result**: Después de completar este ejercicio, ha configurado el sistema operativo de máquinas virtuales de Azure que ejecutan Windows Server 2022 para admitir una instalación de SAP NetWeaver de alta disponibilidad


## Ejercicio 3: Aprovisionar los recursos de red de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad

Duración: 30 minutos

En este ejercicio, implementará Azure Load Balancer para dar cabida a las instalaciones en clúster de SAP NetWeaver.

### Tarea 1: Configure máquinas virtuales de Azure para facilitar la configuración del equilibrio de carga.

   > **Nota**: dado que va a configurar un par de Azure Load Balancer de la SKU de Stardard, primero debe quitar las direcciones IP públicas asociadas a adaptadores de red de dos máquinas virtuales de Azure que servirán como grupo de back-end con equilibrio de carga.

1. En el equipo de laboratorio, en Azure Portal, vaya a la hoja de la máquina virtual de Azure **az12001b-cl-vm0**. 

1. En la hoja **az12001b-cl-vm0**, vaya a la hoja de la dirección IP pública **az12001b-cl-vm0-ip** asociada a su adaptador de red.

1. En la hoja **az12001b-cl-vm0-ip**, primero desasocie la dirección IP pública de la interfaz de red y, a continuación, elimínela.

1. En Azure Portal, vaya a la hoja de la máquina virtual de Azure **az12001b-cl-vm1**. 

1. En la hoja **az12001b-cl-vm1**, vaya a la hoja de la dirección IP pública **az12001b-cl-vm1-ip** asociada a su adaptador de red.

1. En la hoja **az12001b-cl-vm1-ip**, primero desasocie la dirección IP pública de la interfaz de red y, a continuación, elimínela.

1. En Azure Portal, vaya a la hoja de la máquina virtual de Azure **az12001a-vm0**.

1. En la hoja **az12001a-vm0**, vaya a su hoja **Redes**. 

1. En la hoja **az12001a-vm0: redes**, vaya a la interfaz de red de az12001a-vm0. 

1. En la hoja de la interfaz de red de az12001a-vm0, vaya a su hoja configuraciones IP y, desde allí, muestre su hoja **ipconfig1**.

1. En la hoja **ipconfig1**, establezca la asignación de direcciones IP privadas en **Estática** y guarde el cambio.

1. En Azure Portal, vaya a la hoja de la máquina virtual de Azure **az12001a-vm1**.

1. En la hoja **az12001a-vm1**, vaya a su hoja **Redes**. 

1. En la hoja **az12001a-vm1: redes**, vaya a la interfaz de red de az12001a-vm1. 

1. En la hoja de la interfaz de red de az12001a-vm1, vaya a su hoja configuraciones IP y, desde allí, muestre su hoja **ipconfig1**.

1. En la hoja **ipconfig1**, establezca la asignación de direcciones IP privadas en **Estática** y guarde el cambio.

### Tarea 2: creación y configuración de equilibradores de carga de Azure que controlan el tráfico entrante

1. En Azure Portal, haga clic en **+ Crear un recurso**.

1. En la hoja **Nuevo**, inicie la creación de una instancia de Azure Load Balancer con la siguiente configuración:
    
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure* |
    | **Grupo de recursos** | *el nombre del grupo de recursos que contiene el par de máquinas virtuales de Azure de **Windows Server 2022 Datacenter** que aprovisionó en el primer ejercicio de este laboratorio* |
    | **Nombre** | **az12001b-cl-lb0** |
    | **Región** | la misma región de Azure en la que implementó máquinas virtuales de Azure en el primer ejercicio de este laboratorio* |
    | **SKU** | **Estándar** |
    | **Tipo** | **Interno** |
    | **Nombre de IP de front-end** | **frontend-ip1** |
    | **Red virtual** | **adVNET** |
    | **Subred** | **clSubnet** |
    | **Asignación de dirección IP** | **Estática** |
    | **Dirección IP** | **10.0.1.240** |
    | **Zona de disponibilidad** | **Con redundancia de zona** |

1. Espere hasta que se aprovisione el equilibrador de carga y vaya a su hoja en Azure Portal.

1. En la hoja **az12001b-cl-lb0**, agregue un grupo de back-end con la siguiente configuración:
    
    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **az12001b-cl-lb0-bepool** |
    | **Red virtual** | **adVNET** |
    | **Configuración del grupo de back-end** | **Dirección IP** |
    | **Dirección IP** | **10.0.1.4** Nombre de recurso **az1201b-cl-vm0** |
    | **Dirección IP** | **10.0.1.5** Nombre de recurso **az1201b-cl-vm1** |

1. En la hoja **az12001b-cl-lb0**, agregue un sondeo de estado con la siguiente configuración:
    
    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **az12001b-cl-lb0-hprobe** |
    | **Protocolo** | **TCP** |
    | **Puerto** | **59999** |
    | **Intervalo** | **5** *segundos* |
    | **Umbral incorrecto** | **2** *errores consecutivos* |

1. En la hoja **az12001b-cl-lb0**, agregue una regla de equilibrio de carga de red con la siguiente configuración:
     
    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **az12001b-cl-lb0-lbruletcp1433** |
    | **Versión de IP** | **IPv4** |
    | **Frontend IP address** (Dirección IP de front-end) | **10.0.1.240 (LoadBalancerFrontEnd)** |
    | **Puertos de alta disponibilidad** | **Deshabilitado** |
    | **Protocolo** | **TCP** |
    | **Puerto** | **1433** |
    | **Puerto back-end** | **1433** |
    | **Grupo de back-end** | **az12001b-cl-lb0-bepool (2 virtual machines)** |
    | **Sondeo de estado** | **az12001b-cl-lb0-hprobe (TCP:59999)** |
    | **Persistencia de la sesión** | **None** |
    | **Tiempo de espera de inactividad (minutos)** | **4** |
    | **Restablecimiento de TCP** | **Deshabilitado** |
    | **IP flotante (Direct Server Return)** | **Habilitado** |

### Tarea 3: creación y configuración de equilibradores de carga de Azure que controlan el tráfico saliente

1. Desde Azure Portal, inicie una sesión de PowerShell en Cloud Shell. 

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene el par de **Windows Server 2022 Datacenter** máquinas virtuales de Azure que aprovisionó en el primer ejercicio de este laboratorio:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear la dirección IP pública que usará el segundo equilibrador de carga:

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear el segundo equilibrador de carga:

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1. Cierre el panel de Cloud Shell.

1. En Azure Portal, vaya a la hoja que muestra las propiedades de Azure Load Balancer **az12001b-cl-lb1**.

1. En la hoja **az12001b-cl-lb1**, haga clic en **Grupos de back-end**.

1. En la hoja **az12001b-cl-lb1: grupos de back-end**, haga clic en **az12001b-cl-lb1-bepool**.

1. En la hoja **az12001b-cl-lb1-bepool**, especifique la siguiente configuración y haga clic en **Guardar**:
    
    | Configuración | Value |
    |   --    |  --   |
    | **Red virtual** | **adVNET (4 VM)** |
    | **Máquina virtual** | **az12001b-cl-vm0**  DIRECCIÓN IP: **ipconfig1** |
    | **Máquina virtual** | **az12001b-cl-vm1**  DIRECCIÓN IP: **ipconfig1** |

1. En la hoja **az12001b-cl-lb1**, haga clic en **Sondeos de estado**.

1. En la hoja **az12001b-cl-lb1: sondeos de estado**, agregue un sondeo de estado con la siguiente configuración:
    
    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **az12001b-cl-lb1-hprobe** |
    | **Protocolo** | **TCP** |
    | **Puerto** | **80** |
    | **Intervalo** | **5** *segundos* |
    | **Umbral incorrecto** | **2** *errores consecutivos* |

1. En la hoja **az12001b-cl-lb1**, haga clic en **Reglas de equilibrio de carga**.

1. En la hoja **az12001b-cl-lb1: reglas de equilibrio de carga**, agregue una regla de equilibrio de carga de red con la siguiente configuración:
    
    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **az12001b-cl-lb1-lbharule** |
    | **Versión de IP** | **IPv4** |
    | **Frontend IP address** (Dirección IP de front-end) | *acepte el valor predeterminado* |
    | **Puertos de alta disponibilidad** | **Deshabilitado** |
    | **Protocolo** | **TCP** |
    | **Puerto** | **80** |
    | **Puerto back-end** | **80** |
    | **Grupo de back-end** | **az12001b-cl-lb1-bepool (2 máquinas virtuales)** |
    | **Sondeo de estado** | **az12001b-cl-lb1-hprobe (TCP:80)** |
    | **Persistencia de la sesión** | **None** |
    | **Tiempo de espera de inactividad (minutos)** | **4** |
    | **Restablecimiento de TCP** | **Deshabilitado** |
    | **IP flotante (Direct Server Return)** | **Deshabilitado** |

### Tarea 4: Implementación de un host de salto

   > **Nota**: dado que dos máquinas virtuales de Azure en clúster ya no son accesibles directamente desde Internet, implementará una máquina virtual de Azure que ejecute Windows Server 2022 Datacenter que servirá como host de salto. 

1. En el equipo de laboratorio, en Azure Portal, vaya a la hoja **Máquinas virtuales**, haga clic en **+ Crear**y, en el menú desplegable, seleccione **máquina virtual de Azure**.

1. En la hoja **Crear una máquina virtual**, inicie el aprovisionamiento de un **centro de datos de Windows Server 2022: Azure Edition: Gen2** máquina virtual de Azure con la siguiente configuración:
     
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | *el nombre de la suscripción de Azure*  |
    | **Grupo de recursos** | *el nombre del grupo de recursos que contiene el par de **Windows Server 2022 Datacenter: Azure Edition: Gen2** máquinas virtuales de Azure que aprovisionó en el primer ejercicio de este laboratorio* |
    | **Nombre de la máquina virtual** | **az12001b-vm2** |
    | **Región** | *la misma región de Azure en la que implementó máquinas virtuales de Azure en el primer ejercicio de este laboratorio* |
    | **Opciones de disponibilidad** | **No se requiere redundancia de la infraestructura** |
    | **Imagen** | *seleccione* **Windows Server 2022 Datacenter: Azure Edition: Gen2** |
    | **Tamaño** | **Estándar DS1 v2*** o similar* |
    | **Nombre de usuario** | *el mismo nombre de usuario que especificó al implementar la plantilla de Bicep en el primer ejercicio de este laboratorio* |
    | **Contraseña** | *la misma contraseña que especificó al implementar la plantilla de Bicep en el primer ejercicio de este laboratorio* |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **¿Quiere usar una licencia de Windows Server existente?** | **No** |
    | **Tipo de disco del sistema operativo** | **HDD estándar** |
    | **Red virtual** | **adVNET** |
    | **Nombre de subred** | *una nueva subred denominada* **bastionSubnet** |
    | **Intervalo de direcciones de subred** | **10.0.255.0/24** |
    | **Dirección IP pública** | *una nueva dirección IP denominada* **az12001b-vm2-ip** |
    | **Grupo de seguridad de red de NIC** | **Basic**  |
    | **Puertos de entrada públicos** | **Permitir los puertos seleccionados** |
    | **Puertos de entrada seleccionados** | **RDP (3389)** |
    | **Habilitación de la redes aceleradas** | **Desactivado** |
    | **Opciones de equilibrio de carga** | **Ninguno** |
    | **Habilitación de una identidad administrada asignada por el sistema** | **Desactivado** |
    | **Inicio de sesión con Azure AD** | **Desactivado** |
    | **Habilitación del apagado automático** | **Desactivado** |
    | **Diagnósticos de arranque** | **Deshabilitar** |
    | **Habilitación de diagnósticos del SO invitado** | **Desactivado** |
    | **Extensiones** | *Ninguno* |
    | **Etiquetas** | *Ninguno* |

1. Espere a que se complete el aprovisionamiento. Esta operación solo tardará unos minutos.

1. Conecte a la máquina virtual de Azure recién aprovisionada a través de RDP. 

1. Dentro de la sesión RDP a az12001b-vm2, asegúrese de que puede establecer la sesión RDP en az12001b-cl-vm0 y az12001b-cl-vm1 a través de sus direcciones IP privadas (10.0.1.4 y 10.0.1.5, respectivamente). 

> **Result**: Después de completar este ejercicio, ha aprovisionado los recursos de red de Azure necesarios para admitir implementaciones de SAP NetWeaver de alta disponibilidad

## Ejercicio 4: Eliminación de recursos de laboratorio

Duración: 10 minutos

En este ejercicio, quitará los recursos aprovisionados en este laboratorio.

#### Tarea 1: Apertura de Cloud Shell

1. En la parte superior del portal, haga clic en el icono de **Cloud Shell** para abrir el panel de Cloud Shell y elija PowerShell como shell.

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `$resourceGroupName` en el nombre del grupo de recursos que contiene el par de **Windows Server 2022 Datacenter** máquinas virtuales de Azure que aprovisionó en el primer ejercicio de este laboratorio:

    ```
    $resourceGroupNamePrefix = 'az12001b-'
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
