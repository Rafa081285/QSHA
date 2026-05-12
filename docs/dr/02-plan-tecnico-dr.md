# PLAN TÉCNICO DETALLADO DE DISASTER RECOVERY (DR)

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
| Documento | Plan Técnico Detallado de DR |
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

1. Objeto del documento  
2. Referencia arquitectónica  
3. Objetivos técnicos  
4. Principios técnicos de diseño  
5. Modelo de recuperación  
6. Arquitectura técnica objetivo  
7. Matriz técnica de recuperación  
8. Secuencia técnica de recuperación  
9. Actividades de implantación  
10. Criterios de aceptación técnica  
11. Dependencias críticas  
12. Recomendaciones finales  

## 1. Objeto del Documento

El presente documento describe el diseño técnico objetivo para implantar una estrategia de **Disaster Recovery** en Azure, habilitando la recuperación regional en **Spain Central** mediante un modelo **warm-standby**.

Incluye:
- arquitectura objetivo,
- estrategia por capas,
- decisiones técnicas,
- dependencias,
- secuencia de recuperación,
- criterios de aceptación.

## 2. Referencia Arquitectónica

La propuesta se basa en la arquitectura facilitada, que incluye:
- Azure Virtual WAN,
- conectividad privada,
- Application Gateway,
- hub/spoke networking,
- bastion,
- DNS privado **(MSFT validated)**,
- Key Vault **(MSFT validated)**,
- Disk Encryption Keys,
- AKS,
- Azure SQL **(MSFT validated)**,
- Azure Databricks,
- servicios de almacenamiento,
- observabilidad.

**Imagen de referencia:**

![image1](image1)

## 3. Objetivos Técnicos

1. Habilitar una región secundaria preparada para asumir la operación.
2. Garantizar recuperabilidad de datos críticos.
3. Reducir tareas manuales en contingencia.
4. Minimizar la divergencia entre primario y DR.
5. Facilitar pruebas periódicas.
6. Cumplir RTO/RPO acordados.

## 4. Principios Técnicos de Diseño

### 4.1 Infrastructure as Code
Toda infraestructura replicable deberá definirse con:
- Terraform,
- Bicep,
- o herramientas equivalentes controladas.

### 4.2 Automatización
Las capas de aplicación y analítica deberán poder desplegarse de forma repetible.

### 4.3 Recuperación por dependencias
La recuperación seguirá este orden:
1. red,
2. seguridad,
3. datos,
4. aplicación,
5. analítica,
6. publicación.

### 4.4 Configuración versionada
Todo componente crítico deberá estar:
- versionado,
- trazado,
- revisado.

### 4.5 Validación recurrente
Ningún componente DR se considerará operativo sin pruebas.

## 5. Modelo de Recuperación

### 5.1 Región primaria
Presta servicio habitual.

### 5.2 Región DR
**Spain Central** mantendrá:
- infraestructura base preaprovisionada,
- servicios de datos replicados,
- capacidad mínima para componentes críticos,
- mecanismos de publicación listos para activación.

### 5.3 Activación
La región DR asumirá operación mediante:
- failover de datos,
- activación o escalado de AKS,
- activación de Databricks,
- validación de seguridad y secretos,
- cambio de routing/publicación.

## 6. Arquitectura Técnica Objetivo

## 6.1 Red y conectividad

### Componentes requeridos en DR
- Resource Groups equivalentes
- VNet/s equivalentes
- subredes equivalentes
- integración con Azure Virtual WAN
- NSG/UDR equivalentes
- firewalling equivalente
- Bastion o acceso administrativo controlado
- Private DNS Zones **(MSFT validated)**
- DNS Private Resolver si aplica **(MSFT validated)**
- Private Endpoints para SQL, Storage, Key Vault y otros PaaS **(MSFT validated)**
- mecanismo de publicación equivalente

### Requisitos
- preaprovisionamiento completo,
- ausencia de conflictos IP,
- conectividad validada,
- resolución privada operativa,
- rutas y seguridad revisadas.

### Riesgos a revisar
- propagación de rutas,
- allowlists externas,
- DNS privado incompleto,
- dependencia de IPs fijas.

## 6.2 Seguridad, secretos e identidades

### Componentes requeridos
- Key Vault secundario,
- secretos y certificados sincronizados,
- claves de cifrado disponibles,
- RBAC equivalente,
- Managed Identities para recursos DR **(MSFT validated)**,
- logging y controles de seguridad activos.

### Requisitos
- permisos explícitos para recursos DR,
- acceso de AKS y Databricks a secretos,
- disponibilidad de certificados en publicación,
- independencia operativa del Key Vault primario.

### Riesgos
- permisos incompletos,
- secretos desalineados,
- claves no accesibles,
- dependencias manuales.

## 6.3 Datos

### 6.3.1 Azure SQL
**Estrategia recomendada:**
- geo-réplica o auto-failover group hacia Spain Central. **(MSFT validated)**

**Requisitos:**
- listener estable,
- conectividad privada desde DR,
- usuarios y permisos validados,
- backup PITR/LTR operativo **(MSFT validated)**,
- monitorización de replicación.

**Objetivo:** permitir promoción a primario en una ventana compatible con RTO.

### 6.3.2 Storage / Lake
**Estrategia recomendada:**
- redundancia geográfica o cuenta secundaria,
- accesibilidad desde AKS y Databricks DR,
- procedimiento de promoción documentado.

**Riesgos:**
- no transparencia del failover,
- endpoint privado no operativo,
- permisos inconsistentes.

### 6.3.3 Backup y restauración
- políticas de backup vigentes,
- pruebas de restauración periódicas,
- evidencias conservadas.

## 6.4 Plataforma de Aplicación — AKS

### Estrategia
Desplegar un **AKS secundario en Spain Central** con:
- configuración equivalente,
- integración con red privada,
- acceso a ACR,
- integración con Key Vault,
- observabilidad operativa,
- capacidad mínima activa o preparada.

### Elementos a replicar
- namespaces,
- RBAC,
- policies,
- ingress,
- certificados,
- Helm charts / GitOps,
- HPA/PDB/Network Policies,
- configuraciones de observabilidad.

### Riesgos
- imágenes no disponibles,
- secretos no accesibles,
- persistencia regional no protegida,
- diferencias manuales entre clústeres.

## 6.5 Analítica — Azure Databricks

### Estrategia
Desplegar un workspace secundario en Spain Central con:
- networking privado equivalente,
- acceso a SQL y Storage DR,
- notebooks/jobs versionados,
- policies y secretos preparados.

### Elementos a contemplar
- workspace,
- cluster policies,
- repos,
- jobs,
- secret scopes,
- metastore o catálogo,
- access connector / managed identity,
- external locations y credenciales.

### Riesgos
- objetos no versionados,
- secretos no sincronizados,
- dependencias de red privada,
- inconsistencia entre entornos.

## 6.6 Publicación y exposición

### Alternativas
- Azure Front Door
- Traffic Manager
- DNS failover
- App Gateway duplicado con conmutación

### Recomendación
Si la aplicación es pública y requiere conmutación simplificada, valorar **Azure Front Door**.

### Requisitos
- estrategia documentada,
- TTL conocido,
- certificados válidos,
- validación desde Internet y red corporativa.

## 7. Matriz Técnica de Recuperación

| Dominio | Componente | Estrategia DR | Estado esperado en DR | Acción de failover |
|---|---|---|---|---|
| Red | Virtual WAN / Hub-Spoke | réplica funcional | desplegado | validar rutas y conectividad |
| Red | DNS privado | duplicado y enlazado | desplegado | validar resolución |
| Red | Private Endpoints | específicos de DR | desplegado | verificar acceso |
| Seguridad | Key Vault | vault secundario | activo | validar secretos y certificados |
| Seguridad | Managed Identities | permisos explícitos | preparado | validar autorizaciones |
| Datos | Azure SQL | geo-réplica / FOG | replicando | promover secundario |
| Datos | Storage | geo-redundancia / secundario | replicando o preparado | habilitar acceso |
| Aplicación | AKS | clúster DR | mínimo | escalar workloads |
| Analítica | Databricks | workspace DR | preparado | activar jobs/clusters |
| Publicación | Front Door / DNS / AppGW | conmutación preparada | mínimo/activo | redirigir tráfico |
| Operación | Monitorización | operativa | activa | supervisión reforzada |

## 8. Secuencia Técnica de Recuperación

1. Declaración del incidente  
2. Freeze de cambios  
3. Validación de replicación  
4. Failover de Azure SQL **(MSFT validated)**  
5. Validación de Storage  
6. Verificación de Key Vault y secretos **(MSFT validated)**  
7. Escalado y activación de AKS DR  
8. Activación de Databricks DR  
9. Cambio de publicación/routing  
10. Smoke tests  
11. Apertura controlada del servicio  

## 9. Actividades de Implantación

### 9.1 Assessment
- inventario,
- dependencias,
- criticidad,
- RTO/RPO,
- gaps actuales.

### 9.2 Red
- despliegue red DR,
- integración con Virtual WAN,
- DNS privado,
- private endpoints,
- validación de conectividad.

### 9.3 Seguridad
- Key Vault DR,
- permisos,
- identidades,
- claves,
- diagnósticos.

### 9.4 Datos
- SQL réplica **(MSFT validated)**,
- backup/restore **(MSFT validated)**,
- storage redundante,
- pruebas de acceso.

### 9.5 AKS
- clúster DR,
- integración con ACR/KV/Monitor,
- despliegues,
- pruebas funcionales.

### 9.6 Databricks
- workspace DR,
- networking,
- secretos,
- jobs/notebooks,
- pruebas de datos.

### 9.7 Publicación
- diseño de conmutación,
- pruebas DNS/routing,
- simulacro integral.

## 10. Criterios de Aceptación Técnica

El entorno se considerará operativo cuando:
- la red DR esté desplegada y validada,
- el DNS privado resuelva correctamente,
- los secretos y certificados estén disponibles,
- la replicación de datos críticos funcione,
- AKS DR esté operativo,
- Databricks DR esté validado,
- la publicación en DR haya sido probada,
- el simulacro técnico se haya completado,
- RTO/RPO hayan sido medidos y aceptados.

## 11. Dependencias Críticas

- cuotas regionales en Spain Central,
- dependencias con terceros,
- allowlists de IP,
- certificados y dominios,
- persistencia de AKS,
- objetos no versionados en Databricks,
- accesos break-glass,
- dependencias externas a Azure.

## 12. Recomendaciones Finales

Se recomienda priorizar:
1. Azure SQL y Storage  
2. DNS privado y Private Endpoints  
3. Key Vault y Managed Identities  
4. AKS DR  
5. Databricks DR  
6. mecanismo de publicación  
7. automatización y simulacros  

La eficacia del DR dependerá especialmente de:
- la automatización real,
- la consistencia del dato,
- la disponibilidad de secretos,
- la validez de la conectividad privada,
- la disciplina en las pruebas.
