# Blue Sentinel: Pre-configured Wazuh SIEM Stack

**Blue Sentinel** es una solución de ciberseguridad de código abierto basada en Wazuh SIEM, lista para usar. Incluye reglas de detección personalizadas pre-configuradas para monitoreo de eventos de seguridad (FIM) en endpoints Windows y detección de intrusiones (IDS) en servidores Linux con WordPress.

## 1. DESCRIPCIÓN GENERAL

**Nombre del Proyecto:** Blue Sentinel  
**Propósito:** Solución SIEM de código abierto basada en Wazuh, lista para usar con reglas de detección personalizadas pre-configuradas.  
**Stack Tecnológico:** Wazuh Manager, Wazuh Indexer, Wazuh Dashboard (todas versiones 4.7.5)

## 2. INSTALACIÓN Y CONFIGURACIÓN

### 2.1. Requisitos Previos

- Docker Engine v20.10 o superior
- Docker Compose v2.10 o superior
- Sistema operativo: Linux, Windows o macOS

### 2.2. Pasos de Instalación

1. Clona este repositorio:
   ```bash
   git clone https://github.com/wireslab/bluesentinel.git
   cd bluesentinel
   ```

2. Arranca el stack con Docker Compose:
   ```bash
   docker-compose up -d
   ```

3. Accede al Dashboard de Wazuh en `https://localhost:443` usando las credenciales por defecto:
   - Usuario: admin
   - Contraseña: SecretPassword

### 2.3. Configuración Inicial

1. El dashboard estará disponible después de 2-3 minutos de haber iniciado el stack.
2. Registra tus agentes Wazuh siguiendo la documentación oficial de Wazuh.
3. Asegúrate de que los agentes estén correctamente conectados antes de aplicar las reglas personalizadas.

## 3. ARQUITECTURA DEL SISTEMA

### 3.1. Componentes del Stack

- **Wazuh Indexer (v4.7.5):** Almacena y analiza los datos de seguridad.
- **Wazuh Manager (v4.7.5):** Recopila, analiza y correlaciona datos de seguridad de los agentes.
- **Wazuh Dashboard (v4.7.5):** Interfaz web para visualizar y analizar los datos de seguridad.

### 3.2. Persistencia de Datos

- Todos los componentes utilizan volúmenes de Docker para persistir datos críticos entre reinicios.

### 3.3. Configuración de Red (Puertos Expuestos)

- 443 (TCP): Acceso al Dashboard (HTTPS).
- 1514 (TCP): Recepción de logs de los Agentes.
- 1515 (TCP): Registro de nuevos Agentes.
- 55000 (TCP): API de Wazuh.

## 4. FUNCIONALIDAD PRE-CONFIGURADA (VALOR AGREGADO)

Este es el núcleo de la innovación de "Blue Sentinel". La solución incluye reglas personalizadas (custom-rules) montadas automáticamente en el Wazuh Manager.

### 4.1. Regla: (FIM - Windows "Documentos")

- Fichero: custom-rules/001-windows-docs_rules.xml
- Propósito: Monitorear la carpeta C:\\Users\\*\\Documentos en endpoints Windows.
- Nivel de Alerta: 10 (Alta Prioridad).

Código de la Regla:
```xml
<group name="sysmon,">

  <rule id="100001" level="10">
    <if_sid>6</if_sid>
    <field name="win.eventdata.TargetFilename">C:\\Users\\.*\\Documents</field>
    <description>High Priority: Modification detected in user's Documents folder - Windows FIM</description>
    <options>no_email_alert</options>
    <group>fim,pci_dss_11.5,gpg13_4.13,gdpr_II_5.1.f,nist_800_53_SI.7,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>

  <rule id="100002" level="10">
    <if_sid>2</if_sid>
    <field name="win.eventdata.TargetFilename">C:\\Users\\.*\\Documents</field>
    <description>High Priority: File creation detected in user's Documents folder - Windows FIM</description>
    <options>no_email_alert</options>
    <group>fim,pci_dss_11.5,gpg13_4.13,gdpr_II_5.1.f,nist_800_53_SI.7,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>

  <rule id="100003" level="10">
    <if_sid>2</if_sid>
    <field name="win.eventdata.TargetFilename">C:\\Users\\.*\\Documents</field>
    <field name="win.eventdata.FileOpenName">C:\\Users\\.*\\Documents</field>
    <description>High Priority: File deletion detected in user's Documents folder - Windows FIM</description>
    <options>no_email_alert</options>
    <group>fim,pci_dss_11.5,gpg13_4.13,gdpr_II_5.1.f,nist_800_53_SI.7,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>

</group>
```

### 4.2. Regla: (Log - Linux/Docker WordPress)

- Fichero: custom-rules/002-linux-wordpress_rules.xml
- Propósito: Monitorear logs de un contenedor WordPress en un host Linux para detectar intentos de login fallidos (wp-login-failed).
- Nivel de Alerta: 12 (Crítica).

Código de la Regla:
```xml
<group name="web,wordpress,">

  <rule id="200001" level="12">
    <if_sid>31100</if_sid>
    <field name="location">/wp-login.php</field>
    <field name="method">POST</field>
    <field name="status">401</field>
    <description>Critical: WordPress login attempt failed</description>
    <group>pci_dss_10.2.4,pci_dss_10.2.5,pci_dss_11.4,gdpr_IV_35.7.d,gdpr_IV_32.2,nist_800_53_AU.14,nist_800_53_SI.4,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>

  <rule id="200002" level="12">
    <if_sid>31100</if_sid>
    <field name="url">.*wp-login\.php.*login.*</field>
    <field name="method">POST</field>
    <description>Critical: WordPress authentication failed</description>
    <group>pci_dss_10.2.4,pci_dss_10.2.5,pci_dss_11.4,gdpr_IV_35.7.d,gdpr_IV_32.2,nist_800_53_AU.14,nist_800_53_SI.4,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>

  <rule id="200003" level="12">
    <if_sid>31100</if_sid>
    <field name="url">.*wp-login\.php.*</field>
    <field name="data">.*log=.*</field>
    <field name="status">401,403</field>
    <description>Critical: Unauthorized access attempt to WordPress login</description>
    <group>pci_dss_10.2.4,pci_dss_10.2.5,pci_dss_11.4,gdpr_IV_35.7.d,gdpr_IV_32.2,nist_800_53_AU.14,nist_800_53_SI.4,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>

</group>
```

Nota de Implementación del Agente: Para que esta regla funcione, el agente Wazuh en el host Linux debe estar configurado (en ossec.conf) para monitorear el log relevante, ej:
```xml
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/access.log</location>
</localfile>
```

## 5. ESPECIFICACIONES TÉCNICAS

### 5.1. Requerimientos del Servidor (Host)

- RAM: 8GB mínimo, 16GB recomendado
- CPU: 4 vCPUs mínimo, 8 vCPUs recomendado
- Disco: 50GB mínimo para datos de Wazuh, más espacio recomendado para logs históricos

### 5.2. Requerimientos del Agente (Endpoint)

- Agente Wazuh v4.7.5 o compatible
- Sistemas Soportados: Windows 10+, Windows Server 2016+, Ubuntu, CentOS, Debian, etc.

### 5.3. Manifiesto de Archivos (Contenido del Paquete)

```
bluesentinel/
|-- docker-compose.yml
|-- custom-rules/
|   |-- 001-windows-docs_rules.xml
|   |-- 002-linux-wordpress_rules.xml
|-- README.md
```

## 6. MANTENIMIENTO Y CICLO DE VIDA

### 6.1. Proceso de Actualización

1. Detén el stack actual:
   ```bash
   docker-compose down
   ```

2. Actualiza las imágenes de Wazuh:
   ```bash
   docker-compose pull
   ```

3. Arranca el stack actualizado:
   ```bash
   docker-compose up -d
   ```

### 6.2. Estrategia de Backup

1. Realiza backup de los volúmenes persistentes de Docker:
   - wazuh-indexer-data: Contiene los datos analizados por Wazuh
   - wazuh-manager-etc: Contiene la configuración del manager
   - wazuh-manager-data, wazuh-manager-logs, wazuh-manager-queue: Contienen datos del manager

2. Puedes usar volúmenes externos o scripts de backup para exportar estos datos regularmente.

### 6.3. Limitaciones Conocidas

- La configuración del agente (Endpoint) para leer los logs de WordPress (Regla 4.2) debe realizarse manualmente en cada host Linux.
- El monitoreo de FIM (Regla 4.1) puede generar alto "ruido" si la carpeta "Documentos" es usada intensivamente. Se recomienda ajustar la configuración en ossec.conf del agente si es necesario.
