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

## 💡 Medidas de Hardening Implementadas

1. **Protección contra Fuerza Bruta (`Brute Force Detection`)**:
   - Se activó `bruteForceProtected: true` en el realm `cybersecurity`.
   - Se configuró bloqueo automático temporal de cuentas tras 5 intentos fallidos (`failureFactor: 5`) con un tiempo de espera progresivo para evitar ataques de diccionario o credential stuffing.

2. **Evaluación del Método OPTIONS**:
   - Se confirmó que los métodos soportados (`HEAD, POST, GET, OPTIONS`) corresponden al estándar OIDC/OAuth2 para el flujo de autenticación e intercambio de tokens JWT, sin exponer métodos inseguros (`PUT`, `DELETE`, `TRACE`).

3. **Aislamiento en Redes Docker**:
   - La comunicación entre Keycloak, Postgres y OpenLDAP se mantiene encapsulada dentro de la red interna `auth-network`.
