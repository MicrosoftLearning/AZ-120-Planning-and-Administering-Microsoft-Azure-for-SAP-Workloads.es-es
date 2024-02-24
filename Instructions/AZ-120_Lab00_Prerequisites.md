---
lab:
  title: '00: requisitos previos del laboratorio'
  module: Module 00 - Lab prerequisites
---

# AZ 120: Requisitos previos del laboratorio

## Requisitos principales de vCPU

> **Importante**: Los requisitos principales de vCPU dependen de los laboratorios que quiera implementar.

- Para completar el laboratorio 04b: implementación de la arquitectura de SAP en máquinas virtuales de Azure que ejecutan Windows, necesitará una suscripción de Microsoft Azure con al menos 28 vCPU disponibles en la región de Azure que admita zonas de disponibilidad en las que residirán las máquinas virtuales de Azure implementadas en este laboratorio.

    - 4 x Standard_DS1_v2 (1 vCPU cada una) = 4
    - 6 x Standard_D4s_v3 (4 vCPUs cada una) = 24

    > **Nota**: Considere la posibilidad de usar las regiones **Este de EE. UU.** o **Este de EE. UU. 2** para la implementación de los recursos.

    > **Nota**: para identificar las regiones de Azure que admiten zonas de disponibilidad, consulte <https://docs.microsoft.com/en-us/azure/availability-zones/az-overview>

- Para completar el laboratorio 05: automatización de la implementación mediante Azure Center for SAP solutions, necesitará una suscripción de Microsoft Azure con la siguiente disponibilidad de vCPU en la región de Azure donde residirán las máquinas virtuales de Azure implementadas en este laboratorio.

    - 4 x Standard_E4ds_v4 (4 vCPU cada una) o 4 X Standard_D4ds_v4 (4 vCPU cada una) = 8
    - 2 x Standard_M64ms (64 vCPU cada una) = 128

>**Nota**: para minimizar los requisitos de memoria y vCPU, puede usar Standard_M32ts (32 vCPU y 192 GiB de memoria cada una) en lugar de la Standard_M64m.

>**Nota**: Aunque los requisitos de vCPU para los tres primeros laboratorios de este curso son inferiores, se recomienda solicitar el aumento de cuotas para satisfacer los requisitos de todos los laboratorios, ya que el proceso de aumento de cuotas puede tardar algún tiempo (aunque las solicitudes de aumento de cuota se completen normalmente durante el mismo día laborable).

## Antes del laboratorio práctico

Período de tiempo: 30 minutos

### Tarea 1: validación del número suficiente de núcleos de vCPU para el laboratorio Implementación de la arquitectura de SAP en máquinas virtuales de Azure que ejecutan Windows

1. En el equipo de laboratorio, inicie un explorador web y vaya a Azure Portal en `https://portal.azure.com`.
1. En Azure Portal, inicie una sesión de PowerShell en Cloud Shell. 

    > **Nota**: si es la primera vez que inicia Cloud Shell en la suscripción actual de Azure, se le pedirá que cree un recurso compartido de archivos de Azure para conservar los archivos de Cloud Shell. Si es así, acepte los valores predeterminados, lo que dará lugar a la creación de una cuenta de almacenamiento en un grupo de recursos generado automáticamente.

1. En Azure Portal, en el panel de **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute lo siguiente: donde `<Azure_region>` designa la región de Azure de destino que va a usar para este laboratorio (por ejemplo, `eastus`):

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **Nota**: Para identificar los nombres de las regiones de Azure, en **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute `(Get-AzLocation).Location`
   
1. Revise el valor actual y las entradas de límite en la salida de los comandos ejecutados en el paso anterior y asegúrese de que tiene un número suficiente de vCPU en la región de Azure de destino.
1. Si el número de vCPU no es suficiente, en Azure Portal, vuelva a la hoja de suscripción y haga clic en **Uso y cuotas**. 
1. En la hoja **Uso y cuotas** de la suscripción, haga clic en **Solicitar aumento**.
1. En la hoja **Descripción del problema**, especifique lo siguiente:

    -   Tipo de problema: **Límites de servicio y suscripción (cuotas)**
    -   Suscripción: nombre de la suscripción de Azure que usará en este laboratorio
    -   Tipo de cuota: **El límite de suscripción de proceso o máquina virtual (núcleos o vCPU) aumenta**

1. Haga clic **Administrar cuota**.
1. Use el menú desplegable de ubicación para filtrar los resultados a la región de Azure que planea usar. Se recomienda usar **Este de EE. UU.** o **Este de EE. UU. 2**.
1. Busque el tipo de cuota de la **familia DSv3 estándar de vCPU** y seleccione el lápiz de edición.
1. En el campo Nuevo límite, especifique **40** y haga clic en **Guardar y continuar**.
1. Busque el **tipo de cuota Total de vCPU regionales** y seleccione el lápiz de edición.
1. En el campo Nuevo límite, especifique **40** y haga clic en **Guardar y continuar**.

   > **Nota**: la solicitud de aumento de cuota debe aprobarse automáticamente. Si se deniega la solicitud, continúe para abrir una incidencia de soporte técnico para solicitar el aumento de la cuota.

### Tarea 2: validación del número suficiente de núcleos de vCPU para el laboratorio Automatización de la implementación mediante Azure Center for SAP solutions

> **Nota**: para completar este laboratorio (como se describe), necesitará una suscripción de Microsoft Azure con las cuotas de vCPU que admiten la implementación de las siguientes máquinas virtuales:

- 2 máquinas virtuales Standard_E4ds_v4 (4 vCPU y 32 GiB de memoria cada una) o 2 Standard_D4ds_v4 (4 vCPU y 16 GiB de memoria cada una) para el nivel ASCS
- 2 máquinas virtuales Standard_E4ds_v4 (4 vCPU y 32 GiB de memoria cada una) o 2 Standard_D4ds_v4 (4 vCPU y 16 GiB de memoria cada una) para la capa de aplicación 
- 2 máquinas virtuales Standard_M64ms (64 vCPU y 1750 GiB de memoria cada una) para el nivel de base de datos

1. En el equipo de laboratorio, inicie un explorador web y vaya a Azure Portal en `https://portal.azure.com`.
1. En Azure Portal, seleccione el icono **Cloud Shell** e inicie una sesión de PowerShell en Cloud Shell. 
1. En Azure Portal, en el panel de **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute lo siguiente: donde `<Azure_region>` designa la región de Azure a la que va a implementar recursos en este laboratorio (por ejemplo, `eastus`):

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'cores'}
    ```

    > **Nota**: Para identificar los nombres de las regiones de Azure, en **Cloud Shell**, en el símbolo del sistema de PowerShell, ejecute `(Get-AzLocation).Location`

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

    > **Nota**: espere hasta que se complete correctamente la solicitud para aumentar los límites de cuota antes de iniciar la implementación automatizada del laboratorio mediante Azure Center for SAP solutions.
