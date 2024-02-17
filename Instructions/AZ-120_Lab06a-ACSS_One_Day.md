---
lab:
  title: '06a: introducción a los requisitos previos para la implementación de Azure Center for SAP solutions (ACSS)'
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Módulo AZ 1006: Diseño e implementación de una infraestructura para admitir cargas de trabajo de SAP en Azure
# Laboratorio del curso AZ-1006 de un día: Introducción a los requisitos previos para la implementación de cargas de trabajo de SAP con Azure Center for SAP solutions (ACSS)

Tiempo estimado: 100 minutos

Todas las tareas de este laboratorio de cursos de AZ-1006 de un día se realizan desde Azure Portal

## Objetivos

Después de completar este laboratorio, podrá hacer lo siguiente:

- Implementación de los requisitos previos para implementar cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions

## Instrucciones

### Ejercicio 1: Implementación de los requisitos previos para implementar cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions

Duración: 60 minutos

En este ejercicio, revisará e implementará los requisitos previos para implementar cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions. Esto incluye las siguientes actividades:

- Creación de una identidad administrada asignada por el usuario de Microsoft Entra para que la use Azure Center for SAP solutions para el acceso a Azure Storage durante su implementación.
- Creación de la red virtual de Azure que hospeda todas las máquinas virtuales de Azure incluidas en la implementación.
- Creación de un recurso de Azure Bastion para proteger la conectividad con máquinas virtuales de Azure desde Internet.
- Creación de una cuenta de uso general de Azure Storage v2 asociada a Azure Center for SAP solutions para la implementación
- Concesión de la identidad administrada asignada por el usuario de Microsoft Entra que se usa para realizar el acceso de implementación a la suscripción de Azure y la cuenta de uso general v2 de Azure Storage
- Creación de una cuenta de recursos compartidos de archivos Premium de Azure que se usa para implementar el directorio de transporte de SAP
- Creación y configuración de un grupo de seguridad de red (NSG) usado para restringir el acceso saliente desde subredes de la red virtual que hospeda la implementación.
- Creación de una máquina virtual (VM) de Azure que se usará para la instalación de software de SAP como parte de una implementación de Azure Center for SAP solutions.
- Conexión a la máquina virtual de Azure mediante Azure Bastion y configuración para la instalación de software de SAP.
- Eliminación de todos los recursos de Azure aprovisionados en este laboratorio.

Estas actividades corresponden a las siguientes tareas de este ejercicio:

- Tarea 1: Creación de una identidad administrada asignada por el usuario de Microsoft Entra
- Tarea 2: Creación de la red virtual de Azure
- Tarea 3: Creación de un recurso de Azure Bastion
- Tarea 4: Creación de una cuenta de uso general v2 de Azure Storage
- Tarea 5: configuración de la autorización de la identidad administrada asignada por el usuario de Microsoft Entra
- Tarea 6: Creación de una cuenta de recursos compartidos de archivos Premium de Azure
- Tarea 7: Creación y configuración de un grupo de seguridad de red
- Tarea 8: Cree una máquina virtual de Azure.
- Tarea 9: Configurar la máquina virtual de Azure
- Tarea 10: Eliminación de recursos de Azure

#### Tarea 1: Creación de una identidad administrada asignada por el usuario de Microsoft Entra

En esta tarea, creará una identidad administrada asignada por el usuario de Microsoft Entra que usará Azure Center for SAP solutions para el acceso a Azure Storage durante su implementación.

1. En el equipo de laboratorio, inicie un explorador web, vaya a Azure Portal en `https://portal.azure.com`, y autentíquese mediante una cuenta Microsoft o una cuenta de Microsoft Entra ID con el rol Propietario en la suscripción de Azure que use en este laboratorio.
1. En la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Identidades administradas**.
1. En la página **Identidades administradas**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear identidad administrada asignada por el usuario**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que usa en este laboratorio|
`   |Resource group| Elija o cree una nueva **acss-infra-RG**|
   |Region|el nombre de la región de Azure, que se usa para la implementación de ACSS|
   |Nombre|**acss-infra-MI**|

1. En la pestaña **Revisar**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: no espere a que se complete el proceso de aprovisionamiento, sino que avance a la siguiente tarea. El aprovisionamiento debe tardar unos segundos.

   >**Nota**: en una de las próximas tareas, autorizará el acceso de la identidad administrada a la cuenta de almacenamiento que hospeda los medios de instalación de SAP para dar cabida a la instalación de software de SAP mediante Azure Center for SAP solutions.

#### Tarea 2: Crear la red virtual

En esta tarea, creará la red virtual de Azure que hospeda todas las máquinas virtuales de Azure incluidas en la implementación. Además, dentro de la red virtual, se crean las siguientes subredes:

- AzureFirewallSubnet: diseñada para la implementación de Azure Firewall
- AzureBastionSubnet: diseñada para la implementación de Azure Bastion
- dmz: diseñada para la implementación de la máquina virtual de Azure que se usa para implementar software de SAP
- aplicación: diseñada para hospedar la aplicación SAP y los servidores de instancias de SAP Central Services
- db: diseñada para hospedar el nivel de base de datos de SAP

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Redes virtuales**. 
1. En la página **Redes virtuales**, seleccione **y Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear red virtual**, especifique la siguiente configuración y seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre de la red virtual|**acss-infra-VNET**|
   |Region|el nombre de la misma región de Azure que usó en la tarea anterior de este ejercicio|

1. En la pestaña **Seguridad**, acepte la configuración predeterminada y seleccione **Siguiente**.

   >**Nota**: en este momento, puede aprovisionar tanto Azure Bastion como Azure Firewall, pero los aprovisionará por separado una vez creada la red virtual.

1. En la pestaña **Direcciones IP**, especifique la siguiente configuración de subred y, a continuación, seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Espacio de direcciones IP|**10.0.0.0/16 (65 536 direcciones)**|

1. En la lista de subredes, seleccione el icono de papelera para eliminar la subred **predeterminada**.
1. Seleccione **+ Agregar una subred**.
1. En el panel **Agregar una subred**, especifique la siguiente configuración y, a continuación, seleccione **Agregar** (deje los demás con sus valores predeterminados):

   |Configuración|Valor|
   |---|---|
   |Propósito de la subred|**Azure Firewall**|
   |Dirección inicial|**10.0.0.0**|

   >**Nota**: esto asignará automáticamente a la subred el nombre **AzureFirewallSubnet** y establecerá su tamaño en **/26 (64 direcciones).**

1. Seleccione **+ Agregar una subred**.
1. En el panel **Agregar una subred**, especifique la siguiente configuración y, a continuación, seleccione **Agregar** (deje los demás con sus valores predeterminados):

   |Configuración|Value|
   |---|---|
   |Nombre|**dmz**|
   |Dirección inicial|**10.0.0.128**|
   |Size|**/26 (64 direcciones)**|

   >**Nota**: esta subred se usará para hospedar la máquina virtual de Azure que usará para instalar el software de SAP mediante el Azure Center for SAP solutions.

1. Seleccione **+ Agregar una subred**.
1. En el panel **Agregar una subred**, especifique la siguiente configuración y, a continuación, seleccione **Agregar** (deje los demás con sus valores predeterminados):

   |Configuración|Valor|
   |---|---|
   |Propósito de la subred|**Azure Bastion**|
   |Dirección inicial|**10.0.1.0**|
   |Size|**/24 (256 direcciones)**|

   >**Nota**: esto asignará automáticamente a la subred el nombre **AzureBastionSubnet**.

1. Seleccione **+ Agregar una subred**.
1. En el panel **Agregar una subred**, especifique la siguiente configuración y, a continuación, seleccione **Agregar** (deje los demás con sus valores predeterminados):

   |Configuración|Value|
   |---|---|
   |Nombre|**app**|
   |Dirección inicial|**10.0.2.0**|
   |Size|**/24 (256 direcciones)**|

1. Seleccione **+ Agregar una subred**.
1. En el panel **Agregar una subred**, especifique la siguiente configuración y, a continuación, seleccione **Agregar** (deje los demás con sus valores predeterminados):

   |Configuración|Value|
   |---|---|
   |Nombre|**db**|
   |Dirección inicial|**10.0.3.0**|
   |Size|**/24 (256 direcciones)**|

1. En la pestaña **Direcciones IP**, seleccione **Revisar y crear**:
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y, a continuación, seleccione **Crear**.

   >**Nota**: no espere a que se complete el proceso de aprovisionamiento, sino que avance a la siguiente tarea. El aprovisionamiento debe tardar unos segundos.

#### Tarea 3: Creación de un recurso de Azure Bastion

En esta tarea, creará un recurso de Azure Bastion para proteger la conectividad a las máquinas virtuales de Azure desde Internet.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Bastions**. 
1. En la página **Bastions**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Bastions**, especifique la siguiente configuración y seleccione **Siguiente: Avanzado >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre|**acss-infra-BASTION**|
   |Region|el nombre de la misma región de Azure que usó anteriormente en este ejercicio|
   |Nivel|**Basic**|
   |Recuento de instancias|**2**|
   |Red virtual|**acss-infra-VNET**|
   |Subnet|**AzureBastionSubnet (10.0.1.0/24)**|
   |Dirección IP pública|**Crear nuevo**|
   |Nombre de la dirección IP pública|**acss-bastion-PIP**|

1. En la pestaña **Avanzado**, revise las opciones disponibles sin realizar ningún cambio y, a continuación, seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y, a continuación, seleccione **Crear**.

   >**Nota**: no espere a que se complete el aprovisionamiento, sino que continúe con la siguiente tarea. El aprovisionamiento puede tardar unos cinco minutos.

#### Tarea 4: Creación de una cuenta de uso general v2 de Azure Storage

En esta tarea, creará una cuenta de uso general de Azure Storage v2 asociada a Azure Center for SAP solutions para la implementación. Esta cuenta de almacenamiento se usa para hospedar los medios de instalación de SAP para dar cabida a la instalación de software de SAP a través de Azure Center for SAP solutions.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Cuentas de almacenamiento**.
1. En la página **Cuentas de almacenamiento**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear una cuenta de almacenamiento**, especifique la siguiente configuración y seleccione **Siguiente: Opciones avanzadas >**.

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre de la cuenta de almacenamiento|Cualquier nombre globalmente único con una longitud de 3 a 24 caracteres, que consta de letras y dígitos|
   |Region|el nombre de la misma región de Azure que usó anteriormente en este ejercicio|
   |Rendimiento|**Estándar**|
   |Redundancia|**Almacenamiento con redundancia geográfica (GRS)**|
   |Habilite el acceso de lectura a los datos en el caso de que la región esté disponible|Deshabilitado|

1. En la pestaña **Avanzado**, revise las opciones disponibles, acepte los valores predeterminados y seleccione **Siguiente: Redes >**.
1. En la pestaña **Redes**, realice las siguientes acciones y, a continuación, seleccione **Revisar**.

   1. Seleccione **Habilitar el acceso público desde redes virtuales y direcciones IP seleccionadas**.
   1. En la sección **Redes virtuales**, asegúrese de que la lista desplegable **Suscripción de red virtual** muestra el nombre de la suscripción de Azure que usa en este laboratorio.
   1. En la sección **Redes virtuales**, en la **lista desplegable **Red virtual, seleccione **acss-infra-VNET**.
   1. En la lista desplegable **Subredes**, seleccione las subredes**aplicación**, **db**y **dmz**.

1. En la pestaña **Revisar**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: Espere a que se complete el proceso de aprovisionamiento. El aprovisionamiento debe tardar menos de 1 minuto.

1. En la página **Se completó la implementación**, haga clic en **Ir al recurso**.
1. En la página cuenta de almacenamiento, en el menú de navegación vertical del lado izquierdo, en la sección **Almacenamiento de datos**, seleccione **Contenedores**.
1. Seleccione **+ Contenedor**.
1. En el panel **Nuevo contenedor**, en el cuadro de texto **Nombre**, escriba **sapbits** y seleccione **Crear**.

   >**Nota**: el contenedor **sapbits** hospedará los medios de instalación de SAP.

#### Tarea 5: configuración de la autorización de la identidad administrada asignada por el usuario de Microsoft Entra

En esta tarea, usará una asignación de roles de control de acceso basado en rol (RBAC) de Azure para conceder la identidad administrada asignada por el usuario de Microsoft Entra. La identidad administrada se usa para realizar el acceso de implementación a la suscripción de Azure y la cuenta de uso general de Azure Storage v2 creada en la tarea anterior.

1. En Azure Portal, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Identidades administradas**.
1. En la página Identidades administradas y, seleccione la entrada **acss-infra-MI**.
1. En la página **acss-infra-MI**, en el menú de navegación vertical del lado izquierdo, seleccione **Asignaciones de roles de Azure**.
1. En la página **Asignaciones de roles de Azure**, seleccione **+ Agregar asignación de roles (versión preliminar)**.
1. En el panel **+ Agregar asignación de roles (versión preliminar)**, especifique la siguiente configuración y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Ámbito|**Suscripción**|
   |Subscription|Nombre de la suscripción de Azure que usa en este laboratorio|
   |Role|**Rol de servicio de Azure Center for SAP solutions**|

1. De nuevo en la página **Asignaciones de roles de Azure**, seleccione **+ Agregar asignación de roles (versión preliminar)**.
1. En el panel **+ Agregar asignación de roles (versión preliminar)**, especifique la siguiente configuración y seleccione **Guardar**:

   |Configuración|Valor|
   |---|---|
   |Ámbito|**Storage**|
   |Subscription|Nombre de la suscripción de Azure que usa en este laboratorio|
   |Resource|Nombre de la cuenta de Azure Storage que creó en la tarea anterior|
   |Role|**Lector y acceso a los datos**|

#### Tarea 6: Creación de una cuenta de recursos compartidos de archivos Premium de Azure

En esta tarea, creará una cuenta de recursos compartidos de archivos Premium de Azure usada para implementar el directorio de transporte de SAP.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Cuentas de almacenamiento**.
1. En la página **Cuentas de almacenamiento**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear una cuenta de almacenamiento**, especifique la siguiente configuración y seleccione **Siguiente: Opciones avanzadas >**.

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre de la cuenta de almacenamiento|Cualquier nombre globalmente único con una longitud de 3 a 24 caracteres, que consta de letras y dígitos|
   |Region|el nombre de la misma región de Azure que usó anteriormente en este ejercicio|
   |Rendimiento|**Premium**|
   |Tipo de cuenta premium|**Recursos compartidos de archivos**|
   |Redundancia|**Almacenamiento con redundancia de zona (ZRS)**|

1. En la **pestaña Opciones avanzadas**, deshabilite la **opción Requerir transferencia segura para las operaciones** de la API REST y seleccione **Siguiente: Redes >**.

   >**Nota**: El protocolo NFS no admite el cifrado y se basa en la seguridad de nivel de red en su lugar. Esta configuración debe deshabilitarse para que NFS funcione.

1. En la pestaña **Redes**, realice las siguientes acciones y, a continuación, seleccione **Revisar**.

   1. Seleccione **Habilitar el acceso público desde redes virtuales y direcciones IP seleccionadas**.
   1. En la sección **Redes virtuales**, asegúrese de que la lista desplegable **Suscripción de red virtual** muestra el nombre de la suscripción de Azure que usa en este laboratorio.
   1. En la sección **Redes virtuales**, en la **lista desplegable **Red virtual, seleccione **acss-infra-VNET**.
   1. En la lista desplegable **Subredes**, seleccione las subredes**aplicación**, **db**y **dmz**.

   >**Nota**: en general, evite permitir el acceso a los recursos internos desde subredes perimetrales. En este caso, la única razón para hacerlo es permitir validar este acceso más adelante en este laboratorio.

1. En la pestaña **Revisar**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: Espere a que se complete el proceso de aprovisionamiento. El aprovisionamiento debe tardar menos de 1 minuto.

1. En la página **Se completó la implementación**, haga clic en **Ir al recurso**.
1. En la página de cuenta de almacenamiento, en el menú de navegación vertical situado en el lado izquierdo, en la sección **Almacenamiento de datos**, seleccione **Recursos compartidos de archivos** y, después, **+ Recurso compartido de archivos**.
1. En la pestaña **Aspectos básicos** de la página **Nuevo recurso compartido**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

   |Configuración|Value|
   |---|---|
   |Nombre|**trans**|
   |Capacidad aprovisionada|**128**|
   |Protocolo|**NFS**|
   |Squash de raíz|*Sin restringir raíz**|

1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y, a continuación, seleccione **Crear**.

   >**Nota**: Espere a que se complete el aprovisionamiento del recurso compartido de archivos. El aprovisionamiento debe tardar unos segundos.

1. En la página **Conexión a este recurso compartido NFS desde Linux**, en la lista desplegable **Seleccionar la distribución de Linux**, seleccione **SUSE** en la lista desplegable de distribución de Linux y revise los comandos de ejemplo para montar este recurso compartido NFS.

#### Tarea 7: Creación y configuración de un grupo de seguridad de red

En esta tarea, creará y configurará un grupo de seguridad de red (NSG) usado para restringir el acceso saliente desde subredes de la red virtual que hospeda la implementación. Para ello, puede bloquear la conectividad a Internet, pero permitir explícitamente las conexiones a los siguientes servicios:

- Puntos de conexión de infraestructura de actualización de SUSE o Red Hat
- Azure Storage
- Azure Key Vault
- Microsoft Entra ID
- Azure Resource Manager

>**Nota**: en general, debe considerar la posibilidad de usar Azure Firewall en lugar de grupos de seguridad de red para proteger la conectividad de red para la implementación de SAP. En este laboratorio se tratan ambas opciones.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Grupo de seguridad de red**.
1. En la página **Grupos de seguridad de red**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear grupo de seguridad de red**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre|**acss-infra-NSG**|
   |Region|el nombre de la misma región de Azure que usó anteriormente en este ejercicio|

1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: Espere a que se complete el proceso de aprovisionamiento. El aprovisionamiento debe tardar menos de 1 minuto.

1. En la página **Se completó la implementación**, haga clic en **Ir al recurso**.

   >**Nota**: de forma predeterminada, las reglas integradas de los grupos de seguridad de red permiten todo el tráfico saliente, todo el tráfico dentro de la misma red virtual, así como todo el tráfico entre redes virtuales emparejadas. Desde el punto de vista de la seguridad, debe considerar la posibilidad de restringir este comportamiento predeterminado. La configuración propuesta restringe la conectividad saliente a Internet y Azure. También puede usar reglas de NSG para restringir la conectividad dentro de una red virtual.

1. En la página **acss-infra-NSG**, menú de navegación vertical del lado izquierdo, en la sección **Configuración**, seleccione **Reglas de seguridad de salida**.
1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   >**Nota**: se debe agregar la siguiente regla para permitir explícitamente la conectividad a los puntos de conexión de infraestructura de actualización de Red Hat.

   >**Nota**: para identificar las direcciones IP que se van a usar para RHEL, consulte [Preparación de la red para la implementación de la infraestructura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destino|**Direcciones IP**|
   |Intervalos de direcciones IP de destino y CIDR|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|
   |Service|**Personalizada**|
   |Intervalos de puertos de destino|*|
   |Protocolo|**Cualquiera**|
   |Acción|**Permitir**|
   |Priority|**300**|
   |Nombre|**AllowAnyRHELOutbound**|
   |Descripción|**Permitir la conectividad saliente a los puntos de conexión de la infraestructura de actualización de RHEL**|

1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   >**Nota**: se debe agregar la siguiente regla para permitir explícitamente la conectividad a los puntos de conexión de infraestructura de actualización de SUSE.

   >**Nota**: para identificar las direcciones IP que se van a usar para SUSE, consulte [Preparación de la red para la implementación de la infraestructura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destino|**Direcciones IP**|
   |Intervalos de direcciones IP de destino y CIDR|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|
   |Service|**Personalizada**|
   |Intervalos de puertos de destino|*|
   |Protocolo|**Cualquiera**|
   |Acción|**Permitir**|
   |Prioridad|**305**|
   |Nombre|**AllowAnySUSEOutbound**|
   |Descripción|**Permitir la conectividad saliente a los puntos de conexión de la infraestructura de actualización de SUSE**|

   >**Nota**: se debe agregar la siguiente regla para permitir explícitamente la conectividad a Azure Storage.

1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destination|**Etiqueta de servicio**|
   |Etiqueta de servicio de destino|**Storage**|
   |Service|**Personalizada**|
   |Intervalos de puertos de destino|*|
   |Protocolo|**Cualquiera**|
   |Acción|**Permitir**|
   |Prioridad|**400**|
   |Nombre|**AllowAnyCustomStorageOutbound**|
   |Descripción|**Permitir la conectividad saliente a Azure Storage**|

   >**Nota**: puede reemplazar la etiqueta de servicio de **Storage** por una específica de la región, como **Storage.EastUS**.

   >**Nota**: se debe agregar la siguiente regla para permitir explícitamente la conectividad a Azure Key Vault.

1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destination|**Etiqueta de servicio**|
   |Etiqueta de servicio de destino|**AzureKeyVault**|
   |Service|**Personalizada**|
   |Intervalos de puertos de destino|*|
   |Protocolo|**Cualquiera**|
   |Acción|**Permitir**|
   |Prioridad|**500**|
   |Nombre|**AllowAnyCustomKeyVaultOutbound**|
   |Descripción|**Permitir la conectividad saliente a Azure Key Vault**|

   >**Nota**: se debe agregar la siguiente regla para permitir explícitamente la conectividad a Microsoft Entra ID.

1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destination|**Etiqueta de servicio**|
   |Etiqueta de servicio de destino|**AzureActiveDirectory**|
   |Service|**Personalizada**|
   |Intervalos de puertos de destino|*|
   |Protocolo|**Cualquiera**|
   |Acción|**Permitir**|
   |Prioridad|**600**|
   |Nombre|**AllowAnyCustomEntraIDOutbound**|
   |Descripción|**Permitir la conectividad saliente a Microsoft Entra ID**|

   >**Nota**: se debe agregar la siguiente regla para permitir explícitamente la conectividad a Azure Resource Manager.

1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destination|**Etiqueta de servicio**|
   |Etiqueta de servicio de destino|**AzureResourceManager**|
   |Service|**Personalizada**|
   |Intervalos de puertos de destino|*|
   |Protocolo|**Cualquiera**|
   |Acción|**Permitir**|
   |Prioridad|**700**|
   |Nombre|**AllowAnyCustomARMOutbound**|
   |Descripción|**Permitir la conectividad saliente a Azure Resource Manager**|

   >**Nota**: la última regla debe agregarse para bloquear explícitamente la conectividad a Internet.

1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destination|**Etiqueta de servicio**|
   |Etiqueta de servicio de destino|**Internet**|
   |Service|**Personalizada**|
   |Intervalos de puertos de destino|*|
   |Protocolo|**Cualquiera**|
   |Acción|**Deny**|
   |Prioridad|**1000**|
   |Nombre|**DenyAnyCustomInternetOutbound**|
   |Descripción|**Denegar la conectividad saliente a Internet**|

   >**Nota**: por último, debe asignar el NSG a las subredes pertinentes de la red virtual que hospedarán la implementación de SAP.

1. En la página **Agregar regla de seguridad de salida**, menú de navegación vertical del lado izquierdo, en la sección **Configuración**, seleccione **Subredes**.
1. En la página **acss-infra-NSG \| Subredes**, seleccione **+ Asociar**.
1. En el panel **Asociar subred**, en la lista desplegable **Red virtual**, seleccione **acss-intra-VNET (acss-infra-RG)**, en la lista desplegable **Subred**, seleccione **aplicación** y, a continuación, seleccione **Aceptar**.
1. En el panel **Asociar subred**, en la lista desplegable **Red virtual**, seleccione **acss-intra-VNET (acss-infra-RG)**, en la lista desplegable **Subred**, seleccione **base de datos** y, a continuación, seleccione **Aceptar**.

#### Tarea 8: Cree una máquina virtual de Azure.

En esta tarea, creará una máquina virtual (VM) de Azure usada para la instalación de software de SAP como parte de una implementación de Azure Center for SAP solutions.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Máquinas virtuales**.
1. En la página **Máquinas virtuales**, seleccione **+ Crear** y, en el menú desplegable, seleccione **Máquina virtual de Azure**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear una máquina virtual**, especifique la siguiente configuración y seleccione **Siguiente: Discos >** (deje todas las demás configuraciones con su valor predeterminado):

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre de la máquina virtual|**acss-infra-vm0**|
   |Region|el nombre de la misma región de Azure que usó anteriormente en este ejercicio|
   |Opciones de disponibilidad|**No se requiere redundancia de la infraestructura**|
   |Tipo de seguridad|**Máquina virtual de inicio seguro**|
   |Imagen|**Ubuntu Server 20.04 LTS: x64 Gen2**|
   |Arquitectura VM|**x64**|
   |Ejecución con descuento de acceso puntual de Azure|deshabilitado|
   |Size|**Standard_B2ms**|
   |Tipo de autenticación|**Contraseña**|
   |Nombre de usuario|cualquier nombre de usuario válido|
   |Contraseña|cualquier contraseña compleja de su elección|
   |Puertos de entrada públicos|**Ninguno**|
   
    > **Nota**: Asegúrese de recordar el nombre de usuario y la contraseña que especificó. Lo necesitará más adelante en este laboratorio.

1. En la pestaña **Discos**, acepte los valores predeterminados y seleccione **Siguiente: Redes >**.
1. En la pestaña **Redes**, especifique la siguiente configuración y seleccione **Siguiente: Administración >** (deje todas las demás configuraciones con su valor predeterminado):

   |Configuración|Value |
   |---|---|
   |Red virtual|**acss-infra-VNET**|
   |Subnet|**dmz**|
   |Dirección IP pública|**Ninguno**|
   |Grupo de seguridad de red de NIC|**Ninguno**|
   |Eliminar NIC al eliminar la VM|enabled|
   |Opciones de equilibrio de carga|**Ninguno**|

1. En la pestaña **Administración** , deje toda la configuración con su valor predeterminado y seleccione **Siguiente: Supervisión >**
1. En la pestaña **Supervisión**, establezca **Diagnósticos de arranque** en **Deshabilitar** y seleccione **Siguiente: Avanzado >** (deje todas las demás configuraciones con su valor predeterminado)
1. En la **Avanzado**, seleccione **Revisar y crear** (deje toda la configuración con su valor predeterminado).
1. En la pestaña **Revisar y crear** del menú **Crear una máquina virtual**, seleccione **Crear**.

   > **Nota**: Espere a que se complete el aprovisionamiento. El aprovisionamiento puede tardar unos tres minutos.

#### Tarea 9: Configuración de la máquina virtual de Azure

En esta tarea, se conectará a la máquina virtual de Azure mediante Azure Bastion y la configurará para la instalación de software de SAP. 

> **Nota**: antes de iniciar esta tarea, asegúrese de que se ha completado el aprovisionamiento de Azure Bastion.

> **Nota**: asegúrese de que el explorador web no bloquee las ventanas emergentes y, si es así, deshabilite la funcionalidad de bloqueador de elementos emergentes.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Máquinas virtuales**. 
1. En la página **Máquinas virtuales**, seleccione la entrada **acss-infra-vm0**.
1. En la página **acss-infra-vm0**, en la barra de herramientas, seleccione **Conectar** y, en el menú desplegable, seleccione **Conectar a través de Bastion**.
1. En la página **acss-infra-vm0 \| Bastion**, asegúrese de que el **tipo de autenticación** esté establecido en **Contraseña de máquina virtual**, en los cuadros de texto **Nombre de usuario** y **Contraseña**, escriba el nombre de usuario y la contraseña que establezca al aprovisionar la máquina virtual de Azure, asegúrese de que la casilla **Abrir en la nueva pestaña** del explorador esté habilitada y, a continuación, seleccione **Conectar**.

   > **Nota**: debe abrir otra pestaña de ventana del explorador web que muestra la sesión de shell que se ejecuta en la máquina virtual de Azure.

   > **Nota**: para preparar el servidor Ubuntu para la carga de medios de instalación de SAP, instalará la CLI de Azure.

1. En la pestaña del explorador recién abierto, en la sesión de shell, ejecute el siguiente comando para instalar la CLI de Azure:

   ```bash
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

1. En la sesión de shell, ejecute el siguiente comando para instalar PIP3:

   ```bash
   sudo apt install python3-pip
   ```

1. En la sesión de shell, ejecute el siguiente comando para instalar Ansible 2.13.19:

   ```bash
   sudo pip3 install ansible-core==2.13.9
   ```

1. En la sesión de shell, ejecute el siguiente comando para instalar módulos de la colección galaxy de Ansible:

   ```bash
   sudo ansible-galaxy collection install ansible.netcommon:==5.0.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.posix:==1.5.1 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.utils:==2.9.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.windows:==1.13.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install community.general:==6.4.0 -p /opt/ansible/collections
   ```

1. En la sesión de shell, ejecute el siguiente comando para clonar el repositorio de ejemplos de automatización de SAP desde GitHub:

   ```bash
   git clone https://github.com/Azure/SAP-automation-samples.git
   ```

1. En la sesión de shell, ejecute el siguiente comando para clonar el repositorio de automatización de SAP desde GitHub:

   ```bash
   git clone https://github.com/Azure/sap-automation.git
   ```

1. En la sesión de shell, ejecute el siguiente comando para finalizar la sesión:

   ```bash
   logout
   ```

1. Cuando se le pida, seleccione **Ejecutar**.

#### Tarea 10: Eliminación de recursos de Azure

En esta tarea, quitará todos los recursos de Azure aprovisionados en este laboratorio.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, seleccione el icono de **Cloud Shell** para abrir el panel de Cloud Shell. Si es necesario, seleccione **Bash** para iniciar una sesión de shell de Bash. 

   > **Nota**: Si es la primera vez que inicia Cloud Shell en la suscripción de Azure que va a usar en este laboratorio, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En el panel de Cloud Shell, ejecute el siguiente comando para eliminar el grupo de recursos **acss-infra-RG** y todos sus recursos.

   ```cli
   az group delete --name 'acss-infra-RG' --no-wait --yes
   ```

   > **Nota**: el comando se ejecuta de forma asincrónica (según lo determinado por el `--nowait` parámetro ), por lo que, mientras que el símbolo del sistema del shell aparecerá inmediatamente después de invocarlo, unos minutos pasarán antes de que el grupo de recursos y sus recursos se quiten realmente.

1. Cierre el panel de Cloud Shell.
