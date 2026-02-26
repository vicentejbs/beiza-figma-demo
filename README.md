# Subsistema de Monitoreo - USalud
## Información

- **Autor**: Vicente Beiza Silva  
- **Contactos principales**:  
  - vicentebeizas@gmail.com  
  - fjaraw@gmail.com  
  - alonsocarvajal@uchile.cl  
  - jjaraw@gmail.com  
- **Última revisión**: 26 de febrero de 2026

## Contexto
El documento guía al Administrador sobre el sistema de monitoreo. Este supervisa microservicios clínicos, infraestructura y disponibilidad de la plataforma, utilizando herramientas open-source estándar para entornos hospitalarios y de microservicios.

## Componentes previos U-Salud

Para tener una visión clara de la infraestructura base sobre la cual se implementa el monitoreo, consulta el diagrama de componentes existentes:

🔗 **[Diagrama de arquitectura – U-Salud (draw.io)](https://app.diagrams.net/#G18ppW8toIe2YG7AshoIzyJCuS8N07HdeU#%7B%22pageId%22%3A%22dFJ4SN5G_9eXKO1aYNOq%22%7D)**  
*(Haz clic para abrir el diagrama interactivo en diagrams.net)*

## Componentes/Herramientas del Sistema de Monitoreo

El subsistema de monitoreo de U-Salud se encuentra ubicado en:

```bash
/opt/usalud/u-salud-monitoring
```

Consulte la documentación oficial de cada herramienta para detalles de configuración y uso avanzado. Las herramientas se agrupan según su rol en la observabilidad (recolección, almacenamiento/procesamiento y visualización/respuesta).

### A. Recolección de Datos (Exporters y Agentes)
Extraen métricas y registros brutos de infraestructura y microservicios.

- **Node Exporter** — Extrae métricas de hardware (CPU, RAM, Disco, etc.).  
  [Documentación oficial](https://prometheus.io/docs/guides/node-exporter/)

- **cAdvisor** — Monitoreo de consumo de recursos en contenedores Docker.  
  [Documentación oficial](https://github.com/google/cadvisor)

- **Postgres Exporter** — Recolecta métricas específicas de PostgreSQL.  
  [Documentación oficial](https://github.com/prometheus-community/postgres_exporter)

- **Blackbox Exporter** — Sondea disponibilidad externa (HTTP, TCP, DNS, etc.).  
  [Documentación oficial](https://github.com/prometheus/blackbox_exporter)

- **Promtail** — Agente que captura y envía logs de contenedores y archivos locales a Loki.  
  [Documentación oficial](https://grafana.com/docs/loki/latest/send-data/promtail/)

### B. Almacenamiento y Procesamiento (Backend)
Motores para el procesamiento, indexación y almacenamiento histórico de métricas y logs.

- **Prometheus** — Base de datos de series temporales para métricas.  
  [Documentación oficial](https://prometheus.io/docs/introduction/overview/)

- **Loki** — Sistema de agregación y almacenamiento de logs para microservicios.  
  [Documentación oficial](https://grafana.com/docs/loki/latest/)

### C. Visualización y Respuesta (Frontend)
Interfaces de usuario y gestión de eventos críticos.

- **Grafana** — Plataforma de análisis, visualización y dashboards.  
  [Documentación oficial](https://grafana.com/docs/)

- **Alertmanager** — Clasifica, agrupa y envía notificaciones de alertas.  
  [Documentación oficial](https://prometheus.io/docs/alerting/latest/alertmanager/)

- **Docker Compose** — Orquestador para el despliegue y gestión de toda la infraestructura de monitoreo.  
  [Documentación oficial](https://docs.docker.com/compose/)

## Diagrama Operacional Básico de Monitoreo

Diagrama simplificado que muestra el flujo operativo del sistema de monitoreo U-Salud: recolección de métricas y logs, procesamiento, visualización y notificación de alertas.

<img width="6512" height="3596" alt="image" src="https://github.com/user-attachments/assets/42d3ea06-c955-4cd8-b08c-afe93cef09ef" />

**Versión interactiva de mayor calidad (recomendada):**  
[Ver diagrama completo en diagrams.net](https://app.diagrams.net/#G18ppW8toIe2YG7AshoIzyJCuS8N07HdeU#%7B%22pageId%22%3A%22dFJ4SN5G_9eXKO1aYNOq%22%7D)  
*(Haz clic para abrir, explorar y hacer zoom en el diagrama original)*

### Puertos y accesos”
```md
## Puertos y accesos

| Servicio | URL de acceso |
|---|---|
| Prometheus | `http://IP_SERVIDOR:9090` |
| Grafana | `http://IP_SERVIDOR:3010` |
| Alertmanager | `http://IP_SERVIDOR:9093` |
| Blackbox Exporter | `http://IP_SERVIDOR:9115` |
| cAdvisor | `http://IP_SERVIDOR:8082` |
| Loki | `http://IP_SERVIDOR:3100` |
```
> **Nota:** Node Exporter expone métricas en `http://IP_SERVIDOR:9100/metrics` (modo host).

## Instalación e Implementación del Sistema

Esta sección detalla el procedimiento de despliegue del subsistema de monitoreo en el servidor U-Salud.  El objetivo es permitir a futuros administradores reconstruir el entorno de forma controlada y reproducible.

### Requisitos previos

#### Requerimientos de Software
- Docker Engine y Docker Compose instalados (para la gestión de contenedores y aplicaciones multi-contenedor).
- Conectividad saliente a Internet (necesaria para notificaciones vía Slack).
- Acceso a los puertos de los microservicios clínicos de U-Salud.

#### Requisitos del Servidor
- Sistema operativo: Servidor Linux (recomendado **Ubuntu Server 22.04 LTS** o distribución compatible).
- Acceso con privilegios **sudo**.
- Versiones recomendadas:
  - Docker Engine ≥ **v29.1.3**
  - Docker Compose ≥ **v2.5** (o v5 si se refiere a la versión standalone)
- Conectividad de red hacia:
  - Microservicios U-Salud
  - Instancia PostgreSQL
  - Servicio de notificaciones Slack

### Estructura de directorios del subsistema

El subsistema de monitoreo está organizado bajo la siguiente estructura en el servidor:

```bash
/opt/usalud/u-salud-monitoring
├── compose.yml                # Orquestación principal (Docker Compose)
├── alertmanager/
│   └── alertmanager.yml       # Configuración de alertas y rutas de notificación
├── backups/                   # Respaldo manual de configuraciones (opcional)
├── blackbox/
│   └── blackbox.yml           # Configuración de probes HTTP/TCP/ICMP
├── grafana/
│   ├── compose.yml            # Compose específico (si aplica) o parte del principal
│   ├── data/                  # Persistencia (SQLite, plugins instalados, etc.)
│   └── provisioning/          # DataSources, dashboards y alertas preconfigurados
├── loki/
│   └── loki-config.yml        # Configuración del motor de logs
├── prometheus/
│   ├── data/                  # Persistencia de la base TSDB (métricas históricas)
│   ├── prometheus.yml         # Configuración principal de Prometheus
│   └── rules.yml              # Reglas de alerta y reglas de grabación (recording rules)
└── promtail/
    └── promtail.yml           # Configuración del agente de captura y envío de logs
```

> **Nota importante**  
> No es necesario instalar manualmente Prometheus, Grafana, Loki, Alertmanager ni los exporters en el sistema operativo host.  
> Todas las herramientas se despliegan automáticamente como contenedores mediante el archivo maestro de orquestación ubicado en: /opt/usalud/u-salud-monitoring/compose.yml


Este archivo `compose.yml` define y levanta los siguientes componentes:

- **Exporters** → Node Exporter, cAdvisor, Postgres Exporter, Blackbox Exporter
- **Backend** → Prometheus (métricas), Loki (logs)
- **Frontend** → Grafana (visualización y dashboards)
- **Sistema de alertas** → Alertmanager
- **Agente de logs** → Promtail

## Implementación de cada componente

A continuación se describe de forma resumida cómo se implementó cada herramienta en el stack de monitoreo U-Salud.

- **Node Exporter**  
  Rol: Métricas de hardware y sistema operativo (CPU, RAM, disco, red, filesystems).  
  Modo: `network_mode: host` (accede directamente al host sin NAT).  
  **Debido al modo host, las métricas se consultan vía IP del servidor y puerto 9100** (`http://IP_SERVIDOR:9100/metrics`).
  Puerto: **9100**  
  Archivo: `compose.yml`  
  Docs: [https://github.com/prometheus/node_exporter](https://github.com/prometheus/node_exporter)

- **cAdvisor**  
  Rol: Consumo de recursos por contenedor Docker (CPU, memoria, red, FS).  
  Privilegios: Elevados (acceso a cgroups y kernel del host).  
  Puerto: **8082** (host) → 8080 (contenedor)  
  Archivo: `compose.yml`  
  Docs: [https://github.com/google/cadvisor](https://github.com/google/cadvisor)

- **Postgres Exporter**  
  Rol: Métricas internas de PostgreSQL (conexiones, locks, buffers, consultas, estadísticas).  
  Conexión: Definida vía `DATA_SOURCE_NAME=postgresql://usuario:password@IP:PUERTO/base`  
  Puerto: **9187**  
  Docs: [https://github.com/prometheus-community/postgres_exporter](https://github.com/prometheus-community/postgres_exporter)

- **Prometheus**  
  Rol: Núcleo del monitoreo – recolección, almacenamiento TSDB, evaluación de reglas y envío de alertas.  
  Archivos clave:  
  - `prometheus/prometheus.yml` (configuración principal)  
  - `prometheus/rules.yml` (reglas de alerta y recording)  
  Persistencia: `prometheus/data`  
  Puerto: **9090**  
  Docs: [https://prometheus.io/docs/](https://prometheus.io/docs/)

- **Blackbox Exporter**  
  Rol: Pruebas activas de disponibilidad (HTTP 200–404, TCP, ICMP).  
  Configuración: `blackbox/blackbox.yml`  
  Puerto: **9115**  
  Docs: [https://github.com/prometheus/blackbox_exporter](https://github.com/prometheus/blackbox_exporter)

- **Alertmanager**  
  Rol: Agrupación, deduplicación, clasificación y envío de alertas (principalmente a Slack: api_url: <SLACK_WEBHOOK>).  
  Configuración: `alertmanager/alertmanager.yml`  
  Puerto: **9093**  
  Docs: [https://prometheus.io/docs/alerting/latest/alertmanager/](https://prometheus.io/docs/alerting/latest/alertmanager/)

- **Loki**  
  Rol: Almacenamiento centralizado y eficiente de logs (optimizado para microservicios).  
  Configuración: `loki/loki-config.yml`
    > **IMPORTANTE (persistencia de logs)**  
  > En la configuración actual, Loki utiliza almacenamiento en filesystem local (por defecto bajo rutas temporales como `/tmp/loki` dentro del contenedor).  
  > Esto puede implicar pérdida de logs ante recreación del contenedor o limpieza de almacenamiento temporal.  
  > Para un entorno productivo se recomienda configurar persistencia dedicada (volumen/host path).
  > (No te obligo a cambiar el stack ahora; solo lo documentamos como corresponde.)
    
  Puerto: **3100**  
  Docs: [https://grafana.com/docs/loki/](https://grafana.com/docs/loki/)

- **Promtail**  
  Rol: Agente que captura logs de contenedores Docker y los envía a Loki.  
  Acceso: Socket `/var/run/docker.sock` (captura automática sin configuración por contenedor).  
  Configuración: `promtail/promtail.yml`  
  Puerto interno: **9080**  
  Docs: [https://grafana.com/docs/loki/latest/clients/promtail/](https://grafana.com/docs/loki/latest/clients/promtail/)

- **Grafana**  
  Rol: Interfaz principal – visualización de métricas, logs, dashboards y correlación de datos.  
  Características clave:  
  - Persistencia de dashboards y plugins  
  - Provisioning automático de DataSources (Prometheus + Loki)  
  - Acceso restringido a administradores  
  Persistencia: `grafana/data`  
  Provisioning: `grafana/provisioning`  
  Puerto: **3010**  
  Acceso: `http://IP_SERVIDOR:3010`
  Docs: [https://grafana.com/docs/](https://grafana.com/docs/)

## Despliegue del subsistema (Docker Compose)

### Levantar el stack

```bash
cd /opt/usalud/u-salud-monitoring
sudo docker compose up -d
```
### Validar estado de contenedores
```bash
sudo docker ps
```
## Verificación post-despliegue

### Contenedores esperados (docker ps)
- container-prometheus
- container-grafana
- container-alertmanager
- container-node-exporter
- container-cadvisor
- container-blackbox-exporter
- container-loki
- container-promtail
- container-postgres-exporter

### Verificación en Prometheus Targets
1. Abre: `http://IP_SERVIDOR:9090`
2. Ve a **Status → Targets**
3. Confirma que todos los targets estén **UP** (especialmente node-exporter:9100, cadvisor:8080, postgres-exporter:9187, blackbox-exporter:9115, prometheus:9090)

Si algún target está DOWN: revisa logs (`docker logs <nombre>`) y configuración en `prometheus.yml`.

## Operación diaria (Administrador)

### 1) Revisar salud general
- Entrar a Grafana: `http://IP_SERVIDOR:3010` Admin, 12345678
- Revisar dashboards de salud (CPU/RAM/DISCO del host + contenedores + disponibilidad Blackbox).

### 2) Revisar alertas
- Prometheus: `http://IP_SERVIDOR:9090` → **Alerts**
- Alertmanager: `http://IP_SERVIDOR:9093` → alertas activas, silences y agrupación
- Slack: canal `#alerts-u-salud`

### 3) Revisar logs (Loki)
- Grafana → **Explore**
- Fuente de datos: **Loki**
- Filtrar por contenedor, por ejemplo:
  - `{container="rsdue_emr-usalud-1"}`
  - `{container="rsdue_aidbox_project-devbox-1"}`
 
## Recuperación y troubleshooting básico

### Reiniciar el stack completo
```bash
cd /opt/usalud/u-salud-monitoring
sudo docker compose down
sudo docker compose up -d
```
### Reiniciar un servicio puntual
```bash
sudo docker restart container-prometheus
sudo docker restart container-grafana
```

### Ver logs de un contenedor
```bash
sudo docker logs --tail=200 -f container-prometheus
```
## Problemas típicos: Targets DOWN en Prometheus

Si un target aparece en estado **DOWN**:

- Revisar conectividad / puertos del servicio  
- Revisar si el contenedor está en ejecución (`docker ps`)  
- Revisar logs del contenedor correspondiente (`docker logs <nombre-contenedor>`)

## 8) Para que quede “cerrado”: placeholders para Dashboards y Alertas
Como todavía no pegaste dashboards ni rules.yml, agrega estos encabezados (sin inventar contenido):

## Dashboards (Grafana)

> **Pendiente de documentar:** listado de dashboards, propósito de cada uno, paneles clave y consultas principales.

## Catálogo de alertas (Prometheus → Alertmanager)

> **Pendiente de documentar:** listado de alertas definidas en `prometheus/rules.yml`, severidad, condición de disparo, impacto y pasos de mitigación.
