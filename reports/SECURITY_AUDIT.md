# 🛡️ Informe de Seguridad con Nuclei - LDAP / Keycloak OAuth2 Lab

Este informe recopila los resultados de la auditoría de seguridad automatizada ejecutada con **ProjectDiscovery Nuclei** sobre el contenedor del servidor de identidad **Keycloak / OpenLDAP** (`http://keycloak:8080`).

---

## 📋 Resumen del Escaneo
- **Objetivo analizado:** `http://keycloak:8080` (Contenedor Keycloak 26.7.4)
- **Herramienta:** ProjectDiscovery Nuclei v3.11.1
- **Plantillas ejecutadas:** 11,144 plantillas de seguridad
- **Archivo de resultados raw:** [`reports/nuclei-results.jsonl`](./nuclei-results.jsonl)
- **Total de hallazgos:** 1 hallazgo de nivel Informativo (`info`)

---

## 🔍 Hallazgos Principales

### 1. Métodos HTTP Habilitados (OPTIONS Method)
- **Severidad:** Informativa (`info`)
- **Plantilla:** `http/miscellaneous/options-method.yaml`
- **Detalle:**
  Al consultar la raíz del servicio con el método `OPTIONS`, el servidor respondió:
  ```http
  HTTP/1.1 200 OK
  Allow: HEAD, POST, GET, OPTIONS
  ```
- **Evaluación:**
  El soporte de los métodos `GET`, `POST` y `OPTIONS` es requerido para los flujos estándar de OpenID Connect y OAuth2 (negociación CORS *pre-flight* y obtención de tokens JWT mediante `POST`). No se detectaron métodos peligrosos habilitados (como `PUT`, `DELETE` o `TRACE` en la raíz).

### 2. Estado General de Vulnerabilidades
- **CVEs Críticos / RCE:** 0 detectados. La versión utilizada (`quay.io/keycloak/keycloak:26.7.4`) no presenta vulnerabilidades públicas activas catalogadas en las plantillas comunitarias de Nuclei.
- **Configuración de Seguridad:** La integración con OpenLDAP y PostgreSQL opera de forma aislada dentro de la red privada de Docker (`auth-network`).

---

## 💡 Recomendaciones de Hardening en Producción

1. **Deshabilitar modo desarrollo (`start-dev`)**:
   Actualmente el contenedor arranca con `start-dev --import-realm`. Para entornos productivos se recomienda utilizar `start` con certificados TLS/HTTPS obligatorios.

2. **Restricción de Acceso a la Consola de Administración**:
   Bloquear el acceso público al endpoint `/admin/` a través de reglas de firewall o proxy inverso, limitando el acceso únicamente a IPs de administración autorizadas.

3. **Políticas de Complejidad de Contraseñas y Brute-Force Detection**:
   Habilitar en el Realm `cybersecurity` la detección de fuerza bruta (*Brute Force Detection*) en la configuración de seguridad de Keycloak para bloquear cuentas tras intentos fallidos repetidos.
