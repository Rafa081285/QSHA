# ANEXOS DE SEGURIDAD Y OPERACIÓN PARA DR

**Cliente:** [Completar]  
**Proyecto:** Plan de Disaster Recovery en Microsoft Azure  
**Fecha:** 2026-05-12  
**Versión:** 1.0  
**Estado:** Borrador para revisión

---

## Control documental

| Campo | Valor |
|---|---|
| Documento | Anexos de Seguridad y Operación para DR |
| Versión | 1.0 |
| Fecha | 2026-05-12 |
| Autor | Copilot |
| Revisado por | [Completar] |
| Aprobado por | [Completar] |
| Clasificación | Interno |

## Índice

1. Objeto  
2. Matriz de accesos por modo operativo  
3. Inventario de identidades de servicio  
4. Inventario de secretos y certificados críticos  
5. Matriz RTO/RPO por componente  
6. Backlog de validación Microsoft  
7. Evidencias requeridas en simulacros DR  

## 1. Objeto

Este documento complementa el Plan Director, el Plan Técnico y el Runbook de DR, centralizando anexos operativos y de seguridad necesarios para la validación y explotación del modelo de continuidad.

## 2. Matriz de accesos por modo operativo

| Rol | Operación normal | Contingencia | Desastre | Observaciones |
|---|---|---|---|---|
| Incident Commander | Aprobación y coordinación | Sí | Sí | Sin acceso técnico directo salvo excepción aprobada |
| Responsable Azure Platform | Sí | Sí | Sí | Acceso privilegiado sujeto a MFA/PIM |
| Responsable Red y Seguridad | Sí | Sí | Sí | Acceso a red, DNS, firewall y validaciones |
| Responsable Base de Datos | Sí | Sí | Sí | Acceso restringido a operaciones SQL |
| Responsable AKS / Aplicación | Sí | Sí | Sí | Acceso a operación del clúster y despliegues |
| Responsable Databricks / Datos | Sí | Sí | Sí | Acceso a jobs y validaciones de datos |
| Responsable de Negocio | Limitado | Limitado | Limitado | Validación funcional y decisión operativa |
| Auditor / Seguridad | Lectura | Lectura | Lectura | Acceso a evidencias y trazas |

### 2.1 Reglas de control
- Todo acceso privilegiado debe estar protegido con MFA.
- Toda elevación deberá quedar registrada.
- No se permiten cuentas genéricas como mecanismo ordinario de contingencia.
- Las excepciones deberán aprobarse y documentarse.

## 3. Inventario de identidades de servicio

| Componente | Tipo de identidad | Alcance | Recurso asociado | Estado | Observaciones |
|---|---|---|---|---|---|
| AKS control plane | Managed Identity | Suscripción / RG | AKS primario | Pendiente de completar | Validar permisos reales |
| AKS DR | Managed Identity | Suscripción / RG | AKS Spain Central | Pendiente de completar | Validar permisos reales |
| Databricks access connector | Managed Identity | RG / Workspace | Databricks DR | Pendiente de completar | Confirmar uso efectivo |
| Aplicación crítica 1 | UAMI / SAMI | Por componente | [Completar] | Pendiente de completar | Evitar credenciales compartidas |
| Job analítico crítico | UAMI / SPN | Por job | [Completar] | Pendiente de completar | Revisar segregación |

## 4. Inventario de secretos y certificados críticos

| Elemento | Tipo | Ubicación | Consumidor | Rotación | Criticidad | Observaciones |
|---|---|---|---|---|---|---|
| Certificado de publicación | Certificado TLS | Key Vault | Ingress / AppGW / Front Door | [Completar] | Alta | Validar vigencia en DR |
| Secreto de integración 1 | Secreto | Key Vault | Aplicación | [Completar] | Alta | Confirmar sincronización |
| Clave de cifrado 1 | Key / CMK | Key Vault | Storage / Disk / SQL | [Completar] | Alta | Revisar soporte por servicio |
| Credencial analítica | Secreto / Identity | Key Vault / MI | Databricks | [Completar] | Media | Preferir MI |

### 4.1 Reglas mínimas
- Todo secreto crítico debe estar inventariado.
- Debe existir owner por secreto/certificado.
- Debe definirse política de rotación.
- Debe existir validación en DR.

## 5. Matriz RTO/RPO por componente

| Componente | RTO objetivo | RPO objetivo | Estrategia | Prioridad |
|---|---|---|---|---|
| Azure SQL | 5 a 30 min | 5 a 30 min | Geo-réplica / FOG | Alta |
| AKS | 30 a 90 min | N/A o según persistencia | Clúster DR y despliegue | Alta |
| Storage / Data Lake | Según servicio | Según replicación | Redundancia geográfica | Alta |
| Databricks | 30 a 120 min | Según datasets / repositorios | Workspace DR | Media/Alta |
| DNS / publicación | 15 a 60 min | N/A | Conmutación DNS / Front Door / AppGW | Alta |
| Key Vault | 15 a 60 min | N/A | Vault DR / estrategia equivalente | Alta |
| Monitorización | 15 a 60 min | Baja pérdida admisible | Continuidad operativa | Media |

## 6. Backlog de validación Microsoft

| Elemento | Estado | Acción pendiente |
|---|---|---|
| Azure SQL DR | Parcialmente validado | Confirmar diseño final contra Microsoft Learn |
| Private Endpoints / DNS privado | Parcialmente validado | Revisar diseño de DR por servicio |
| Key Vault / CMK / Managed Identity | Parcialmente validado | Confirmar cobertura exacta por servicio |
| Databricks DR | Pendiente | Contrastar contra documentación oficial específica |
| Publicación / Front Door / AppGW | Pendiente | Validar patrón final |
| Storage geo-redundancy | Pendiente | Confirmar comportamiento exacto por servicio |
| Logging inmutable | Pendiente | Contrastar mecanismo final soportado |

## 7. Evidencias requeridas en simulacros DR

Cada simulacro deberá producir como mínimo:
- acta de inicio y fin,
- responsables participantes,
- cronología de acciones,
- tiempos reales por fase,
- evidencias de failover,
- evidencias de acceso con MFA,
- evidencias de lectura de secretos desde Key Vault,
- evidencias de centralización de logs,
- incidencias detectadas,
- plan de remediación.
