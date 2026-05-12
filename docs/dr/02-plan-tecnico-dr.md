# PLAN TÉCNICO DETALLADO DE DISASTER RECOVERY (DR)

**Cliente:** [Completar]  
**Proyecto:** Plan de Disaster Recovery en Microsoft Azure  
**Modalidad:** Warm-Standby **(MSFT validated)**  
**Región secundaria de contingencia:** Spain Central  
**Fecha:** 2026-05-12  
**Versión:** 1.2  
**Estado:** Borrador para revisión

---

## Control documental

| Campo | Valor |
|---|---|
| Documento | Plan Técnico Detallado de DR |
| Versión | 1.2 |
| Fecha | 2026-05-12 |
| Autor | Copilot |
| Revisado por | [Completar] |
| Aprobado por | [Completar] |
| Clasificación | Interno |

## Historial de versiones

| Versión | Fecha | Descripción | Autor |
|---|---|---|---|
| 1.0 | 2026-05-12 | Versión inicial | Copilot |
| 1.1 | 2026-05-12 | Inclusión de controles, gaps y mitigaciones de seguridad | Copilot |
| 1.2 | 2026-05-12 | Ampliación del alcance técnico por categoría y servicio | Copilot |

## Índice

1. Objeto del documento  
2. Referencia arquitectónica  
3. Objetivos técnicos  
4. Alcance técnico por categoría y servicio  
5. Principios técnicos de diseño  
6. Modelo de recuperación  
7. Arquitectura técnica objetivo  
8. Matriz técnica de recuperación  
9. Secuencia técnica de recuperación  
10. Actividades de implantación  
11. Criterios de aceptación técnica  
12. Dependencias críticas  
13. Controles y gaps de seguridad  
14. Recomendaciones finales  

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

## 4. Alcance técnico por categoría y servicio

El alcance técnico del presente plan incluye los servicios identificados en la arquitectura y define para cada uno su tratamiento esperado en el escenario de DR en Spain Central.

| Categoría | Servicio | Rol técnico | Estrategia DR | Estado esperado en DR | Validaciones clave | Riesgo principal |
|---|---|---|---|---|---|---|
| Red | Azure Virtual WAN | Conectividad troncal | réplica funcional | desplegado | rutas, reachability, propagación | rutas incompletas |
| Red | Hub/Spoke VNets | segmentación | despliegue equivalente | desplegado | subredes, NSG, UDR | diferencias de configuración |
| Red | Firewall / publicación | control perimetral | componente equivalente | activo/preparado | reglas, backend, certificados | desalineación de reglas |
| Red | DNS privado | resolución interna | duplicación de zonas y links | operativo | resolución desde DR | errores de resolución |
| Red | Private Endpoints | acceso privado PaaS | endpoints específicos de DR | operativo | conectividad a SQL/KV/Storage | endpoints no funcionales |
| Aplicación | AKS | ejecución de workloads | clúster secundario | mínimo/escalable | pods, ingress, secretos, conectividad | divergencia de clúster |
| Aplicación | Ingress / publicación | exposición de apps | backend preparado para conmutación | preparado | TLS, routing, health | publicación no alineada |
| Datos | Azure SQL | dato relacional crítico | geo-réplica / failover group | replicando | failover, conectividad, integridad | réplica no validada |
| Datos | Storage / Data Lake | almacenamiento crítico | GRS/secundario/promoción | replicando/preparado | acceso desde AKS/DBX | permisos/endpoints |
| Datos | Backup | recuperación alternativa | restore probado | operativo | PITR/LTR/restore | backup no validado |
| Analítica | Databricks Workspace | analítica y pipelines | workspace DR | preparado | acceso, secretos, networking | objetos no sincronizados |
| Analítica | Jobs / notebooks | lógica analítica | versionado y redeploy | preparado | ejecución priorizada | dependencias manuales |
| Seguridad | Key Vault | secretos y claves | vault DR / estrategia equivalente | operativo | acceso, secretos, certificados | secretos desalineados |
| Seguridad | Managed Identity | auth entre servicios | asignación equivalente | preparado | permisos por componente | permisos insuficientes |
| Operación | Monitorización | métricas, logs, alertas | continuidad operativa | activo | logs, alertas, dashboards | pérdida de trazabilidad |

### 4.1 Consideraciones de alcance
El alcance técnico incluye tanto componentes permanentemente desplegados en la región secundaria como componentes preparados para activación o escalado durante contingencia.

Se consideran dentro del alcance:
- la infraestructura base necesaria para conmutación,
- la protección y recuperabilidad del dato,
- la ejecución de cargas críticas,
- la continuidad de la publicación,
- la seguridad operativa,
- la trazabilidad y la observabilidad del entorno DR.

Quedan sujetos a validación específica en fases posteriores:
- dependencias de terceros,
- integraciones no reflejadas completamente en la arquitectura,
- restricciones regionales de capacidad,
- compatibilidades de cifrado o publicación en servicios concretos,
- y cualquier componente manual no versionado.

## 5. Principios Técnicos de Diseño

### 5.1 Infrastructure as Code
Toda infraestructura replicable deberá definirse con:
- Terraform,
- Bicep,
- o herramientas equivalentes controladas.

### 5.2 Automatización
Las capas de aplicación y analítica deberán poder desplegarse de forma repetible.

### 5.3 Recuperación por dependencias
La recuperación seguirá este orden:
1. red,
2. seguridad,
3. datos,
4. aplicación,
5. analítica,
6. publicación.

### 5.4 Configuración versionada
Todo componente crítico deberá estar:
- versionado,
- trazado,
- revisado.

### 5.5 Validación recurrente
Ningún componente DR se considerará operativo sin pruebas.

## 6. Modelo de Recuperación

### 6.1 Región primaria
Presta servicio habitual.

### 6.2 Región DR
**Spain Central** mantendrá:
- infraestructura base preaprovisionada,
- servicios de datos replicados,
- capacidad mínima para componentes críticos,
- mecanismos de publicación listos para activación.

### 6.3 Activación
La región DR asumirá operación mediante:
- failover de datos,
- activación o escalado de AKS,
- activación de Databricks,
- validación de seguridad y secretos,
- cambio de routing/publicación.

## 7. Arquitectura Técnica Objetivo

## 7.1 Red y conectividad

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

## 7.2 Seguridad, secretos e identidades

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

## 7.3 Datos

### 7.3.1 Azure SQL
**Estrategia recomendada:**
- geo-réplica o auto-failover group hacia Spain Central. **(MSFT validated)**

**Requisitos:**
- listener estable,
- conectividad privada desde DR,
- usuarios y permisos validados,
- backup PITR/LTR operativo **(MSFT validated)**,
- monitorización de replicación.

**Objetivo:** permitir promoción a primario en una ventana compatible con RTO.

### 7.3.2 Storage / Lake
**Estrategia recomendada:**
- redundancia geográfica o cuenta secundaria,
- accesibilidad desde AKS y Databricks DR,
- procedimiento de promoción documentado.

**Riesgos:**
- no transparencia del failover,
- endpoint privado no operativo,
- permisos inconsistentes.

### 7.3.3 Backup y restauración
- políticas de backup vigentes,
- pruebas de restauración periódicas,
- evidencias conservadas.

## 7.4 Plataforma de Aplicación — AKS

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

## 7.5 Analítica — Azure Databricks

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

## 7.6 Publicación y exposición

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

## 8. Matriz Técnica de Recuperación

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

## 9. Secuencia Técnica de Recuperación

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

## 10. Actividades de Implantación

### 10.1 Assessment
- inventario,
- dependencias,
- criticidad,
- RTO/RPO,
- gaps actuales.

### 10.2 Red
- despliegue red DR,
- integración con Virtual WAN,
- DNS privado,
- private endpoints,
- validación de conectividad.

### 10.3 Seguridad
- Key Vault DR,
- permisos,
- identidades,
- claves,
- diagnósticos.

### 10.4 Datos
- SQL réplica **(MSFT validated)**,
- backup/restore **(MSFT validated)**,
- storage redundante,
- pruebas de acceso.

### 10.5 AKS
- clúster DR,
- integración con ACR/KV/Monitor,
- despliegues,
- pruebas funcionales.

### 10.6 Databricks
- workspace DR,
- networking,
- secretos,
- jobs/notebooks,
- pruebas de datos.

### 10.7 Publicación
- diseño de conmutación,
- pruebas DNS/routing,
- simulacro integral.

## 11. Criterios de Aceptación Técnica

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

## 12. Dependencias Críticas

- cuotas regionales en Spain Central,
- dependencias con terceros,
- allowlists de IP,
- certificados y dominios,
- persistencia de AKS,
- objetos no versionados en Databricks,
- accesos break-glass,
- dependencias externas a Azure.

## 13. Controles y gaps de seguridad

### 13.1 Criterio técnico de evaluación

Los estados utilizados son:
- **Cumple**
- **Cumple parcialmente**
- **No evidenciado**
- **No cumple**
- **Pendiente de validación**

La valoración se realiza con base en la arquitectura aportada, la documentación generada y la evidencia disponible en esta fase. Un requisito no demostrado técnicamente no se considera cumplido.

### 13.2 Matriz detallada de cumplimiento técnico

| Requisito | Estado | Evidencia actual | Gap / riesgo técnico | Mitigación técnica propuesta | Prioridad |
|---|---|---|---|---|---|
| mTLS o comunicaciones punto a punto dedicadas cifradas en todos los canales de replicación entre CPDs | No evidenciado | No se observa en la arquitectura evidencia explícita de mTLS extremo a extremo; Virtual WAN y conectividad no demuestran por sí solas cumplimiento criptográfico de todos los flujos | Riesgo de incumplimiento del requisito de cifrado extremo a extremo; MPLS sin cifrado no sería aceptable | Inventariar todos los flujos de replicación; exigir TLS/mTLS por canal cuando el servicio lo soporte; documentar túneles cifrados o controles compensatorios por flujo | Alta |
| Prohibición de MPLS sin cifrado | No evidenciado | No se documenta el medio real entre CPDs ni su cifrado | Riesgo de uso de red dedicada sin cifrado suficiente | Declarar explícitamente en diseño que MPLS sin cifrado queda excluido; añadir requisito de cifrado en tránsito en todos los enlaces intersite | Alta |
| Identidades de servicio únicas por componente | Cumple parcialmente | Presencia de Managed Identity en arquitectura y documentos | No se demuestra unicidad por workload, clúster, job o servicio; posible reutilización de credenciales | Implantar una identidad gestionada por componente crítico y eliminar credenciales compartidas | Alta |
| MFA para consolas de gestión de clústeres y orquestación | No evidenciado | No hay evidencia documental de Conditional Access, MFA ni PIM | Riesgo de acceso privilegiado insuficientemente protegido | Configurar MFA obligatorio con Microsoft Entra ID, Conditional Access, PIM y acceso just-in-time | Alta |
| Matriz de accesos en modo normal, contingencia y desastre | No cumple | No existía una matriz formal en la versión previa del plan | Riesgo operativo y de segregación de funciones durante DR | Crear matriz RACI/accesos por modo operativo y anexarla al plan técnico y runbook | Alta |
| Secretos centralizados en bóveda de claves con generación segura y rotación | Cumple parcialmente | Uso de Key Vault en arquitectura **(MSFT validated)** | No se demuestra cobertura total ni política de rotación formal | Consolidar secretos en Azure Key Vault, definir ownership, rotación, expiración y auditoría | Alta |
| Control de versiones y acceso auditado para runbooks/playbooks | Cumple parcialmente | Los documentos están en GitHub y versionados | Falta evidencia de branch protection, revisión obligatoria y auditoría formal | Activar protección de ramas, PR obligatoria, CODEOWNERS y trazabilidad de cambios | Media |
| Hardening documentado de imágenes base en nodos HA | No evidenciado | No se aporta baseline de hardening ni referencia a imágenes golden | Riesgo de nodos DR con baseline inconsistente respecto a producción | Definir baseline CIS/corporativa, pipeline de hardening, escaneo y evidencias por imagen | Alta |
| Escenarios de pruebas para cifrado, logs y controles de acceso ante conmutación | No cumple | El runbook no detalla aún pruebas específicas de seguridad en failover | Riesgo de perder controles en el paso a DR sin detectarlo | Añadir casos de prueba DR de seguridad: cifrado en tránsito, accesos, auditoría y centralización de logs | Alta |
| TLS 1.3 en todos los canales de replicación de información | Pendiente de validación | No hay evidencia técnica exhaustiva por servicio y flujo | Riesgo de no cumplir el requisito si algún servicio o integración opera con otra versión o abstracción gestionada | Elaborar catálogo por flujo: origen, destino, protocolo, versión TLS soportada, evidencia y excepción si aplica | Alta |
| Cifrado en reposo con claves gestionadas por QS en todos los nodos y entornos DR | Cumple parcialmente | Se observan Key Vault, Disk Encryption Key y referencias de cifrado | No se demuestra que todos los servicios soporten o usen claves gestionadas por QS en DR | Crear matriz de cobertura de CMK/BYOK por servicio; cerrar gaps en SQL, discos, storage y servicios analíticos según soporte real | Alta |
| Proceso aprobado de anonimización o datos sintéticos antes de pruebas de carga | No cumple | No existe en la documentación actual | Riesgo de uso indebido de datos reales en pruebas | Definir procedimiento formal de anonimización/sintetización y aprobación previa a pruebas del proyecto 4.2 | Alta |
| Anonimización en origen antes del traslado de datos | No cumple | No documentado | Riesgo de mover datos sensibles sin protección previa | Implementar en origen pipelines o procesos de masking antes de exportar o replicar datos a entornos no productivos | Alta |
| Logs de auditoría centralizados en plataforma inmutable y operativos en todos los nodos | Cumple parcialmente | La arquitectura contempla observabilidad | No se acredita inmutabilidad ni cobertura homogénea en todos los nodos/regiones | Definir plataforma central de logs, retención, inmutabilidad y validación también en entorno DR | Alta |

### 13.3 Controles técnicos requeridos por dominio

#### 13.3.1 Identidades y accesos
Se deberán implantar como mínimo los siguientes controles:
- una identidad gestionada por componente crítico,
- prohibición expresa de cuentas compartidas para servicios,
- MFA obligatorio para operadores y administradores,
- PIM o elevación temporal para privilegios administrativos,
- matriz de accesos diferenciada para operación normal, contingencia y desastre,
- revisión periódica de permisos.

#### 13.3.2 Gestión de secretos
Se deberán implantar como mínimo los siguientes controles:
- bóveda centralizada de secretos,
- rotación definida por tipo de secreto,
- expiración y alertado,
- uso preferente de Managed Identity frente a secretos estáticos,
- evidencias de acceso auditado.

#### 13.3.3 Protección de datos
Se deberán implantar como mínimo los siguientes controles:
- inventario de cifrado en tránsito por flujo,
- inventario de cifrado en reposo por servicio,
- CMK/BYOK donde aplique y exista soporte,
- procedimiento de anonimización en origen,
- control de uso de datos sintéticos para pruebas.

#### 13.3.4 Seguridad en el ciclo de vida
Se deberán implantar como mínimo los siguientes controles:
- versionado de runbooks,
- revisión y aprobación de cambios,
- hardening documentado de imágenes base,
- escaneo de vulnerabilidades,
- casos de prueba específicos de seguridad en simulacros DR.

#### 13.3.5 Monitorización y auditoría
Se deberán implantar como mínimo los siguientes controles:
- centralización de logs,
- retención definida,
- control de acceso a logs,
- inmutabilidad donde el requisito corporativo lo exija,
- validación de continuidad de la auditoría tras failover.

### 13.4 Criterios de aceptación específicos de seguridad

El plan técnico no deberá considerarse completamente aceptado hasta que se disponga de:
- matriz de accesos por modo operativo,
- inventario de identidades por componente,
- evidencia de MFA/PIM para operación,
- política de secretos y rotación,
- baseline de hardening aprobada,
- catálogo de cifrado en tránsito y en reposo,
- procedimiento de anonimización en origen,
- casos de prueba DR de seguridad,
- diseño de centralización e inmutabilidad de logs.

### 13.5 Acciones priorizadas

1. Definir matriz de accesos normal / contingencia / desastre.  
2. Confirmar MFA, Conditional Access y PIM para operación privilegiada.  
3. Inventariar identidades de servicio y eliminar credenciales compartidas.  
4. Inventariar flujos de replicación y validar cifrado real por canal.  
5. Definir política de Key Vault y rotación de secretos.  
6. Formalizar hardening de imágenes base y nodos.  
7. Añadir pruebas de seguridad al runbook y a los simulacros DR.  
8. Definir anonimización en origen y uso de datos sintéticos para pruebas.  
9. Diseñar centralización e inmutabilidad de logs en ambos nodos/regiones.  

## 14. Recomendaciones Finales

Se recomienda priorizar:
1. Azure SQL y Storage  
2. DNS privado y Private Endpoints  
3. Key Vault y Managed Identities  
4. AKS DR  
5. Databricks DR  
6. mecanismo de publicación  
7. automatización y simulacros  
8. cierre de gaps de seguridad identificados  

La eficacia del DR dependerá especialmente de:
- la automatización real,
- la consistencia del dato,
- la disponibilidad de secretos,
- la validez de la conectividad privada,
- la disciplina en las pruebas,
- y la implantación efectiva de los controles de seguridad requeridos.
