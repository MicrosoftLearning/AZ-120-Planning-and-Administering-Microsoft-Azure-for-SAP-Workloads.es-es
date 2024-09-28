---
lab:
  title: '02a: Implementación de clústeres de Linux en máquinas virtuales de Azure'
  module: Module 02 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 Módulo 2: Fundamentos de IaaS para SAP en Azure
# Laboratorio 2a: Implementación de clústeres de Linux en máquinas virtuales de Azure

Tiempo estimado: 90 minutos

Todas las tareas de este laboratorio se realizan desde Azure Portal (incluida la sesión de Cloud Shell de Bash)  

   > **Nota**: cuando no se usa Cloud Shell, la máquina virtual de laboratorio debe tener la CLI de Azure instalada [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows) e incluir un cliente de SSH, como PuTTY, disponible en [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

Archivos de laboratorio: ninguno

## Escenario
  
Como preparación para la implementación de SAP HANA en Azure, Adatum Corporation quiere explorar el proceso de implementación de la agrupación en clústeres en máquinas virtuales de Azure que ejecutan la distribución SUSE de Linux.

## Objetivos
  
Después de completar este laboratorio, podrá:

- Aprovisionamiento de los recursos de proceso de Azure necesarios para admitir implementaciones de SAP HANA de alta disponibilidad

- Configuración del sistema operativo de máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP HANA de alta disponibilidad

- Aprovisionamiento de los recursos de red de Azure necesarios para admitir implementaciones de SAP HANA de alta disponibilidad

## Requisitos

- Una suscripción a Microsoft Azure con un número suficiente de unidades vCPU de tipo DSv3 (2 x 4) y DSv2 (1 x 1) disponibles

- Un equipo de laboratorio con un explorador web compatible con Azure Cloud Shell y acceso a Azure

## Ejercicio 1: Aprovisionar los recursos de proceso de Azure necesarios para admitir implementaciones de SAP HANA de alta disponibilidad

Duración: 30 minutos

En este ejercicio, implementará los componentes de proceso de infraestructura de Azure necesarios para configurar la agrupación en clústeres de Linux. Esto implicará la creación de un par de máquinas virtuales de Azure que ejecutan Linux SUSE en el mismo conjunto de disponibilidad y aprovisionamiento de Azure Bastion.

### Tarea 1: Implementación de máquinas virtuales de Azure que ejecutan SUSE de Linux

1. En el equipo de laboratorio, inicie un explorador web y vaya a Azure Portal en **https://portal.azure.com**.

1. Si se le solicita, inicie sesión con la cuenta profesional o educativa o personal de Microsoft con el rol de propietario o colaborador en la suscripción de Azure que usará para este laboratorio.

1. En Azure Portal, use el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a la hoja **Grupos con ubicación por proximidad** y, en la hoja **Grupos de selección de ubicación de proximidad**, seleccione **+ crear**.

1. En la pestaña **Aspectos básicos** de la hoja **Crear grupos con ubicación por proximidad**, especifica la siguiente configuración y selecciona **Revisar + crear** (deja los demás con sus valores predeterminados):

    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | el nombre de la suscripción de Azure |
    | Sección **Grupo de recursos** | Selecciona **Crear nuevo**, escribe **az12001a-RG** y selecciona **Aceptar** |
    | **Nombre del grupo con ubicación por proximidad** | **az12001a-ppg** |
    | **Región** | la región de Azure donde tiene suficientes cuotas de vCPU |
    | **Tamaños de máquinas virtuales** | **Estándar D4s v3** |

   > **Nota**: Considere la posibilidad de usar las regiones **Este de EE. UU.** o **Este de EE. UU. 2** para la implementación de los recursos.

1. En la pestaña **Revisar y crear** de la hoja **Crear grupos con ubicación por proximidad**, seleccione **Crear**.

   > **Nota**: Espere a que se complete el aprovisionamiento. Debería tardar menos de un minuto.

1. En Azure Portal, use el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a la hoja **Máquinas virtuales** y, después, en la hoja **Máquinas virtuales**, seleccione **+ Crear** y, en el menú desplegable, seleccione **Máquina virtual** de Azure.

1. En la pestaña **Aspectos básicos** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Discos >** (deje todas las demás configuraciones con su valor predeterminado):

    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | el nombre de la suscripción de Azure |
    | **Grupo de recursos** | El nombre del grupo de recursos que has creado anteriormente en esta tarea |
    | **Nombre de la máquina virtual** | **az12001a-vm0** |
    | **Región** | El nombre de la región de Azure que has elegido al crear el grupo con ubicación por proximidad |
    | **Opciones de disponibilidad** | **El conjunto de disponibilidad** |
    | **El conjunto de disponibilidad** | Un nuevo conjunto de disponibilidad denominado **az12001a-avset** con 2 dominios de error y 5 dominios de actualización |
    | **tipo de seguridad** | **Estándar** |
    | **Imagen** | **SUSE Enterprise Linux para SAP 15 SP5 - BYOS - x64 Gen 2** |
    | **ejecución con Azure Spot Discount** | deshabilitado |
    | **Tamaño** | **Standard_D4s_v3** |
    | **Tipo de autenticación** | **Contraseña** |
    | **Nombre de usuario** | **estudiante** |
    | **Contraseña** | cualquier contraseña compleja de su elección |
   
    > **Nota**: asegúrese de recordar la contraseña que especificó durante la implementación. Lo necesitará más adelante en este laboratorio.

    > **Nota**: Para buscar la imagen, haz clic en el vínculo **Ver todas las imágenes**, en la hoja **Seleccionar una imagen**, en el cuadro de texto de búsqueda, escribe **SUSE Enterprise Linux** y, en la lista de resultados, haz clic en **SUSE Enterprise Linux para SAP 15 SP5 - BYOS** y selecciona **Generación 2**.

1. En la pestaña **Disks** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Redes >** (deje todas las demás configuraciones con su valor predeterminado):

    | Configuración | Valor |
    |   --    |  --   |
    | **Tipo de disco del sistema operativo** | **SSD prémium (almacenamiento con redundancia local)**  |
    | **Administración de claves** | **Clave administrada por la plataforma** |

1. En la pestaña **Redes** de la hoja **Crear una máquina virtual**, en la sección **Interfaz de red**, directamente debajo del cuadro de texto **Red virtual**, selecciona **Crear nueva**. 
1. En el panel **Crear red virtual**, especifica la siguiente configuración y selecciona **Aceptar**:

    | Configuración | Value |
    |   --    |  --   |
    | **Nombre** | **az12001a-RG-vnet** |
    | **Espacio de direcciones** | **192.168.0.0/20** |
    | **Nombre de subred** | **subnet-0** |
    | **Intervalo de direcciones** | **192.168.0.0/24** |

1. De nuevo en la pestaña **Redes** de la hoja **Crear una máquina virtual**, especifica la siguiente configuración y selecciona **Siguiente: Administración >** (deja todas las demás opciones con su valor predeterminado):

    | Configuración | Valor |
    |   --    |  --   |
    | **Dirección IP pública** | **Ninguno** |
    | **Grupo de seguridad de red de NIC** | **Avanzado**  |
    | **Habilitación de la redes aceleradas** | enabled |
    | **Opciones de equilibrio de carga** | **None** |
    
    > **Nota**: esta implementación tiene reglas de NSG preconfiguradas.

1. En la pestaña **Administración** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Supervisión >** (deje todas las demás configuraciones con su valor predeterminado):
   
   | Configuración | Valor |
   |   --    |  --   |
   | **Habilitación del plan básico de forma gratuita** | deshabilitado |
   | **Habilitación de una identidad administrada asignada por el sistema** | deshabilitado |
   | **Habilitación del apagado automático** | deshabilitado  |

   > **Nota**: el **plan básico para la configuración gratuita** no está disponible si ya ha habilitado Microsoft Defender for Cloud en su suscripción.

1. En la pestaña **Supervisión** de la hoja **Crear una máquina virtual**, seleccione **Siguiente: Avanzado >** (deje todas las configuraciones con su valor predeterminado)

1. En la pestaña **Opciones avanzadas** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Revisar y crear** (deje todas las demás opciones con su valor predeterminado):

   | Configuración | Valor |
   |   --    |  --   |
   | **Grupo con ubicación por proximidad** | **az12001a-ppg** |

1. En la pestaña **Revisar y crear** de la hoja **Crear una máquina virtual**, seleccione **Crear**.

   > **Nota**: Espere a que se complete el aprovisionamiento. Esto debería tardar menos de 3 minutos.

1. En Azure Portal, use el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a la hoja **Máquinas virtuales** y, después, en la hoja **Máquinas virtuales**, seleccione **+ Crear** y, en el menú desplegable, seleccione **Máquina virtual** de Azure.

1. En la pestaña **Aspectos básicos** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Discos >** (deje todas las demás configuraciones con su valor predeterminado):
   
    | Configuración | Valor |
    |   --    |  --   |
    | **Suscripción** | el nombre de la suscripción de Azure |
    | **Grupo de recursos** | El nombre del grupo de recursos que has creado anteriormente en esta tarea |
    | **Nombre de la máquina virtual** | **az12001a-vm1** |
    | **Región** | El nombre de la región de Azure que has elegido al crear el grupo con ubicación por proximidad |
    | **Opciones de disponibilidad** | **El conjunto de disponibilidad** |
    | **El conjunto de disponibilidad** | **az12001a-avset** |
    | **tipo de seguridad** | **Estándar** |
    | **Imagen** | **SUSE Enterprise Linux para SAP 15 SP5 - BYOS - x64 Gen 2** |
    | **ejecución con Azure Spot Discount** | deshabilitado |
    | **Tamaño** | **Standard_D4s_v3** |
    | **Tipo de autenticación** | **Contraseña** |
    | **Nombre de usuario** | **estudiante** |
    | **Contraseña** | la misma contraseña que especificó durante la primera implementación |
   
    > **Nota**: Para buscar la imagen, haz clic en el vínculo **Ver todas las imágenes**, en la hoja **Seleccionar una imagen**, en el cuadro de texto de búsqueda, escribe **SUSE Enterprise Linux** y, en la lista de resultados, haz clic en **SUSE Enterprise Linux para SAP 15 SP5 - BYOS** y selecciona **Generación 2**.

1. En la pestaña **Disks** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Redes >** (deje todas las demás configuraciones con su valor predeterminado):

    | Configuración | Valor |
    |   --    |  --   |
    | **Tipo de disco del sistema operativo** | **SSD Premium**  |
    | **Administración de claves** | **Clave administrada por la plataforma** |

1. En la pestaña **Redes** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Administración >** (deje todas las demás configuraciones con su valor predeterminado):
    
    | Configuración | Value |
    |   --    |  --   |
    | **Red virtual** | **az12001a-RG-vnet** |
    | **Subred** | **subnet-0 (192.168.0.0/24)** |
    | **Dirección IP pública** | **Ninguno** |
    | **Grupo de seguridad de red de NIC** | **Avanzado**  |
    | **Habilitación de la redes aceleradas** | **Activado** |
    | **Opciones de equilibrio de carga** | **None** |

1. En la pestaña **Administración** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Supervisión >** (deje todas las demás configuraciones con su valor predeterminado):
    
   | Configuración | Valor |
   |   --    |  --   |
   | **Habilitación del plan básico de forma gratuita** | deshabilitado |
   | **Habilitación de una identidad administrada asignada por el sistema** | deshabilitado |
   | **Habilitación del apagado automático** | deshabilitado |

   > **Nota**: el **plan básico para la configuración gratuita** no está disponible si ya ha seleccionado el plan de Azure Security Center.

1.  En la pestaña **Supervisión** de la hoja **Crear una máquina virtual**, seleccione **Siguiente: Avanzado >** (deje todas las configuraciones con su valor predeterminado)

1.  En la pestaña **Avanzado** de la hoja **Crear una máquina virtual**, asegúrate de que está configurado el siguiente ajuste y selecciona **Revisar + crear** (deja todas las demás opciones con su valor predeterminado):
    
   | Configuración | Valor |
   |   --    |  --   |
   | **Grupo con ubicación por proximidad** | **az12001a-ppg** |

1.  En la pestaña **Revisar y crear** de la hoja **Crear una máquina virtual**, seleccione **Crear**.

   > **Nota**: Espere a que se complete el aprovisionamiento. Esto debería tardar menos de 3 minutos.


### Tarea 2: Creación y configuración de una instancia de discos de máquinas virtuales de Azure

1. En Azure Portal, inicie una sesión de Bash en Cloud Shell. 

   > **Nota**: si es la primera vez que inicia Cloud Shell en la suscripción actual de Azure, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `RESOURCE_GROUP_NAME` en el nombre del grupo de recursos que contiene los recursos que aprovisionó en la tarea anterior:

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear el primer conjunto de ocho discos administrados que asociará a la primera máquina virtual de Azure que implementó en la tarea anterior:

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear el segundo conjunto de ocho discos administrados que asociará a la segunda máquina virtual de Azure que implementó en la tarea anterior:

   ```cli
   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Cierre el panel de Cloud Shell.

1. En Azure Portal, vaya a la hoja de la primera máquina virtual de Azure que aprovisionó en la tarea anterior (**az12001a-vm0**).

1. En la hoja **az12001a-vm0**, vaya a la hoja **az12001a-vm0 \| Disks**.

1. En la hoja **az12001a-vm0 \| Disks**, seleccione **Conectar disco de datos existente** y conecte el disco de disk con la siguiente configuración a az12001a-vm0:
    
   | Configuración | Value |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nombre del disco** | **az12001a-vm0-DataDisk0** |
   | **Grupo de recursos** | El nombre del grupo de recursos que has usado anteriormente en esta tarea |
   | **ALMACENAMIENTO EN CACHÉ DE HOST** | **Solo lectura** |

2. Repita el paso anterior para conectar los siete discos restantes con el prefijo **az12001a-vm0-DataDisk** (para el total de ocho). Asigne el número LUN que coincida con el último carácter del nombre del disco. Establezca el ALMACENAMIENTO EN CACHÉ DE HOST del disco con LUN **1** en **Solo lectura** y, para todos los demás, establezca ALMACENAMIENTO EN CACHÉ DE HOST en **Ninguno**.

3. Guarde los cambios. 

4. En Azure Portal, vaya a la hoja de la segunda máquina virtual de Azure que aprovisionó en la tarea anterior (**az12001a-vm1**).

5. En la hoja **az12001a-vm1**, vaya a la hoja **az12001a-vm1 \| Disks**.

6. En la hoja **az12001a-vm1 \| Disks**, conecte discos de datos con la siguiente configuración a az12001a-vm1:
    
   | Configuración | Value |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nombre del disco** | **az12001a-vm1-DataDisk0** |
   | **Grupo de recursos** | El nombre del grupo de recursos que has usado anteriormente en esta tarea |
   | **ALMACENAMIENTO EN CACHÉ DE HOST** | **Solo lectura** |

7. Repita el paso anterior para conectar los siete discos restantes con el prefijo **az12001a-vm1-DataDisk** (para el total de ocho). Asigne el número LUN que coincida con el último carácter del nombre del disco. Establece el ALMACENAMIENTO EN CACHÉ DE HOST del disco con LUN **1** en **Solo lectura** y, para todos los demás, establece el ALMACENAMIENTO EN CACHÉ en **Ninguno**.

8. Guarde los cambios. 

#### Tarea 3: Aprovisionamiento de Azure Bastion 

> **Nota**: Azure Bastion permite la conexión a las máquinas virtuales de Azure (que implementó en la tarea anterior de este ejercicio) sin usar puntos de conexión públicos, al tiempo que proporciona protección contra vulnerabilidades de seguridad bruta que tienen como destino las credenciales de nivel de sistema operativo.

> **Nota**: Para usar Azure Bastion, asegúrese de que el explorador tenga habilitada la funcionalidad emergente.

1. En la ventana del explorador en que se muestra Azure Portal, abra otra pestaña y, en la pestaña del explorador, vaya a [**Azure Portal**](https://portal.azure.com).
1. En Azure Portal, seleccione el icono de la barra de herramientas inmediatamente a la derecha del cuadro de texto de búsqueda para abrir el panel de **Cloud Shell**.
1. En la sesión de PowerShell del panel de Cloud Shell, ejecute lo siguiente para agregar una subred denominada **AzureBastionSubnet** a la red virtual **az12001a-RG-vnet** que creamos anteriormente en este ejercicio:

   ```powershell
   $resourceGroupName = 'az12001a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az12001a-RG-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 192.168.15.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cierre el panel de Cloud Shell.
1. En Azure Portal, busque y seleccione **Bastions** y, en la hoja **Bastions**, seleccione **+ Crear**.
1. En la pestaña **Básico** de la hoja **Crear una instancia de Bastion**, especifique la siguiente configuración y seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**az12001a-RG**|
   |Nombre|**az12001a-bastion**|
   |Region|La misma región de Azure donde implementamos los recursos en la tarea anterior de este ejercicio|
   |Nivel|**Basic**|
   |Red virtual|**az12001a-RG-vnet**|
   |Subnet|**AzureBastionSubnet (192.168.15.0/24)**|
   |Dirección IP pública|**Crear nuevo**|
   |Nombre de la IP pública|**az12001a-RG-vnet-ip**|

1. En la pestaña **Revisar y crear** de la hoja **Crear una instancia de Bastion**, seleccione **Crear**:

   > **Nota**: Espere a que la implementación se complete antes de avanzar a la siguiente tarea de este ejercicio. La implementación puede tardar unos 5 minutos.

> **Result**: Después de completar este ejercicio, ha aprovisionado los recursos de proceso de Azure necesarios para admitir implementaciones de SAP HANA de alta disponibilidad.


## Ejercicio 2: Configurar el sistema operativo de máquinas virtuales de Azure que ejecutan Linux para admitir una implementación de SAP HANA de alta disponibilidad

Duración: 30 minutos

En este ejercicio, configurará el sistema operativo y el almacenamiento en máquinas virtuales de Azure que ejecuten SUSE Linux Enterprise Server para dar cabida a las instalaciones en clúster de SAP HANA.

### Tarea 1: Conexión a máquinas virtuales Linux de Azure

1. En el equipo de laboratorio, en Azure Portal, busque y seleccione **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, seleccione la entrada **az12001a-vm0**. Se abrirá la hoja **az12001a-vm0**.

1. En la hoja **az12001a-vm0**, seleccione **Conectar**, en el menú desplegable, seleccione **Conectar a través de Bastion**, en la pestaña **Bastion** de **az12001a-vm0**, deje el **tipo de autenticación** establecido en **Contraseña de máquina virtual**, proporcione las credenciales que establezca al implementar la máquina virtual **az12001a-vm0**, deje la casilla **Abrir en una nueva pestaña del explorador** habilitada y seleccione**Conectar**:

1. Repita los dos pasos anteriores para conectarse a través de Bastion a la máquina virtual de Azure **az12001a-vm1**.

### Tarea 2: Configuración del almacenamiento de máquinas virtuales de Azure que ejecutan Linux

1. En la sesión de Bastion a la máquina virtual de Azure **az12001a-vm0**, ejecute el siguiente comando para elevar privilegios: 

   ```sh
   sudo su -
   ```

1. Ejecute el siguiente comando para identificar la asignación entre los dispositivos recién conectados y sus números de LUN:
   
   ```sh
   lsscsi
   ```

1. Cree volúmenes físicos para 6 (de los 8) discos de datos mediante la ejecución de:
   
   ```sh
   pvcreate /dev/sdc
   pvcreate /dev/sdd
   pvcreate /dev/sde
   pvcreate /dev/sdf
   pvcreate /dev/sdg
   pvcreate /dev/sdh
   ```

1. Cree grupos de volúmenes mediante la ejecución de:
   
   ```sh
   vgcreate vg_hana_data /dev/sdc /dev/sdd
   vgcreate vg_hana_log /dev/sde /dev/sdf
   vgcreate vg_hana_backup /dev/sdg /dev/sdh
   ```

1. Cree volúmenes lógicos mediante la ejecución de:

   ```sh
   lvcreate -l 100%FREE -n hana_data vg_hana_data
   lvcreate -l 100%FREE -n hana_log vg_hana_log
   lvcreate -l 100%FREE -n hana_backup vg_hana_backup
   ```

   > **Nota**: estamos creando un único volumen lógico por cada grupo de volúmenes

1. Dé formato a los volúmenes lógicos mediante la ejecución de:

   ```sh
   mkfs.xfs /dev/vg_hana_data/hana_data -m crc=1
   mkfs.xfs /dev/vg_hana_log/hana_log -m crc=1
   mkfs.xfs /dev/vg_hana_backup/hana_backup -m crc=1
   ```

   > **Nota**: a partir de SUSE Linux Enterprise Server 12, tiene la opción de usar el nuevo formato en disco (v5) del sistema de archivos XFS, que ofrece sumas de comprobación automáticas de metadatos XFS, compatibilidad con tipos de archivo y un mayor límite en el número de listas de control de acceso por archivo. El nuevo formato se aplica automáticamente al usar YaST para crear los sistemas de archivos XFS. Para crear un sistema de archivos XFS en el formato anterior por motivos de compatibilidad, use el comando mkfs.xfs sin la opción `-m crc=1`. 

1. Cree particiones del disco **/dev/sdi** mediante la ejecución de:

   ```sh
   fdisk /dev/sdi
   ```

1. Cuando se le solicite, escriba, en secuencia, `n`, `p`, `1` (seguido de la tecla **Entrar** cada vez) presione la tecla **Entrar** dos veces y, a continuación, escriba `w` para completar la escritura.

1. Cree particiones del disco **/dev/sdj** mediante la ejecución de:

   ```sh
   fdisk /dev/sdj
   ```

1. Cuando se le solicite, escriba, en secuencia, `n`, `p`, `1` (seguido de la tecla **Entrar** cada vez) presione la tecla **Entrar** dos veces y, a continuación, escriba `w` para completar la escritura.

1. Dé formato a la partición recién creada ejecutando (escriba `y` y presione la tecla **Entrar** cuando se le pida confirmación):

   ```sh
   mkfs.xfs /dev/sdi -m crc=1 -f
   mkfs.xfs /dev/sdj -m crc=1 -f
   ```

1. Cree los directorios que servirán como puntos de montaje mediante la ejecución de:

   ```sh
   mkdir -p /hana/data
   mkdir -p /hana/log
   mkdir -p /hana/backup
   mkdir -p /hana/shared
   mkdir -p /usr/sap
   ```

1. Para mostrar los identificadores de los volúmenes lógicos, ejecute:

   ```sh
   blkid
   ```

   > **Nota**: identifique los valores **UUID** asociados a los grupos de volúmenes y particiones recién creados, incluidos **/dev/sdi** (que se usará para **/hana/shared**) y **dev/sdj** (que se usará para **/usr/sap**).


1. Abra **/etc/fstab** en el editor vi (puede usar cualquier otro editor) ejecutando:

   ```sh
   vi /etc/fstab
   ```

   > **Nota**: Si usas el editor vi, presiona la tecla **i** para introducir en el modo INSERT.

   ![Indicador del modo INSERT del editor vi](../media/az120-lab01-vieditor-insert.png)

1. En el editor, agregue las siguientes entradas a **/etc/fstab** (donde `\<UUID of /dev/vg\_hana\_data-hana\_data\>`, `\<UUID of /dev/vg\_hana\_log-hana\_log\>`, `\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`, `\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>` y `\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>`, representan los identificadores que identificó en el paso anterior):

   ```sh
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
   ```

1. Guarde los cambios y cierre el editor.

1. Monte los nuevos volúmenes mediante la ejecución de:

   ```sh
   mount -a
   ```

1. Compruebe que el montaje se realizó correctamente ejecutando:

   ```sh
   df -h
   ```

   ![salida df-h](../media/az120-lab01-df-output.png)
1. Sal del modo con privilegios ejecutando:

   ```sh
   exit
   ```

1. Cambia a la sesión de Bastion a **az12001a-vm1** y repite todos los pasos de estas tareas para configurar el almacenamiento en **az12001a-vm1**.


### Tarea 3: Habilitación del acceso de SSH sin contraseña entre nodos

1. En la sesión de Bastion a **az12001a-vm1**, genera un par de claves de SSH sin frase de contraseña mediante la ejecución de:

      ```sh
   ssh-keygen -trsa
   ```

1. Cuando se te solicite, presiona la tecla **Entrar** tres veces.

1. Copia la clave pública del par de claves recién generado en **az12001a-vm0** mediante la ejecución de:

      ```sh
   ssh-copy-id -i /home/student/.ssh/id_rsa.pub student@az12001a-vm0
   ```

1. Cuando se te pida que confirmes si deseas continuar con la conexión, escribe **sí** y presiona la tecla **Entrar**.

1. Cuando se te pida que te autentiques, escribe la contraseña que has establecido al aprovisionar **az12001a-vm0** anteriormente en este laboratorio.

1. Cambia a la sesión de Bastion a la máquina virtual de Azure **az12001a-vm0**.

1. En la sesión de Bastion a **az12001a-vm0**, genera un par de claves de SSH sin frase de contraseña mediante la ejecución de:

      ```sh
   ssh-keygen -trsa
   ```

1. Cuando se te solicite, presiona la tecla **Entrar** tres veces.

1. Copia la clave pública del par de claves recién generado en **az12001a-vm1** mediante la ejecución de:

      ```sh
   ssh-copy-id -i /home/student/.ssh/id_rsa.pub student@az12001a-vm1
   ```

1. Cuando se te pida que confirmes si deseas continuar con la conexión, escribe **sí** y presiona la tecla **Entrar**.

1. Cuando se te pida que te autentiques, escribe la contraseña que has establecido al aprovisionar **az12001a-vm1** anteriormente en este laboratorio.

1. Para comprobar que la configuración se ha realizado correctamente, en la sesión de Bastion a la máquina virtual de Azure **az12001a-vm0**, establece una sesión de SSH como **alumno** a **az12001a-vm1** mediante la ejecución de: 

   ```sh
   ssh student@az12001a-vm1
   ```

1. Asegúrese de que no se le solicite la contraseña.

1. Cierra la sesión de SSH de **az12001a-vm0** a **az12001a-vm1** mediante la ejecución de: 

   ```sh
   exit
   ```

1. Cambia a la sesión de Bastion a la máquina virtual de Azure **az12001a-vm1**.

1. En la sesión de Bastion a **az12001a-vm1**, establece una sesión SSH como **estudiante** a **az12001a-vm0** mediante la ejecución de: 

   ```sh
   ssh student@az12001a-vm0
   ```

1. Asegúrese de que no se le solicite la contraseña.

1. Cierra la sesión de SSH de **az12001a-vm1** a **az12001a-vm0** mediante la ejecución de: 

   ```sh
   exit
   ```

> **Result**: después de completar este ejercicio, ha configurado el sistema operativo de máquinas virtuales de Azure que ejecutan Linux para admitir una instalación de SAP HANA de alta disponibilidad


## Ejercicio 3: Aprovisionar los recursos de red de Azure necesarios para admitir implementaciones de SAP HANA de alta disponibilidad

Duración: 30 minutos

En este ejercicio, implementará Azure Load Balancer para dar cabida a las instalaciones en clúster de SAP HANA.


### Tarea 1: Configure máquinas virtuales de Azure para facilitar la configuración del equilibrio de carga.

1. En Azure Portal, vaya a la hoja de la máquina virtual de Azure **az12001a-vm0**.

1. En la hoja **az12001a-vm0**, vaya a la hoja **az12001a-vm0 \| Redes**. 

1. En la **hoja az12001a-vm0 \| Redes**, seleccione la entrada que representa la interfaz de red de az12001a-vm0. 

1. En la hoja de la interfaz de red de az12001a-vm0, vaya a su hoja configuraciones IP y, desde allí, muestre su hoja **ipconfig1**.

1. En la hoja **ipconfig1**, establezca la asignación de direcciones IP privadas en **Estática** y guarde el cambio.

1. En Azure Portal, vaya a la hoja de la máquina virtual de Azure **az12001a-vm1**.

1. En la hoja **az12001a-vm1**, vaya a la hoja **az12001a-vm1 \| Redes**. 

1. En la hoja **az12001a-vm1 \| Redes**, vaya a la interfaz de red de az12001a-vm1. 

1. En la hoja de la interfaz de red de az12001a-vm1, vaya a su hoja configuraciones IP y, desde allí, muestre su hoja **ipconfig1**.

1. En la hoja **ipconfig1**, establezca la asignación de direcciones IP privadas en **Estática** y guarde el cambio.


### Tarea 2: creación y configuración de equilibradores de carga de Azure que controlan el tráfico entrante

1. En Azure Portal, use el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal para buscar y navegar a la hoja **Equilibradores de carga** y, en la hoja **Equilibradores de carga**, seleccione **+ Crear**.

1. En la pestaña **Aspectos básicos** de la hoja **Crear equilibrador de carga**, especifique la siguiente configuración y seleccione **Revisar y crear** (deje los demás con sus valores predeterminados):
    
   | Configuración | Valor |
   |   --    |  --   |
   | **Suscripción** | el nombre de la suscripción de Azure |
   | **Grupo de recursos** | **az12001a-RG** |
   | **Nombre** | **az12001a-lb0** |
   | **Región** | la misma región de Azure en la que implementó máquinas virtuales de Azure en el primer ejercicio de este laboratorio |
   | **SKU** | **Estándar** |
   | **Tipo** | **Interno** |

1. Haga clic en **Siguiente: Configuración de IP de front-end**. En la pantalla **Configuración de IP de front-end**, haga clic en **Agregar una configuración de IP de front-end** y, a continuación, haga clic en **Agregar**.
    
   | Configuración | Value |
   |   --    |  --   |
   | **Nombre** | **frontend1** |
   | **Red virtual** | **az12001a-RG-vnet** |
   | **Subred** | **subnet-0** |
   | **Asignación de dirección IP** | **Estática** |
   | **Dirección IP** | **192.168.0.240** |
   | **Zona de disponibilidad** | **Con redundancia de zona** |

1. Seleccione **Revisar y crear** y, luego, **Crear**.

   > **Nota**: espere hasta que se aprovisione el equilibrador de carga. Debería tardar menos de un minuto. 

1. En Azure Portal, vaya a la hoja que muestra las propiedades del equilibrador de carga **az12001a-lb0** recién aprovisionado. 

1. En la hoja **az12001a-lb0**, seleccione **Grupos de back-end**, seleccione **+ Agregar** y, en **Agregar grupo de back-end**, especifique la siguiente configuración (deje el resto con sus valores predeterminados):
    
   | Configuración | Value |
   |   --    |  --   |
   | **Nombre** | **az12001a-lb0-bepool** |
   | **Red virtual** | **az12001a-RG-vnet** |
   | **Configuración del grupo de back-end** | **Dirección IP** |
   | **Dirección IP** | **192.168.0.4** Nombre del recurso: **az12001a-vm0** |
   | **Dirección IP** | **192.168.0.5** Nombre del recurso: **az12001a-vm1** |

1. En la hoja **az12001a-lb0**, seleccione **Sondeos de estado**, seleccione **+ Agregar** y, en la hoja **Agregar sondeo de estado**, especifique la siguiente configuración (deje el resto con sus valores predeterminados):
    
   | Configuración | Value |
   |   --    |  --   |
   | **Nombre** | **az12001a-lb0-hprobe** |
   | **Protocolo** | **TCP** |
   | **Puerto** | **62500** |
   | **Intervalo** | **5** *segundos* |

1. En la hoja **az12001a-lb0**, seleccione **Reglas de equilibrio de carga**, seleccione **+ Agregar** y, en la hoja **Agregar regla de equilibrio de carga**, especifique la siguiente configuración (deje el resto con sus valores predeterminados):
    
   | Configuración | Value |
   |   --    |  --   |
   | **Nombre** | **az12001a-lb0-lbruleAll** |
   | **Versión de IP** | **IPv4** |
   | **Frontend IP address** (Dirección IP de front-end) | **192.168.0.240 (LoadBalancerFrontEnd)** |
   | **Puertos de alta disponibilidad** | enabled |
   | **Grupo de back-end** | **az12001a-lb0-bepool (2 virtual machines)** |
   | **Sondeo de estado** | **az12001a-lb0-hprobe (TCP:62500)** |
   | **Persistencia de la sesión** | **None** |
   | **Tiempo de espera de inactividad (minutos)** | **4** |
   | **Restablecimiento de TCP** | deshabilitado |
   | **IP flotante (Direct Server Return)** | enabled |

### Tarea 3: creación y configuración de equilibradores de carga de Azure que controlan el tráfico saliente

1. En Azure Portal, inicie una sesión de Bash en Cloud Shell. 

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `RESOURCE_GROUP_NAME` en el nombre del grupo de recursos que contiene los recursos que aprovisionó en el primer ejercicio de este laboratorio:

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear la dirección IP pública que usará el segundo equilibrador de carga:

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   PIP_NAME='az12001a-lb1-pip'

   az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
   ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear el segundo equilibrador de carga:

   ```cli
   LB_NAME='az12001a-lb1'

   LB_BE_POOL_NAME='az12001a-lb1-bepool'

   LB_FE_IP_NAME='az12001a-lb1-fe'

   az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
   ```

1. En el panel de Cloud Shell, ejecute el siguiente comando para crear la regla de salida del segundo equilibrador de carga:

   ```cli
   LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

   az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
   ```

1. Cierre el panel de Cloud Shell.

1. En Azure Portal, vaya a la hoja que muestra las propiedades del Azure Load Balancer **az12001a-lb1** recién creado.

1. En la hoja **az12001a-cl-lb1**, haga clic en **Grupos de back-end**.

1. En la hoja **az12001a-lb1 \| Grupos de back-end**, haga clic en **az12001a-lb1-bepool**.

1. En la hoja **az12001a-lb1-bepool**, especifique la siguiente configuración y haga clic en **Guardar**:
   
   | Configuración | Value |
   |   --    |  --   |
   | **Red virtual** | **az12001a-rg-vnet (2 VM)** |
   | **Máquina virtual** | **az12001a-vm0** Configuración de IP: **ipconfig1 (192.168.0.4)** |
   | **Máquina virtual** | **az12001a-vm1**  Configuración de IP: **ipconfig1 (192.168.0.5)** |

> **Result**: Después de completar este ejercicio, ha aprovisionado los recursos de red de Azure necesarios para admitir implementaciones de SAP HANA de alta disponibilidad


## Ejercicio 4: Eliminación de recursos de laboratorio

Duración: 10 minutos

En este ejercicio, quitará los recursos aprovisionados en este laboratorio.

#### Tarea 1: Enumeración de grupos de recursos que se van a eliminar

1. En la parte superior del portal, haga clic en el icono de **Cloud Shell** para abrir el panel de Cloud Shell y elija Bash como shell.

1. En el panel de Cloud Shell, ejecute el siguiente comando para establecer el valor de la variable `RESOURCE_GROUP_PREFIX` en el prefijo del nombre del grupo de recursos que contiene los recursos que aprovisionó en este laboratorio:

   ```cli
   RESOURCE_GROUP_PREFIX='az12001a-'
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
