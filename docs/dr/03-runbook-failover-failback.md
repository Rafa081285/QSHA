# RUNBOOK OPERATIVO DE FAILOVER Y FAILBACK

**Cliente:** [Completar]  
**Proyecto:** Plan de Disaster Recovery en Microsoft Azure  
**Modalidad:** Warm-Standby **(MSFT validated)**  
**Región secundaria de contingencia:** Spain Central  
**Fecha:** 2026-05-12  
**Versión:** 1.0  
**Estado:** Borrador para revisión

---

## Control documental

| Campo | Valor |
|---|---|
| Documento | Runbook Operativo de Failover y Failback |
| Versión | 1.0 |
| Fecha | 2026-05-12 |
| Autor | Copilot |
| Revisado por | [Completar] |
| Aprobado por | [Completar] |
| Clasificación | Interno |

## Historial de versiones

| Versión | Fecha | Descripción | Autor |
|---|---|---|---|
| 1.0 | 2026-05-12 | Versión inicial | Copilot |

## Índice

1. Propósito  
2. Alcance operativo  
3. Roles y responsables  
4. Condiciones de activación  
5. Prerrequisitos  
6. Checklist de preparación permanente  
7. Procedimiento de failover  
8. Criterios Go / No-Go  
9. Validaciones post-failover  
10. Procedimiento de failback  
11. Actividades posteriores al incidente  
12. Plan de pruebas recomendado  
13. Evidencias a conservar  
14. Conclusión  

## 1. Propósito

Este documento define el procedimiento operativo para ejecutar:
- el failover de la plataforma hacia **Spain Central**,
- la estabilización del servicio en el entorno DR,
- el posterior failback hacia la región primaria.

Está diseñado para ser utilizado en:
- simulacros,
- pruebas controladas,
- incidentes reales.

## 2. Alcance Operativo

Aplica a:
- red y conectividad,
- publicación,
- Azure SQL **(MSFT validated)**,
- Storage,
- AKS,
- Databricks,
- Key Vault y secretos **(MSFT validated)**,
- monitorización,
- validaciones funcionales.

## 3. Roles y Responsables

| Rol | Responsabilidad |
|---|---|
| Incident Commander | coordinación general y decisiones |
| Responsable Azure Platform | infraestructura y plataforma |
| Responsable Red y Seguridad | DNS, routing, firewall, conectividad |
| Responsable Base de Datos | failover y validación SQL |
| Responsable AKS / Aplicación | escalado y pruebas funcionales |
| Responsable Databricks / Datos | activación de procesos y validación |
| Responsable de Negocio | validación operativa mínima |
| PM / Service Manager | seguimiento y comunicaciones |

## 4. Condiciones de Activación

Se considerará la activación del DR cuando exista:
- caída total o severa de la región primaria,
- indisponibilidad incompatible con RTO,
- fallo grave de servicios críticos,
- incidente de seguridad que obligue a aislamiento,
- imposibilidad de recuperación local en tiempo razonable.

La decisión deberá quedar formalmente registrada.

## 5. Prerrequisitos

Antes de un incidente o simulacro debe existir:
- runbooks aprobados,
- accesos administrativos vigentes,
- cuentas break-glass disponibles,
- contactos actualizados,
- monitorización operativa,
- pipelines listos,
- secretos y certificados vigentes,
- inventario y dependencias actualizados.

## 6. Checklist de Preparación Permanente

### 6.1 Plataforma
- [ ] Red DR desplegada
- [ ] Routing validado
- [ ] DNS privado operativo **(MSFT validated)**
- [ ] Private Endpoints operativos **(MSFT validated)**
- [ ] Key Vault DR disponible **(MSFT validated)**
- [ ] Monitorización operativa

### 6.2 Datos
- [ ] Replicación SQL activa **(MSFT validated)**
- [ ] Backups vigentes **(MSFT validated)**
- [ ] Restore tests ejecutados
- [ ] Storage DR validado

### 6.3 Aplicación
- [ ] AKS DR desplegado
- [ ] Imágenes disponibles
- [ ] Despliegues automatizados
- [ ] Secrets externalizados

### 6.4 Analítica
- [ ] Databricks DR desplegado
- [ ] Jobs versionados
- [ ] Accesos a datos validados

## 7. Procedimiento de Failover

## 7.1 Fase A — Declaración y control inicial

### 7.1.1 Apertura del incidente
- registrar incidente,
- registrar hora T0,
- identificar alcance preliminar,
- convocar comité de crisis.

### 7.1.2 Decisión inicial
Determinar si aplica:
- recuperación local,
- degradación controlada,
- o activación formal del DR.

### 7.1.3 Freeze de cambios
- detener despliegues no esenciales,
- detener cambios de red no autorizados,
- pausar mantenimientos,
- informar a los equipos.

## 7.2 Fase B — Evaluación técnica inicial

### 7.2.1 Confirmación de impacto
Validar:
- estado de la región primaria,
- conectividad,
- publicación,
- servicios de datos,
- capacidad administrativa.

### 7.2.2 Confirmación de estrategia
Definir si el failover será:
- total,
- parcial,
- o por componente.

### 7.2.3 Comunicación inicial
Informar a:
- responsables técnicos,
- negocio,
- operación,
- gestores del servicio.

## 7.3 Fase C — Recuperación de datos

### 7.3.1 Validación de replicación SQL
Revisar:
- estado de réplica,
- latencia,
- consistencia,
- posibilidad de promoción.

### 7.3.2 Decisión sobre base de datos
Elegir entre:
- failover de réplica,
- failover group,
- restore desde backup si aplica.

### 7.3.3 Ejecución de failover SQL
- promover secundario en Spain Central, **(MSFT validated)**
- verificar endpoint/listener,
- validar conectividad desde DR.

### 7.3.4 Validación de base de datos
- prueba de disponibilidad,
- lectura,
- escritura,
- usuarios y roles,
- logs básicos.

## 7.4 Fase D — Servicios base y seguridad

### 7.4.1 Red DR
Validar:
- Virtual WAN,
- subredes,
- rutas,
- NSG/UDR,
- conectividad entre componentes.

### 7.4.2 DNS privado
Validar:
- zonas,
- links,
- resolución de SQL,
- resolución de Storage,
- resolución de Key Vault.

### 7.4.3 Private Endpoints
Confirmar conectividad a:
- SQL,
- Storage,
- Key Vault,
- otros servicios críticos.

### 7.4.4 Key Vault y secretos
Verificar:
- acceso desde AKS,
- acceso desde Databricks,
- certificados,
- claves de cifrado.

## 7.5 Fase E — Activación de AKS

### 7.5.1 Estado del clúster DR
Verificar:
- control plane,
- node pools,
- salida a red,
- acceso a ACR,
- acceso a monitorización.

### 7.5.2 Escalado de capacidad
- escalar node pools,
- habilitar pools necesarios,
- confirmar capacidad.

### 7.5.3 Activación de workloads
- desplegar o promover workloads críticos,
- validar namespaces,
- validar ingress,
- validar secretos,
- revisar readiness y liveness.

### 7.5.4 Validación técnica
- pods healthy,
- acceso a SQL,
- acceso a Storage,
- acceso a Key Vault,
- conectividad saliente.

### 7.5.5 Smoke tests
- login,
- navegación,
- operaciones principales,
- escritura,
- revisión de logs.

## 7.6 Fase F — Activación de Databricks

### 7.6.1 Validación de workspace
- acceso al workspace,
- conectividad privada,
- acceso a secretos.

### 7.6.2 Activación de capacidad
- iniciar clusters,
- habilitar pools si aplica,
- validar configuración.

### 7.6.3 Validación de accesos
- SQL DR,
- Storage,
- catálogo/metastore,
- credenciales.

### 7.6.4 Activación de jobs
- ejecutar jobs prioritarios,
- validar resultados,
- registrar incidencias.

## 7.7 Fase G — Publicación y apertura de servicio

### 7.7.1 Activación de publicación
Según diseño:
- actualizar Front Door,
- o Traffic Manager,
- o DNS,
- o backends de App Gateway.

### 7.7.2 Validación de acceso externo
Comprobar:
- resolución,
- TLS,
- reachability,
- latencia,
- acceso desde Internet y red corporativa.

### 7.7.3 Apertura controlada
- habilitar acceso progresivo,
- monitorizar errores,
- validar comportamiento.

## 7.8 Fase H — Estabilización

### 7.8.1 Monitorización reforzada
Supervisar:
- errores 4xx/5xx,
- CPU/memoria,
- latencia,
- estado de base de datos,
- jobs,
- alertas de seguridad.

### 7.8.2 Ajuste de capacidad
- escalar AKS si procede,
- ampliar capacidad analítica si procede,
- ajustar publicación si procede.

### 7.8.3 Comunicación
Emitir comunicación de estado:
- servicio restaurado,
- limitaciones,
- próximos pasos.

## 8. Criterios Go / No-Go

### 8.1 Go
- SQL consistente y operativo,
- red y DNS validados,
- secretos disponibles,
- aplicación crítica funcional,
- publicación validada,
- observabilidad activa.

### 8.2 No-Go
- riesgo de corrupción de datos,
- inaccesibilidad a secretos o claves,
- conectividad privada incompleta,
- pruebas críticas fallidas,
- riesgo no asumible para negocio.

## 9. Validaciones Post-Failover

Registrar:
- [ ] acceso de usuario
- [ ] autenticación y autorización
- [ ] operaciones de lectura
- [ ] operaciones de escritura
- [ ] integraciones externas
- [ ] jobs prioritarios
- [ ] monitorización y alertas
- [ ] trazabilidad y logging
- [ ] rendimiento aceptable

## 10. Procedimiento de Failback

## 10.1 Fase I — Preparación del retorno

### 10.1.1 Confirmación de recuperación primaria
Verificar:
- estabilidad,
- conectividad,
- datos,
- aplicación,
- publicación.

### 10.1.2 Análisis de brecha
Identificar:
- cambios durante DR,
- datos generados,
- diferencias de configuración,
- riesgos del retorno.

### 10.1.3 Aprobación de ventana
- definir ventana,
- informar a negocio,
- congelar cambios.

## 10.2 Fase J — Resincronización

### 10.2.1 Reconfiguración de replicación
- restablecer dirección de replicación,
- verificar consistencia,
- confirmar punto seguro.

### 10.2.2 Preparación de primaria
- validar Key Vault,
- validar DNS,
- validar red,
- validar AKS,
- validar Databricks,
- validar publicación.

### 10.2.3 Ensayo previo
- ejecutar validaciones técnicas,
- decidir Go / No-Go.

## 10.3 Fase K — Ejecución del retorno

### 10.3.1 Redirigir publicación
- conmutar tráfico a primaria,
- validar resolución,
- comprobar acceso.

### 10.3.2 Validación funcional
- repetir smoke tests,
- validar datos,
- validar integraciones.

### 10.3.3 Restablecer DR
- reducir capacidad sobrante en Spain Central,
- restablecer replicación,
- verificar estado warm-standby.

### 10.3.4 Cierre
- cerrar incidente,
- dejar evidencias,
- actualizar bitácora.

## 11. Actividades Posteriores al Incidente

- informe post-mortem,
- comparación real contra RTO/RPO,
- registro de incidencias,
- backlog de mejoras,
- actualización de runbooks,
- comunicación de lecciones aprendidas.

## 12. Plan de Pruebas Recomendado

### Mensual
- validaciones parciales,
- conectividad,
- secretos,
- replicación.

### Trimestral
- failover técnico controlado,
- recuperación SQL,
- validación AKS y DNS.

### Semestral
- simulacro técnico end-to-end.

### Anual
- ejercicio integral con negocio.

## 13. Evidencias a Conservar

Cada prueba o incidente deberá conservar:
- hora de inicio y fin,
- decisiones tomadas,
- tiempos por fase,
- resultados,
- incidencias detectadas,
- acciones correctivas,
- estado final.

## 14. Conclusión

Este runbook proporciona una base operativa para ejecutar la recuperación hacia **Spain Central** de forma controlada, repetible y auditable.  
Su eficacia dependerá de la automatización, la consistencia de los datos, la disponibilidad de secretos y la ejecución periódica de pruebas.
