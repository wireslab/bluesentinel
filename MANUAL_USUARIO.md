# Manual de Usuario: Monitoreo de Endpoints con Wazuh Dashboard

## Introducción

Este manual guía a los usuarios sobre cómo configurar y monitorear endpoints Windows y Linux usando el dashboard de Wazuh que forma parte del stack Blue Sentinel.

## Prerrequisitos

- Acceso al dashboard de Wazuh (https://localhost:443 por defecto)
- Usuario y contraseña válidos (admin/SecretPassword por defecto)
- Conexión de red entre los endpoints y el servidor Wazuh
- Agentes Wazuh instalados en los endpoints objetivo

## Configuración de Agentes

### Instalación del Agente en Windows

1. Descarga el instalador del agente Wazuh desde el sitio oficial o el servidor Wazuh
2. Ejecuta el instalador como administrador
3. Durante la instalación, proporciona la IP o nombre de dominio del servidor Wazuh
4. Configura el servidor Wazuh con el certificado o token proporcionado

### Instalación del Agente en Linux

1. Ejecuta el siguiente comando en el terminal del servidor Linux:
   ```bash
   curl -s https://packages.wazuh.com/4.x/apt/install.sh | sudo bash
   sudo apt install wazuh-agent
   ```

2. Edita el archivo de configuración `/var/ossec/etc/ossec.conf`:
   ```bash
   sudo nano /var/ossec/etc/ossec.conf
   ```

3. Asegúrate de que la sección de servidor apunte al servidor Wazuh:
   ```xml
   <client>
     <server>
       <address>TU_SERVIDOR_WAZUH</address>
       <port>1514</port>
       <protocol>tcp</protocol>
     </server>
   </client>
   ```

4. Reinicia el servicio:
   ```bash
   sudo systemctl restart wazuh-agent
   sudo systemctl enable wazuh-agent
   ```

## Conexión de Agentes al Servidor

### En el Dashboard de Wazuh

1. Accede al dashboard de Wazuh con las credenciales apropiadas
2. Navega a **Agents** en el menú lateral izquierdo
3. Aquí verás todos los agentes conectados al servidor
4. Los agentes recién instalados aparecerán como "Never connected" hasta que se registren

### Aprobación de Agentes

1. En la sección de Agents, selecciona los agentes pendientes
2. Haz clic en **Actions** y selecciona **Approve selected agents**
3. Alternativamente, puedes aprobar agentes individualmente
4. Una vez aprobados, los agentes comenzarán a enviar datos al servidor

## Configuración del Monitoreo

### Monitoreo de Windows (Documentos)

El stack Blue Sentinel incluye reglas personalizadas para monitorear cambios en la carpeta "Documentos" de Windows:

1. Estas reglas están configuradas en el servidor Wazuh (`custom-rules/001-windows-docs_rules.xml`)
2. Detectan modificaciones, creaciones y eliminaciones en carpetas `C:\Users\*\Documents`
3. Generan alertas de nivel 10 (Alta Prioridad) en el dashboard
4. Las alertas aparecerán en la sección de **Security events**

### Monitoreo de Linux WordPress

Para monitorear servidores Linux con WordPress:

1. Asegúrate de que el agente Wazuh pueda leer los logs de WordPress/Apache/Nginx
2. Edita el archivo `/var/ossec/etc/ossec.conf` en el agente Linux para incluir:

   ```xml
   <localfile>
     <log_format>apache</log_format>
     <location>/var/log/apache2/access.log</location>
   </localfile>
   ```

3. Estas reglas están configuradas en el servidor Wazuh (`custom-rules/002-linux-wordpress_rules.xml`)
4. Detectan intentos de inicio de sesión fallidos en WordPress
5. Generan alertas de nivel 12 (Crítica) en el dashboard

## Interfaz de Usuario del Dashboard

### Vista General de Agentes

1. Navega a **Agents** → **Production** (o la vista que tengas configurada)
2. Aquí puedes ver:
   - Estado de conexión de los agentes
   - Sistema operativo
   - Última conexión
   - Grupos asignados

### Vista de Eventos de Seguridad

1. Navega a **Security events** en el menú lateral
2. Filtra eventos por:
   - Agente específico (por nombre o ID)
   - Nivel de alerta
   - Tipo de regla
   - Fecha y hora

### Dashboard Personalizados

1. Dirígete a **Dashboards** para ver vistas predefinidas
2. Filtra por agente específico para ver solo sus eventos
3. Usa **Discover** para búsquedas personalizadas

## Monitoreo en Tiempo Real

### Visualización de Eventos

1. En la sección **Security events**, los eventos aparecen en tiempo real
2. Para filtrar por un agente específico:
   - Haz clic en el ícono de lupa junto al campo "agent.name"
   - Selecciona el agente deseado
3. Para ver solo eventos de nivel crítico:
   - Filtra por "rule.level" con valor mayor o igual a 10

### Alertas Personalizadas

1. Puedes crear alertas personalizadas en **Rules** → **Groups**
2. Las reglas de Blue Sentinel ya están configuradas para detectar eventos relevantes
3. Revisa las reglas en el directorio `custom-rules` para entender su funcionamiento

## Tareas Comunes

### Verificar el Estado de un Agente

1. Ve a la sección **Agents**
2. Busca el agente por nombre o IP
3. El icono de color verde indica conexión activa
4. El icono de color rojo indica desconexión

### Verificar Registros de Eventos

1. Navega a **Security events**
2. Filtra por el agente específico
3. Examina los eventos recientes
4. Revisa las descripciones para entender el tipo de evento

### Crear un Reporte

1. Navega a **Modules** → el módulo deseado
2. Personaliza la vista según tus necesidades
3. Haz clic en el botón de exportación
4. Selecciona el formato y el rango de fechas
5. Descarga el reporte

## Resolución de Problemas

### Agente No Conectado

1. Verifica la conectividad de red entre el agente y el servidor
2. Asegúrate de que los puertos 1514 y 1515 estén abiertos
3. Revisa los logs del agente en `/var/ossec/logs/ossec.log`
4. Confirma que el agente esté configurado con la IP correcta del servidor

### No Aparecen Eventos

1. Verifica que las reglas personalizadas estén correctamente instaladas
2. Revisa en el servidor Wazuh que los archivos XML estén en la ubicación correcta
3. Confirma que el agente esté enviando datos revisando los logs
4. Verifica que las reglas coincidan con los eventos que esperas capturar

## Seguridad y Acceso

### Gestión de Usuarios

1. Accede a **Security** → **Users** para gestionar cuentas
2. Crea roles específicos basados en necesidades de monitoreo
3. Asigna permisos específicos a usuarios

### Auditoría

1. Revisa los eventos de seguridad para detectar accesos no autorizados
2. Configura alertas para intentos de inicio de sesión fallidos en el dashboard
3. Supervisa cambios en la configuración del sistema

## Conclusiones

El dashboard de Wazuh proporciona una interfaz poderosa y completa para monitorear endpoints Windows y Linux. Con Blue Sentinel, tienes reglas preconfiguradas que facilitan la detección temprana de eventos de seguridad importantes en tus sistemas Windows (monitoreo de carpetas "Documentos") y servidores Linux con WordPress (detección de intentos de inicio de sesión fallidos).

Sigue esta guía para asegurar una correcta configuración, monitorización y mantenimiento de tu infraestructura de seguridad.