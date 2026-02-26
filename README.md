# Subsistema de Monitoreo - USalud
## Información

- **Autor**: Vicente Beiza Silva  
- **Contactos principales**:  
  - vicentebeizas@gmail.com  
  - fjaraw@gmail.com  
  - alonsocarvajal@uchile.cl  
  - jjaraw@gmail.com  
- **Última revisión**: 26 de febrero de 2026
- 
## Contexto
El documento guía al Administrador sobre el sistema de monitoreo. Este supervisa microservicios clínicos, infraestructura y disponibilidad de la plataforma, utilizando herramientas open-source estándar para entornos hospitalarios y de microservicios.

## Componentes previos U-Salud

Para tener una visión clara de la infraestructura base sobre la cual se implementa el monitoreo, consulta el diagrama de componentes existentes:

🔗 **[Diagrama de arquitectura – U-Salud (draw.io)](https://app.diagrams.net/#G18ppW8toIe2YG7AshoIzyJCuS8N07HdeU#%7B%22pageId%22%3A%22dFJ4SN5G_9eXKO1aYNOq%22%7D)**  
*(Haz clic para abrir el diagrama interactivo en diagrams.net)*

## Componentes/Herramientas del Sistema de Monitoreo

El subsistema de monitoreo de U-Salud se encuentra ubicado en:
/opt/usalud/u-salud-monitoring


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
