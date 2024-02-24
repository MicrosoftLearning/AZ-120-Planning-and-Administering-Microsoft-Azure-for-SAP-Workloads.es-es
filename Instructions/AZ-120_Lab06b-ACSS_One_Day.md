---
lab:
  title: '06b: información general sobre la implementación y el mantenimiento del Azure Center for SAP solutions (ACSS)'
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Módulo AZ 1006: Diseño e implementación de una infraestructura para admitir cargas de trabajo de SAP en Azure
# Laboratorio del curso AZ-1006 de un día: Información general sobre la implementación y el mantenimiento del Azure Center for SAP solutions (ACSS)

>**Importante**: Actualmente **no se admite este laboratorio** (febrero de 2024).
    - Para obtener información detallada sobre la implementación de requisitos previos manuales, consulte [Laboratorio 6a](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab06a-ACSS_One_Day.md)
    - Para obtener información sobre la implementación de la infraestructura de demostración de ACSS, consulte [Laboratorio 6c(https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab06c-ACSS_One_Day.md)].

Tiempo estimado: 100 minutos

Todas las tareas de este laboratorio de cursos de AZ-1006 de un día se realizan desde Azure Portal

## Objetivos

Después de completar este laboratorio, podrá hacer lo siguiente:

- Implementación de los requisitos previos mínimos para evaluar la implementación de cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions
- Implementación de la infraestructura que hospeda las cargas de trabajo de SAP en Azure usando Azure Center for SAP solutions
- Mantenimiento de cargas de trabajo de SAP en Azure junto con el uso de Azure Center for SAP solutions

## Instrucciones

### Ejercicio 1: Implementación de los requisitos previos mínimos para evaluar la implementación de cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions

Duración: 30 minutos

En este ejercicio, implementará los requisitos previos mínimos para evaluar la implementación de cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions. Esto incluye las siguientes actividades:

>**Importante**: los requisitos previos implementados en este ejercicio no* están *diseñados para representar los procedimientos recomendados para implementar cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions. Su propósito es minimizar el tiempo, el costo y los recursos necesarios para evaluar la mecánica de la implementación de cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions y la realización de tareas posteriores a la implementación y mantenimiento. La implementación de los requisitos previos incluye las siguientes actividades:

- Creación de una identidad administrada asignada por el usuario de Microsoft Entra para que la use Azure Center for SAP solutions para el acceso a Azure Storage durante su implementación.
- Concesión de la identidad administrada asignada por el usuario de Microsoft Entra que se usa para realizar el acceso de implementación a la suscripción de Azure y la cuenta de uso general v2 de Azure Storage
- Creación de la red virtual de Azure que hospeda todas las máquinas virtuales de Azure incluidas en la implementación.
- Creación y configuración de un grupo de seguridad de red (NSG) que se usa para restringir el acceso saliente desde subredes de la red virtual que hospeda la implementación.
- Creación y configuración de una puerta de enlace NAT que permitirá la conectividad saliente desde máquinas virtuales de Azure que forman parte de la implementación.

Estas actividades corresponden a las siguientes tareas de este ejercicio:

- Tarea 1: Creación de una identidad administrada asignada por el usuario de Microsoft Entra
- Tarea 2: Configuración de asignaciones de roles de control de acceso basado en rol (RBAC) de Azure para la identidad administrada asignada por el usuario de Microsoft Entra ID
- Tarea 3: Creación de la red virtual de Azure
- Tarea 4: Creación y configuración de un grupo de seguridad de red
- Tarea 5: Creación y configuración de una puerta de enlace NAT

#### Tarea 1: Creación de una identidad administrada asignada por el usuario de Microsoft Entra

En esta tarea, creará una identidad administrada asignada por el usuario de Microsoft Entra que usará Azure Center for SAP solutions para el acceso a Azure Storage durante su implementación.

1. En el equipo de laboratorio, inicie un explorador web, vaya a Azure Portal en `https://portal.azure.com`, y autentíquese mediante una cuenta Microsoft o una cuenta de Microsoft Entra ID con el rol Propietario en la suscripción de Azure que use en este laboratorio.
1. En la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Identidades administradas**.
1. En la página **Identidades administradas**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear identidad administrada asignada por el usuario**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|Nombre de un **nuevo** grupo de recursos **acss-infra-RG**|
   |Region|el nombre de la región de Azure que se usa para la implementación de ACSS|
   |Nombre|**acss-infra-MI**|

1. En la pestaña **Revisar**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: no espere a que se complete el proceso de aprovisionamiento, sino que avance a la siguiente tarea. El aprovisionamiento debe tardar unos segundos.

   >**Nota**: en una de las próximas tareas, autorizará el acceso de la identidad administrada a la cuenta de almacenamiento que hospeda los medios de instalación de SAP para dar cabida a la instalación de software de SAP mediante Azure Center for SAP solutions.

#### Tarea 2: Configure las asignaciones de roles de control de acceso basado en rol (RBAC) de Azure para la cuenta de usuario de Microsoft Entra ID que se usará para realizar la implementación

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Suscripciones**.
1. En la página **Suscripciones**, seleccione la entrada que representa la suscripción de Azure que va a usar para este laboratorio. 
1. En la página que muestra las propiedades de la suscripción de Azure, seleccione **Control de acceso (IAM)**.
1. En la página **Control de acceso (IAM)**, seleccione **+ Agregar** y, después, en el menú desplegable, seleccione **Agregar asignación de roles**.
1. En la pestaña **Rol** de la página **Agregar asignación de roles**, en la lista de **roles de función de trabajo**, busque y seleccione la entrada del **rol de servicio Azure Center for SAP solutions** y seleccione **Siguiente**.
1. En la pestaña **Miembros** de la página **Agregar asignación de roles**, en **Asignar acceso a**, seleccione **Identidad administrada** y, a continuación, haga clic en **+ Seleccionar miembros**.
1. En el panel **Seleccionar identidades administradas**, especifique la siguiente configuración:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Identidad administrada|**Identidad administrada asignada por el usuario**|
   |Seleccionar|**acss-infra-MI**|
1. En la lista de identidades administradas, seleccione la entrada **acss-infra-MI** y, a continuación, haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Miembros**, seleccione **Revisar y asignar**.  
1. En la pestaña **Revisar y asignar**, seleccione **Revisar y asignar**.
   
   ##### Agregue: "Identidad del administrador de Azure Center for SAP solutions"
   
1. De nuevo en la página **Control de acceso (IAM),** seleccione **+ Agregar** y, en el menú desplegable, seleccione **Agregar asignación de roles**.
1. En la pestaña **Rol** de la página **Agregar asignación de roles**, en la lista de **roles de función de trabajo**, busque y seleccione la entrada del **administrador de Azure Center for SAP solutions** y seleccione **Siguiente**.
1. En la pestaña **Miembros**
   - para **Asignar acceso a**, seleccione **Usuario, Grupo o Principio de servicio**
   - Haga clic en **+ Seleccionar miembros**.
1. En el panel **Seleccionar miembros**, en el cuadro de texto **Seleccionar**, escriba el nombre de la cuenta de usuario de Microsoft Entra ID que usó para acceder a la suscripción de Azure que usa para este laboratorio, selecciónela en la lista de resultados que coincidan con la entrada y, a continuación, haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Miembros**, seleccione **Revisar y asignar**.
1. En la pestaña **Revisar y asignar**, seleccione **Revisar y asignar**.
   
   ##### Agregue: "Operador de identidad administrada"
   
1. De nuevo en la página **Control de acceso (IAM),** seleccione **+ Agregar** y, en el menú desplegable, seleccione **Agregar asignación de roles**.
1. En la pestaña **Rol** de la página **Agregar asignación de roles**, en la lista de **roles de función de trabajo**, busque y seleccione la entrada del **Operador de identidad administrada** y seleccione **Siguiente**.
1. En la pestaña **Miembros**
   - para **Asignar acceso a**, seleccione **Usuario, Grupo o Principio de servicio**
   - Haga clic en **+ Seleccionar miembros**.
1. En el panel **Seleccionar miembros**, en el cuadro de texto **Seleccionar**, escriba el nombre de la cuenta de usuario de Microsoft Entra ID que usó para acceder a la suscripción de Azure que usa para este laboratorio, selecciónela en la lista de resultados que coincidan con la entrada y, a continuación, haga clic en **Seleccionar**.
1. De nuevo en la pestaña **Miembros**, seleccione **Revisar y asignar**.
1. En la pestaña **Revisar y asignar**, seleccione **Revisar y asignar**.

#### Tarea 3: Crear la red virtual

En esta tarea, creará la red virtual de Azure que hospeda todas las máquinas virtuales de Azure incluidas en la implementación. Además, dentro de la red virtual, se crean las siguientes subredes:

- aplicación: diseñada para hospedar la aplicación SAP y los servidores de instancias de SAP Central Services
- db: diseñada para hospedar el nivel de base de datos de SAP

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Redes virtuales**.
1. En la página **Redes virtuales**, seleccione **y Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear red virtual**, especifique la siguiente configuración y seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre de la red virtual|**acss-infra-VNET**|
   |Region|el nombre de la misma región de Azure que usó en la tarea anterior de este ejercicio|

1. En la pestaña **Seguridad**, acepte la configuración predeterminada y seleccione **Siguiente**.

   >**Nota**: En este momento, puede aprovisionar tanto Azure Bastion como Azure Firewall, pero los aprovisionará por separado una vez creada la red virtual.

1. En la pestaña **Direcciones IP**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Espacio de direcciones IP|**10.0.0.0/16 (65 536 direcciones)**|

1. En la lista de subredes, seleccione el icono de papelera para eliminar la subred **predeterminada**.
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

#### Tarea 4: Creación y configuración de un grupo de seguridad de red

En esta tarea, creará y configurará un grupo de seguridad de red (NSG) que se usa para restringir el acceso saliente desde subredes de la red virtual que hospeda la implementación. Para ello, puede bloquear la conectividad a Internet, pero permitir explícitamente las conexiones a los siguientes servicios:

- Puntos de conexión de infraestructura de actualización de SUSE o Red Hat
- Azure Storage
- Azure Key Vault
- Microsoft Entra ID
- Azure Resource Manager

>**Nota**: en general, debe considerar la posibilidad de usar Azure Firewall en lugar de grupos de seguridad de red para proteger la conectividad de red para la implementación de SAP. 

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

1. En la página **acss-infra-NSG**, en el menú de navegación, en la sección **Configuración**, seleccione **Reglas de seguridad de salida**.
1. En la página **Reglas de seguridad de salida \| acss-infra-NSG**, seleccione **+ Agregar**.
1. En el panel **Agregar regla de seguridad de salida**, especifique la siguiente configuración y seleccione **Agregar**:

   >**Nota**: se debe agregar la siguiente regla para permitir explícitamente la conectividad a los puntos de conexión de infraestructura de actualización de Red Hat.

   >**Nota**: para identificar las direcciones IP que se van a usar para RHEL, consulte [Preparación de la red para la implementación de la infraestructura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Configuración|Valor|
   |---|---|
   |Origen|**Cualquiera**|
   |Rangos del puerto origen|*|
   |Destino|**Direcciones IP**|
   |Intervalos de direcciones IP de destino y CIDR|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198,52.136.197.163,20.225.226.182,52.142.4.99,20.248.180.252,20.24.186.80**|
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

1. En el panel **Agregar regla de seguridad de salida**, en el menú de navegación, en la sección **Configuración**, seleccione **Subredes**.
1. En la página **acss-infra-NSG \| Subredes**, seleccione **+ Asociar**.
1. En el panel **Asociar subred**, en la lista desplegable **Red virtual**, seleccione **acss-intra-VNET (acss-infra-RG)**, en la lista desplegable **Subred**, seleccione **aplicación** y, a continuación, seleccione **Aceptar**.
1. En el panel **Asociar subred**, en la lista desplegable **Red virtual**, seleccione **acss-intra-VNET (acss-infra-RG)**, en la lista desplegable **Subred**, seleccione **base de datos** y, a continuación, seleccione **Aceptar**.

#### Tarea 5: Creación y configuración de una puerta de enlace NAT

En esta tarea, creará y configurará una puerta de enlace de traducción de direcciones de red (NAT) que permitirá que las máquinas virtuales de Azure incluidas en la implementación lleguen a servicios externos, como puntos de conexión de SUSE y RHEL.

1. En el equipo de laboratorio, en la ventana del explorador web que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **NAT Gateways**.
1. En la página **NAT Gateways**, seleccione **+ Crear**.
1. En la **pestaña Aspectos básicos** de la **página Crear puerta de enlace de traducción de direcciones de red (NAT),** especifique la siguiente configuración y, a continuación, seleccione **Siguiente: Dirección IP de salida >**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-infra-RG**|
   |Nombre de NAT Gateway|**acss-infra-NATGW**|
   |Region|el nombre de la misma región de Azure que usó anteriormente en este ejercicio|
   |Zona de disponibilidad|**Sin zona**|
   |Tiempo de espera de inactividad de TCP (minutos)|**4**|

1. En la pestaña **IP de salida**, debajo de la lista desplegable **Direcciones IP públicas**, seleccione el vínculo **Crear una nueva dirección IP pública**, en el panel **Agregar una dirección IP pública**, en el cuadro de texto **Nombre**, escriba **acss-infra-NATWG-PIP** y seleccione **Aceptar**.
1. De nuevo en la pestaña **IP de salida**, seleccione **Siguiente: Subred >**.
1. En la pestaña **Subred**, en la lista desplegable **Red virtual**, seleccione **acss-infra-VNET**, en la lista de subredes, active la casilla situada junto a las entradas **aplicación** y **base de datos** y, a continuación, seleccione **Revisar y crear**:
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: Espere a que se complete el proceso de aprovisionamiento. El aprovisionamiento tardará alrededor de 1 minuto.

### Ejercicio 2: Implementación de la infraestructura que hospeda las cargas de trabajo de SAP en Azure usando Azure Center for SAP solutions

Duración: 40 minutos

En este ejercicio, realizará la implementación de Azure Center for SAP solutions. Esto incluye la siguiente actividad:

- Use Azure Center for SAP solutions para implementar la infraestructura capaz de hospedar cargas de trabajo de SAP en una suscripción de Azure.

Esta actividad corresponde a la siguiente tarea de este ejercicio:

- Tarea 1: Creación de soluciones de Azure Virtual Instance para SAP (VIS)

>**Nota**: después de la implementación correcta, puede continuar con la instalación del software de SAP mediante el uso de soluciones del Centro de Azure para SAP. Sin embargo, la instalación de software de SAP no se incluye en este laboratorio.

>**Nota**: para obtener información sobre cómo instalar software de SAP mediante el Centro de Azure para soluciones de SAP, consulte la documentación de Microsoft Learn que describe cómo [obtener medios](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) de instalación de SAP e [Instalar software](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software) SAP. 

#### Tarea 1: Creación de soluciones de Azure Virtual Instance para SAP (VIS)

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Azure Center for SAP Solutions**. 
1. En la página **Azure Center for SAP Solutions \| Información general**, seleccione **Crear un nuevo sistema SAP**.
1. En la pestaña **Aspectos básicos** de la página **Crear Virtual Instance for SAP solutions**, especifique la siguiente configuración y seleccione **Siguiente: Máquinas virtuales**

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|Nombre de un **nuevo** grupo de recursos **acss-vi-RG**|
   |Nombre (SID)|**VI1**|
   |Region|el nombre de la región de Azure que hospeda la implementación de SAP registrada por ACSS u otra región en la misma geografía|
   |Tipo de entorno|**No producción**|
   |Producto de SAP|**S/4HANA**|
   |Base de datos|**HANA**|
   |Método de escalado de HANA|**Escalado vertical (recomendado)**|
   |Tipo de implementación|**Distribuido con alta disponibilidad (HA)**|
   |Disponibilidad de proceso|**99.95 (conjunto de disponibilidad)**|
   |Red virtual|**acss-infra-VNET**|
   |Subred de aplicación|**aplicación (10.0.2.0/24)**|
   |Subred de base de datos|**db (10.0.3.0/24)**|
   |Opciones de imagen del sistema operativo de la aplicación|**Uso de una imagen de Marketplace**|
   |Imagen del sistema operativo de la aplicación|**Red Hat Enterprise Linux 8.2 para aplicaciones SAP: x64 Gen2 más reciente**|
   |Opciones de imagen del sistema operativo de base de datos|**Uso de una imagen de Marketplace**|
   |Imagen del sistema operativo de la base de datos|**Red Hat Enterprise Linux 8.2 para aplicaciones SAP: x64 Gen2 más reciente**|
   |Opción de transporte de SAP|**No incluir el directorio de transporte de SAP**|
   |Tipo de autenticación|**SSH público**|
   |Nombre de usuario|**contososapadmin**|
   |Origen de la clave pública SSH|**Generar par de claves nuevo**|
   |Nombre del par de claves|**contosovi1key**|
   |SQP FQDN|**sap.contoso.com**|
   |Origen de identidad administrada|**Uso de una identidad administrada asignada por el usuario existente**|
   |Nombre de la identidad administrada|**acss-infra-MI**|

1. En la pestaña **Máquinas virtuales**, especifique la configuración siguiente:

   |Configuración|Valor|
   |---|---|
   |Generación de recomendaciones basadas en|**SAP Application Performance Standard (SAPS): seleccione esta opción para proporcionar SAPS para el tamaño de la memoria de base de datos y el nivel de aplicación y haga clic en Generación de recomendaciones**|
   |SAPS para el nivel de aplicación|**1000**|
   |Tamaño de memoria de la base de datos (GiB)|**128**|

1. Seleccione **Generar recomendación**.
1. Revise el tamaño y el número de máquinas virtuales para las máquinas virtuales de ASCS, aplicación y base de datos.

   >**Nota**: si es necesario, ajuste los tamaños recomendados seleccionando el vínculo **Ver todos los tamaños** de cada conjunto de máquinas virtuales y eligiendo un tamaño alternativo. De forma predeterminada, el tipo de implementación distribuida con alta disponibilidad, así como el tamaño de memoria de base de datos y SAPS del nivel de aplicación especificado anteriormente, da como resultado las siguientes recomendaciones mínimas de SKU de máquina virtual:
   - 2 x Standard_D4ds_v5 para las máquinas virtuales ASCS (4 vCPU y 16 GiB de memoria cada una)
   - 2 x Standard_D4ds_v5 para las máquinas virtuales de la aplicación (4 vCPU y 16 GiB de memoria cada una)
   - 2 x Standard_E16ds_v5 para las máquinas virtuales de base de datos (16 vCPU y 128 GiB de memoria cada una)

   >**Nota**: si es necesario, puede solicitar el aumento de cuota seleccionando el vínculo **Solicitar cuota** para una SKU específica de máquinas virtuales y enviando una solicitud de aumento de cuota. El procesamiento de una solicitud suele tardar unos minutos.

   >**Nota**: Azure Center for SAP solutions aplica el uso de las SKU de máquina virtual admitidas por SAP durante la implementación.

1. En la pestaña **Máquinas virtuales**, en la sección **Discos de datos**, seleccione el vínculo **Ver y personalizar la configuración**.
1. En la página **Configuración del disco de base de datos**, revise la configuración recomendada sin realizar ningún cambio y seleccione **Cerrar**.
1. De nuevo en la pestaña **Máquinas virtuales**, seleccione **Siguiente: Visualizar arquitectura**.
1. En la pestaña **Visualizar arquitectura**, revise el diagrama que ilustra la arquitectura recomendada y seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación, active la casilla de confirmación que tiene una cuota amplia disponible en la región de implementación para evitar que se produzca un error de "Cuota insuficiente" y seleccione **Crear**.
1. Cuando se le solicite, en la ventana **Generar nuevo par de claves**, seleccione **Descargar clave privada y crear recursos**.

   >**Nota**: La clave privada necesaria para conectarse a las máquinas virtuales de Azure incluidas en la implementación se descargará en el equipo desde el que ejecuta este laboratorio.

   >**Nota**: Espere a que la implementación se complete. Esto puede tardar unos 25 minutos.

   >**Nota**: después de la implementación, puede continuar con la instalación del software de SAP mediante Azure Center for SAP solutions. En este laboratorio, explorará las funcionalidades de Azure Center for SAP solutions sin instalar software de SAP.

### Ejercicio 3: Mantenimiento de cargas de trabajo de SAP en Azure mediante Azure Center for SAP solutions

Duración: 60 minutos

En este ejercicio, revisará la administración y supervisión posteriores a la implementación de las cargas de trabajo de SAP mediante Azure Center for SAP solutions. Esto incluye las siguientes actividades:

- Implementación de requisitos previos para la copia de seguridad de cargas de trabajo de SAP administradas por Azure Center for SAP solutions 
- Implementación de requisitos previos para la recuperación ante desastres de cargas de trabajo de SAP administradas por Azure Center for SAP solutions 
- Revisión de las opciones de supervisión disponibles para cargas de trabajo de SAP administradas por Azure Center for SAP solutions 
- Eliminación de todos los recursos de Azure aprovisionados en este laboratorio.

Estas actividades corresponden a las siguientes tareas de este ejercicio:

- Tarea 1: Implementación de requisitos previos para la copia de seguridad de cargas de trabajo de SAP administradas por Azure Center for SAP solutions 
- Tarea 2: Implementación de requisitos previos para la recuperación ante desastres de cargas de trabajo de SAP administradas por Azure Center for SAP solutions 
- Tarea 3: Revisión de las opciones de supervisión de las cargas de trabajo de SAP administradas por Azure Center for SAP solutions 
- Tarea 4: Eliminación de los recursos de Azure aprovisionados en este laboratorio

#### Tarea 1: Implementación de requisitos previos para la copia de seguridad de cargas de trabajo de SAP administradas por Azure Center for SAP solutions

>**Nota**: al configurar Azure Backup en el nivel de recurso VIS en Azure Center for SAP solutions, puede, en un paso, habilitar la copia de seguridad de las máquinas virtuales de Azure que hospedan la base de datos, los servidores de aplicaciones y la instancia de SAP Central Services y para la base de datos de HANA. Para la copia de seguridad de base de datos de HANA, Azure Center for SAP solutions ejecuta automáticamente el script de registro previo de copia de seguridad.

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Azure Center for SAP Solutions**.
1. En la página **Azure Center for SAP solutions \| Información general**, en el menú de navegación vertical del lado izquierdo, seleccione **Instancias virtuales para soluciones de SAP** y, en la lista de instancias virtuales, seleccione la instancia que implementó en el ejercicio anterior.
1. En la página de la instancia virtual, en el menú de navegación vertical del lado izquierdo, en la sección **Operaciones**, seleccione **Copia de seguridad (versión preliminar)**.
1. Tenga en cuenta el mensaje que indica que no se puede configurar la copia de seguridad, ya que no se ha completado la instalación o registro de software de SAP para este sistema SAP.

   >**Nota**: Se espera que esto sea así. No podrá configurar la copia de seguridad de esta manera hasta que se complete la instalación del software de SAP. Sin embargo, completar la configuración también implica requisitos previos adicionales, incluida la creación de almacenes y directivas de copia de seguridad, que revisaremos aquí. 

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Centro de copia de seguridad**. 
1. En la página **Centro de copia de seguridad**, en el menú de navegación vertical del lado izquierdo, en la sección **Administrar**, seleccione **Almacenes**.
1. En la página **Centro de copia de seguridad\| Almacenes**, seleccione **+ Almacén**.
1. En la página **Inicio: crear almacén**, revise los tipos de almacén disponibles, asegúrese de que el **almacén de Recovery Services** (que admite **máquinas virtuales de Azure** y **SAP HANA en tipos de orígenes de datos de máquina virtual de Azure**) está seleccionado y, a continuación, seleccione **Continuar**.
1. En la pestaña **Aspectos básicos** de la página **Crear almacén de Recovery Services**, especifique la siguiente configuración y seleccione **Siguiente: Redundancia**

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|Nombre de un **nuevo** grupo de recursos **acss-mgmt-RG**|
   |Nombre del almacén|**acss-backup-RSV**|
   |Region|el nombre de la región de Azure que hospeda la implementación de SAP registrada por ACSS|

1. En la pestaña **Redundancia**, especifique la siguiente configuración y seleccione **Siguiente: Propiedades del almacén**

   |Configuración|Valor|
   |---|---|
   |Redundancia de almacenamiento de copias de seguridad|**Redundancia geográfica**|
   |Restauración entre regiones|**Habilitar**|

1. En la pestaña Propiedades** del **almacén, revise la **opción Habilitar inmutabilidad** sin habilitarla y seleccione **Siguiente : Redes**
1. En la pestaña **Redes**, acepte la opción predeterminada **Permitir el acceso público desde todas las redes** y seleccione **Revisar y crear**
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: Espere a que se complete el proceso de aprovisionamiento. El aprovisionamiento puede tardar aproximadamente 2 minutos.

1. En la página **Se completó la implementación**, haga clic en **Ir al recurso**.
1. En la página **acss-backup-RSV**, en el menú de navegación vertical del lado izquierdo, en la sección **Administrar**, seleccione **directivas de copia de seguridad**.
1. En la página **acss-backup-RSV \| Directivas de copia de seguridad de RSV**, seleccione **+ Agregar**.
1. En la página **Seleccionar tipo de directiva**, revise los tipos de directiva disponibles.

   >**Nota**: para empezar, configurará la directiva de copia de seguridad de las máquinas virtuales de Azure que hospedan la base de datos, los servidores de aplicaciones y la instancia de SAP Central Services.

1. En la página **Seleccionar tipo de directiva**, seleccione **Máquina virtual de Azure**.
1. En la página **Crear directiva**, revise las diferencias entre los subtipos de directiva **Estándar** y **Mejorado** y seleccione **Mejorado**.

   >**Nota**: debe basar su elección en sus necesidades de resistencia. Del mismo modo, debe ajustar los siguientes valores enumerados según sus necesidades.

1. En el cuadro de texto **Nombre de directiva**, escriba **acss-vm-enhanced-backup-policy**.
1. Establezca la frecuencia de programación de copia de seguridad en **Cada hora**, la hora de inicio en **6:00 p.m.**, programe en **Cada 6 horas**, la duración en **12 horas** y la zona horaria a la zona horaria local.
1. Establezca el valor de **Conservar instantánea(s) de recuperación de instancia** en **5** días.
1. Establezca **Retención de puntos de copia de seguridad diarios** en **60** días.
1. Establezca el período de retención de puntos de copia de seguridad semanales, mensuales y anuales en función de sus preferencias.

   >**Nota**: para habilitar la ordenación por niveles, debe habilitar puntos de copia de seguridad mensuales o anuales.

   >**Nota**: también tiene la opción de designar la convención de nomenclatura del grupo de recursos generado automáticamente por Azure Backup.

1. Seleccione **Crear**.
1. En la página **Se completó la implementación**, haga clic en **Ir al recurso**.
1. En la página **acss-backup-RSV**, en el menú de navegación vertical del lado izquierdo, en la sección **Administrar**, seleccione **directivas de copia de seguridad**.
1. En la página **acss-backup-RSV \| Directivas de copia de seguridad de RSV**, seleccione **+ Agregar**.

   >**Nota**: a continuación, configurará las directivas de copia de seguridad de la base de datos de HANA. La configuración que se presenta aquí sigue las instrucciones descritas en el artículo de MS Learn [Copia de seguridad de instantáneas de instancia de base de datos de SAP HANA en máquinas virtuales de Azure](https://learn.microsoft.com/en-us/azure/backup/sap-hana-database-instances-backup).

1. En la página **Seleccionar tipo de directiva**, seleccione **SAP HANA en la máquina virtual de Azure (base de datos a través de Backint)**
1. En la página **Crear directiva**, en el cuadro de texto **Nombre de directiva**, escriba **acss-hanadb-backint-backup-policy**.
1. Acepte la configuración predeterminada para las copias de seguridad completas y de registro. Mantenga deshabilitadas las copias de seguridad diferenciales e incrementales.

   >**Nota**: tiene la opción de habilitar la compresión de copia de seguridad de HANA y mover puntos de recuperación aptos al archivo del almacén. 

1. Seleccione **Crear**.

1. En la página **Se completó la implementación**, haga clic en **Ir al recurso**.
1. En la página **acss-backup-RSV**, en el menú de navegación vertical del lado izquierdo, en la sección **Administrar**, seleccione **directivas de copia de seguridad**.
1. En la página **acss-backup-RSV \| Directivas de copia de seguridad de RSV**, seleccione **+ Agregar**.
1. En la página **Seleccionar tipo de directiva**, seleccione **SAP HANA en la máquina virtual de Azure (instancia de base de datos mediante instantánea)**
1. En la página **Crear directiva**, en el cuadro de texto **Nombre de directiva**, escriba **acss-hanadb-snapshot-backup-policy**.
1. Establezca la frecuencia de copia de seguridad de instantáneas a la 1:30 p. m. de su zona horaria actual.
1. Configure para conservar la instantánea de recuperación instantánea durante **5** días.
1. Mantenga la selección predeterminada del grupo de recursos de instantáneas, que debe establecerse en **acss-mgmt-RG**.
1. En la sección **identidad administrada**, seleccione **Crear identidad administrada**. Esto abre automáticamente otra pestaña en la ventana del explorador web, en la que se muestra la página **identidad administrada** en Azure Portal.
1. En la página **Identidad administrada**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear identidad administrada asignada por el usuario**, especifique la siguiente configuración y, a continuación, seleccione **Revisar y crear**:

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-mgmt-RG**|
   |Region|el nombre de la región de Azure que hospeda la implementación de ACSS|
   |Nombre|**acss-mgmt-MI**|

1. En la pestaña **Revisar**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: Espere a que se complete el proceso de aprovisionamiento. El aprovisionamiento debe tardar unos segundos.

1. Cierre la pestaña actual del explorador y vuelva a la que muestra la página **Crear directiva**.
1. En la sección **Identidad administrada**, en la lista desplegable **Grupo de recursos**, seleccione **acss-mgmt-RG** y, en la lista desplegable **Identidad administrada**, seleccione **acss-mgmt-MI**.
1. Seleccione **Crear**.

   >**Nota**: al configurar la copia de seguridad en el nivel VIS en la interfaz de Azure Center for SAP solutions, podrá aprovechar el almacén existente y sus directivas.

   >**Nota**: una vez configurada la copia de seguridad en el nivel VIS, puede supervisar el estado de los trabajos de copia de seguridad de las máquinas virtuales de Azure y la base de datos de HANA desde la interfaz VIS de Azure Portal.

#### Tarea 2: Implementación de requisitos previos para la recuperación ante desastres de cargas de trabajo de SAP administradas por Azure Center for SAP solutions 

>**Nota**: aunque el servicio Azure Center for SAP solutions es un servicio con redundancia de zona, no hay ninguna conmutación por error iniciada por Microsoft en caso de interrupción de una región. Para corregir este escenario, debe configurar la recuperación ante desastres para los sistemas SAP implementados mediante Azure Center for SAP solutions siguiendo las instrucciones descritas en [Introducción a la recuperación ante desastres e instrucciones de infraestructura para la carga de trabajo de SAP](https://learn.microsoft.com/en-us/azure/sap/workloads/disaster-recovery-overview-guide), lo que implica el uso de Azure Site Recovery (ASR). En esta tarea, recorrerá el proceso de implementación de una solución de recuperación ante desastres basada en ASR que se basa en esa guía.

>**Nota**: ASR es la solución recomendada para los servidores de aplicaciones y las instancias de SAP Central Services. En el caso de los servidores de bases de datos, debe considerar el uso de su funcionalidad de replicación nativa.

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Almacenes de Recovery Services**.
1. En la página **Almacén de Recovery Services**, seleccione **+ Crear**.
1. En la pestaña **Aspectos básicos** de la página **Crear almacén de Recovery Services**, especifique la siguiente configuración (deje los demás con sus valores predeterminados) y seleccione **Siguiente: Redundancia**.

   |Configuración|Valor|
   |---|---|
   |Suscripción|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|Nombre de un **nuevo** grupo de recursos **acss-dr-RG**|
   |Nombre del almacén|**acss-dr-RSV**|
   |Region|el nombre de la región de Azure emparejada con la implementación de SAP registrada por ACSS|

   >**Nota**: para identificar la región que está emparejada con la que hospeda las cargas de trabajo de producción, consulte la documentación de MS Learn que describe las [regiones emparejadas de Azure](https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure#azure-paired-regions).

1. En la pestaña **Redundancia**, especifique la siguiente configuración y seleccione **Siguiente: Propiedades del almacén**

   |Configuración|Valor|
   |---|---|
   |Redundancia de almacenamiento de copias de seguridad|**Redundancia local**|

1. En la pestaña Propiedades** del **almacén, revise la **opción Habilitar inmutabilidad** sin habilitarla y seleccione **Siguiente : Redes**
1. En la pestaña **Redes**, acepte la opción predeterminada **Permitir el acceso público desde todas las redes** y seleccione **Revisar y crear**
1. En la pestaña **Revisar y crear**, espere a que se complete el proceso de validación y seleccione **Crear**.

   >**Nota**: no espere a que se complete el proceso de aprovisionamiento, sino que avance al siguiente paso. El aprovisionamiento puede tardar aproximadamente 2 minutos.

   >**Nota**: ahora configurará el entorno de recuperación ante desastres en la región emparejada en la que creó el almacén de Recovery Services. Este entorno incluirá una red virtual que hospedará réplicas de las máquinas virtuales de Azure hospedadas actualmente en la región primaria donde aprovisionó Instancia virtual para SAP. 

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Almacenes de Recovery Services**.
1. En la página **Almacenes de Azure Recovery Services**, seleccione **acss-dr-RSV**.
1. En la página **acss-dr-RSV**, en el menú de navegación vertical del lado izquierdo, en la sección **Introducción**, seleccione **Site Recovery**.
1. En la página **acss-dr-RSV \| Site Recovery**, en la sección **máquinas virtuales de Azure**, seleccione **1. Habilitación de la replicación**. 
1. En la pestaña **Origen** de la página **Habilitar replicación**, especifique la siguiente configuración y, a continuación, seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Region|el nombre de la región de Azure que hospeda la instancia virtual para SAP (VIS)|
   |Subscription|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-vi-RG**|
   |Modelo de implementación de máquina virtual|**Resource Manager**|
   |Recuperación ante desastres entre zonas de disponibilidad|**No**|

   >**Nota**: Es posible que la **recuperación ante desastres entre zonas de disponibilidad** no sea configurable, en función de si la región de origen admite zonas de disponibilidad.

1. En la pestaña **Máquinas virtuales**, seleccione las cuatro primeras máquinas virtuales de la lista (**vi1appvm0**, **vi1appvm1**, **vi1ascsvm0**, and **vi1ascsvm0**) y, a continuación, seleccione **Siguiente**.

   >**Nota**: como se mencionó anteriormente, la replicación basada en ASR se aplicará a los servidores de aplicaciones y a las instancias de SAP Central Services. Los servidores de bases de datos se mantendrán sincronizados mediante la función de replicación de base de datos nativa.

1. En la pestaña **Configuración de replicación**, realice las siguientes acciones:

   1. Si es necesario, en la lista desplegable **Ubicación de destino**, seleccione la región de Azure en la que creó el almacén de Recovery Services **acss-dr-RSV**.
   1. Asegúrese de que el nombre de la suscripción de Azure que se usa en este laboratorio aparece en la lista desplegable **Suscripción de destino**.
   1. En la lista desplegable **Grupo de recursos de destino**, seleccione **acss-dr-RG**.
   1. Debajo de la lista desplegable **Red virtual de conmutación por error**, seleccione **Crear nueva**.
   1. En el panel **Crear red virtual**, en el cuadro de texto **Nombre**, escriba **CONTOSO-VNET-asr**
   1. En la sección **Espacio de direcciones**, en el cuadro de texto **Intervalo de direcciones**, reemplace la entrada predeterminada por **10.10.0.0/16**.
   1. En la sección **Subredes**, en el cuadro de texto **Nombre de subred**, escriba **aplicación** y, en el cuadro de texto **Intervalo de direcciones**, escriba **10.10.0.0/24**.
   1. Debajo de la entrada de subred recién agregada, en la sección **Subredes**, en el cuadro de texto **Nombre de subred**, escriba **base de datos** y, en el cuadro de texto **Intervalo de direcciones**, escriba **10.10.2.0/24**.
   1. En el panel **Crear red virtual**, seleccione **Aceptar**.
   1. De nuevo en la página **Habilitar replicación**, asegúrese de que la entrada **(nueva) aplicación (10.10.0.0/24)** aparece en la lista desplegable **Subred de conmutación por error**.
   1. En la sección **Almacenamiento**, seleccione el vínculo **Ver o editar configuración de almacenamiento**.
   1. En la página **Personalizar la configuración de destino**, revise la configuración resultante, pero no realice ningún cambio y seleccione **Cancelar**.
   1. En la sección **Opciones de disponibilidad**, seleccione el vínculo **Ver o editar opciones de disponibilidad**.
   1. En la página **Opciones de disponibilidad** tiene la opción de implementar grupos de selección de ubicación de proximidad para los recursos de destino, pero no realice ningún cambio y seleccione **Cancelar**.

   >**Nota**: también tiene la opción de configurar la reserva de capacidad.

   >**Importante**: tenga en cuenta que los espacios de direcciones IP difieren entre la red virtual en las regiones primarias y secundarias. Esto es intencionado, ya que permitirá conectar las dos redes virtuales juntas, lo que es necesario para configurar la replicación entre los servidores de bases de datos hospedados en las dos regiones. Esta conexión se puede establecer mediante el emparejamiento de red virtual.

1. De nuevo en la pestaña **Configuración de replicación** de la página **Habilitar replicación**, seleccione **Siguiente**.
1. En la pestaña **Administrar** de la página **Habilitar replicación**, realice las siguientes acciones:

   1. Debajo de la lista desplegable **Directiva de replicación**, seleccione **Crear nueva**.
   1. En el panel **Crear directiva de replicación**, en el cuadro de texto **Nombre**, escriba **2-day-retention-policy**, en el cuadro de texto **Período de retención (en días)**, escriba **2** y, a continuación, seleccione **Aceptar**.
   1. Tiene la opción de crear grupos de replicación. Esta opción no es aplicable a nuestro caso de uso.
   1. Deje la opción **Actualizar configuración** de **Ajustes de extensión** configurada para **Permitir que ASR administre**.
   1. Acepte el nombre predeterminado asignado automáticamente de la **cuenta de Automation**.

1. En la pestaña **Administrar** de la página **Habilitar replicación**, seleccione **Siguiente**.
1. En la pestaña **Revisar** de la página **Habilitar replicación**, seleccione **Habilitar replicación**.

   >**Nota**: ahora recorrerá paso a paso la habilitación de la replicación para las dos máquinas virtuales restantes de Azure que hospedan los servidores de base de datos.

1. De nuevo en la página **acss-dr-RSV \| Site Recovery**, en la sección **máquinas virtuales de Azure**, seleccione **1. Habilitación de la replicación**. 
1. En la pestaña **Origen** de la página **Habilitar replicación**, especifique la siguiente configuración y, a continuación, seleccione **Siguiente**:

   |Configuración|Valor|
   |---|---|
   |Region|el nombre de la región de Azure que hospeda la instancia virtual para SAP (VIS)|
   |Subscription|Nombre de la suscripción de Azure que se usa en este laboratorio|
   |Resource group|**acss-vi-RG**|
   |Modelo de implementación de máquina virtual|**Resource Manager**|
   |Recuperación ante desastres entre zonas de disponibilidad|**No**|

   >**Nota**: Es posible que la **recuperación ante desastres entre zonas de disponibilidad** no sea configurable, en función de si la región de origen admite zonas de disponibilidad.

1. En la pestaña **Máquinas virtuales**, seleccione las cuatro primeras máquinas virtuales de la lista (**vi1appvm0**, **vi1appvm1**, **vi1ascsvm0**, and **vi1ascsvm0**) y, a continuación, seleccione **Siguiente**.

   >**Nota**: debe configurar la replicación para las dos máquinas virtuales de Azure restantes por separado, ya que están conectadas a otra subred.

1. En la pestaña **Configuración de replicación**, realice las siguientes acciones:

   1. Si es necesario, en la lista desplegable **Ubicación de destino**, seleccione la región de Azure en la que creó el almacén de Recovery Services **acss-dr-RSV**.
   1. Asegúrese de que el nombre de la suscripción de Azure que se usa en este laboratorio aparece en la lista desplegable **Suscripción de destino**.
   1. En la lista desplegable **Grupo de recursos de destino**, seleccione **acss-dr-RG**.
   1. En la lista desplegable **Red virtual de conmutación por error**, seleccione **CONTOSO-VNET**.
   1. En la lista desplegable **Subred de conmutación** por error, seleccione **aplicación (10.10.0.0/24)**.
   1. En la sección **Almacenamiento**, seleccione el vínculo **Ver o editar configuración de almacenamiento**.
   1. En la página **Personalizar la configuración de destino**, revise la configuración resultante, pero no realice ningún cambio y seleccione **Cancelar**.
   1. En la sección **Opciones de disponibilidad**, seleccione el vínculo **Ver o editar opciones de disponibilidad**.
   1. En la página **Opciones de disponibilidad**, tiene la opción de implementar grupos de selección de ubicación de proximidad para los recursos de destino, pero no realice ningún cambio y seleccione **Cancelar**.

   >**Nota**: también tiene la opción de configurar la reserva de capacidad.

1. De nuevo en la pestaña **Configuración de replicación** de la página **Habilitar replicación**, seleccione **Siguiente**.
1. En la pestaña **Administrar** de la página **Habilitar replicación**, realice las siguientes acciones:

   1. Debajo de la lista desplegable **Directiva de replicación**, seleccione **Crear nueva**.
   1. En el panel **Crear directiva de replicación**, en el cuadro de texto **Nombre**, escriba **2-day-retention-policy**, en el cuadro de texto **Período de retención (en días)**, escriba **2** y, a continuación, seleccione **Aceptar**.
   1. Tiene la opción de crear grupos de replicación. Esta opción no es aplicable a nuestro caso de uso.
   1. Deje la opción **Actualizar configuración** de **Ajustes de extensión** configurada para **Permitir que ASR administre**.
   1. Acepte el nombre predeterminado asignado automáticamente de la **cuenta de Automation**.

1. En la pestaña **Administrar** de la página **Habilitar replicación**, seleccione **Siguiente**.
1. En la pestaña **Revisar** de la página **Habilitar replicación**, seleccione **Habilitar replicación**.

   >**Nota**: la replicación inicial puede tardar un tiempo considerable en completarse. Teniendo en cuenta el tiempo limitado asignado a este laboratorio, consulte al instructor en relación con los pasos adicionales que se deben realizar como parte de esta tarea. En ausencia de cualquier guía específica, continúe directamente con la siguiente tarea.

   >**Nota**: en este momento, puede aprovisionar tanto Azure Bastion como Azure Firewall, pero en su lugar debe automatizar su aprovisionamiento como parte del procedimiento de conmutación por error de recuperación ante desastres. Esto minimizará los cargos asociados al mantenimiento del entorno de recuperación ante desastres. Lo mismo debe aplicarse a otros componentes de ese entorno que reflejen la configuración de la instancia virtual principal para SAP, como los recursos compartidos de archivos Premium de Azure y el enrutamiento personalizado.

#### Tarea 3: Revisión de las opciones de supervisión de las cargas de trabajo de SAP administradas por Azure Center for SAP solutions

>**Nota**: al igual que con la copia de seguridad, no podrá experimentar completamente las funcionalidades de supervisión de las soluciones de Azure Center for SAP solutions. Esto requiere la instalación del software de SAP o el registro de una instancia existente de Azure Monitor para soluciones de SAP. En su lugar, en esta tarea, recorrerá la interfaz disponible en la Azure Virtual Instance for SAP solutions para identificar y revisar estas funcionalidades.

1. En el equipo de laboratorio, en la ventana de Microsoft Edge que muestra Azure Portal, en el cuadro de texto **Buscar**, busque y seleccione **Instancias virtuales para soluciones de SAP**. 
1. En la página **Instancias virtuales para soluciones de SAP**, revise la información de estado resumida de la instancia **VI1**, incluidos los indicadores visuales de estado y estado generales.
1. En la página **Instancias virtuales para soluciones de SAP**, seleccione **VI1**.
1. En la página **VI1**, en el menú de navegación vertical del lado izquierdo, seleccione **Información general** y, a continuación, en el panel del lado derecho, seleccione **Supervisión**.
1. Revise la telemetría de supervisión que se muestra en el panel de supervisión.

   >**Nota**: el panel de supervisión incluye gráficos y métricas de uso de vCPU para servidores de aplicaciones, servidores de bases de datos e instancias de SAP Central Services. También incluye servidores de bases de datos de estadísticas de IOPS de disco de base de datos. 

1. En la página **VI1**, en el menú de navegación vertical del lado izquierdo, en la sección **Supervisión**, seleccione **Información de calidad**.
1. En la página **VI1 \| Información de calidad \| Libro 1**, revise la pestaña **Recomendación de Advisor**, que está pensada para proporcionar recomendaciones para optimizar la Azure Virtual Instance for SAP solutions (VIS), la instancia de servidor central, las instancias de App Service y las bases de datos.

   >**Nota**: estas recomendaciones requieren completar la instalación del software de SAP.

1. En la página **VI1 \| Información de calidad \| Libro 1**, seleccione la pestaña **Máquina virtual** y revise el contenido de las pestañas **Azure Compute**, **Lista de proceso**, **Extensiones de proceso**, **Proceso y disco de sistema operativo** y**Proceso y discos de datos**.

   >**Nota**: cada una de estas pestañas debe incluir datos reales recopilados de las máquinas virtuales de Azure que forman parte de la instancia virtual para soluciones de SAP.

1. En la página **VI1 \| Información de calidad \| Libro 1**, seleccione la pestaña **Comprobaciones de configuración** y revise el contenido de las pestañas **Redes aceleradas**, **IP pública**, **Copia de seguridad** y **Load Balancer**. Este contenido proporciona información general rápida sobre la configuración relacionada con el rendimiento y la seguridad de los componentes de proceso y red de la Virtual Instance for SAP solutions. La pestaña **Load Balancer** incluye información del **Monitor de Load Balancer** que muestra las métricas clave del equilibrador de carga.
1. En la página **VI1**, en el menú de navegación vertical del lado izquierdo, en la sección **Supervisión**, seleccione **Azure Monitor para soluciones de SAP**.
1. En la página **VI1 \| Azure Monitor para soluciones de SAP**, tenga en cuenta el mensaje que indica que AMS no se puede configurar, ya que no se ha completado el registro o la instalación de software de SAP para VIS.

   >**Nota**: una vez instalado el software de SAP, podrá integrarlo con un recurso de soluciones de Azure Monitor para SAP nuevo o existente. Azure Monitor para soluciones de SAP se basa en las funcionalidades de Azure Monitor de Log Analytics y libros para proporcionar una supervisión completa de las cargas de trabajo de SAP hospedadas en máquinas virtuales de Azure, incluida la compatibilidad con visualizaciones personalizadas, consultas y alertas.
