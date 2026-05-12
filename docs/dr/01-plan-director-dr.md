# PLAN DIRECTOR DE DISASTER RECOVERY (DR) — VISIÓN EJECUTIVA

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
| Documento | Plan Director de DR — Visión Ejecutiva |
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
| 1.1 | 2026-05-12 | Inclusión de evaluación de requisitos de seguridad, gaps y mitigaciones | Copilot |
| 1.2 | 2026-05-12 | Ampliación del alcance por categoría y servicio | Copilot |

## Índice

1. Resumen ejecutivo  
2. Objeto del documento  
3. Contexto  
4. Alcance  
5. Estrategia de continuidad propuesta  
6. Justificación de la estrategia  
7. Objetivos de recuperación  
8. Beneficios esperados  
9. Riesgos cubiertos  
10. Riesgos residuales  
11. Principios rectores  
12. Componentes estratégicos  
13. Roadmap de implantación  
14. Requisitos organizativos  
15. Indicadores de éxito  
16. Cumplimiento de requisitos de seguridad  
17. Conclusión  

## 1. Resumen Ejecutivo

El presente documento define la estrategia de **Disaster Recovery (DR)** para la plataforma desplegada en **Microsoft Azure**, tomando como base la arquitectura facilitada y considerando una estrategia de **alta disponibilidad regional con failover en modo warm-standby** hacia la región **Spain Central**.

El objetivo es garantizar la continuidad del servicio ante un incidente grave o una indisponibilidad prolongada de la región primaria, reduciendo el impacto sobre:
- la disponibilidad de los servicios,
- la integridad de los datos,
- la continuidad operativa,
- los compromisos de recuperación.

La estrategia propuesta contempla una región secundaria preparada para asumir la operación con una infraestructura parcialmente activa y sincronizada, evitando reconstrucciones manuales complejas durante la contingencia.

## 2. Objeto del Documento

Este documento tiene por objeto presentar una visión ejecutiva de la estrategia de recuperación ante desastres para la plataforma Azure, incluyendo:
- el alcance de la iniciativa,
- el modelo objetivo de continuidad,
- los principios de diseño,
- los riesgos cubiertos,
- los beneficios esperados,
- el roadmap de implantación.

## 3. Contexto

La arquitectura analizada refleja una plataforma empresarial basada en Azure con componentes de:
- conectividad y seguridad perimetral,
- Azure Virtual WAN,
- publicación de servicios,
- topología hub-and-spoke,
- contenedores y cargas sobre AKS,
- Azure SQL **(MSFT validated)**,
- Azure Databricks,
- Key Vault y claves de cifrado **(MSFT validated)**,
- Private Endpoints y DNS privado **(MSFT validated)**,
- observabilidad y servicios operacionales.

**Imagen de referencia de arquitectura:**

![image1](image1)

La criticidad de los componentes y la dependencia de servicios regionales hacen recomendable la implantación de una estrategia formal de DR con capacidad de failover hacia una segunda región.

## 4. Alcance

El presente plan cubre, a nivel ejecutivo, los dominios y servicios identificados en la arquitectura objetivo. El alcance no se limita a una clasificación funcional, sino que incorpora el tratamiento previsto de Disaster Recovery por servicio dentro de cada categoría.

### 4.1 Conectividad y red

#### Azure Virtual WAN
- **Rol en la arquitectura:** backbone de conectividad entre redes, hubs y servicios conectados.
- **Objetivo en DR:** mantener la conectividad troncal y el enrutamiento necesarios para operar desde Spain Central.
- **Tratamiento DR:** réplica funcional de la conectividad y validación de rutas, propagación y reachability.
- **Estado esperado en DR:** preaprovisionado y validado.
- **Riesgo principal:** rutas incompletas o dependencias no reflejadas en DR.

#### Topología hub/spoke
- **Rol en la arquitectura:** segmentación de redes y separación de cargas.
- **Objetivo en DR:** reproducir el aislamiento y la conectividad de los entornos críticos.
- **Tratamiento DR:** despliegue equivalente de VNets, subredes, NSGs y UDRs.
- **Estado esperado en DR:** desplegado y alineado con el diseño primario.
- **Riesgo principal:** diferencias de direccionamiento o reglas no replicadas.

#### Firewalling y publicación perimetral
- **Rol en la arquitectura:** control de tráfico y exposición de servicios.
- **Objetivo en DR:** mantener la publicación segura y controlada de los servicios críticos.
- **Tratamiento DR:** despliegue de componente equivalente de publicación y seguridad, listo para conmutación.
- **Estado esperado en DR:** activo o preparado para activación.
- **Riesgo principal:** reglas, certificados o backends no sincronizados.

#### DNS privado y Private Endpoints **(MSFT validated)**
- **Rol en la arquitectura:** resolución interna y acceso privado a servicios PaaS.
- **Objetivo en DR:** asegurar conectividad privada funcional desde cargas ejecutadas en Spain Central.
- **Tratamiento DR:** duplicación de zonas DNS privadas, enlaces y private endpoints necesarios.
- **Estado esperado en DR:** operativo antes del failover.
- **Riesgo principal:** errores de resolución o endpoints no enlazados correctamente.

### 4.2 Plataforma de aplicación

#### AKS
- **Rol en la arquitectura:** plataforma principal de ejecución de workloads contenerizados.
- **Objetivo en DR:** permitir la ejecución de servicios críticos en la región secundaria.
- **Tratamiento DR:** clúster secundario desplegado con capacidad mínima y escalado bajo demanda.
- **Estado esperado en DR:** preaprovisionado, integrado con red, secretos, imágenes y observabilidad.
- **Riesgo principal:** divergencia de configuración, secretos no disponibles o persistencia no protegida.

#### Ingress / publicación de aplicaciones
- **Rol en la arquitectura:** exposición de servicios ejecutados sobre AKS u otras plataformas.
- **Objetivo en DR:** conmutar la entrada del servicio al entorno secundario sin rediseño en crisis.
- **Tratamiento DR:** configuración equivalente de publicación, certificados y backends.
- **Estado esperado en DR:** preparado para activación.
- **Riesgo principal:** certificados, reglas o mappings no alineados.

### 4.3 Datos

#### Azure SQL **(MSFT validated)**
- **Rol en la arquitectura:** repositorio relacional crítico para la operación.
- **Objetivo en DR:** garantizar recuperabilidad con RTO/RPO acordados.
- **Tratamiento DR:** geo-réplica o auto-failover group hacia Spain Central.
- **Estado esperado en DR:** replicando y listo para promoción.
- **Riesgo principal:** latencia de replicación, accesos no validados o failover no probado.

#### Storage / Data Lake
- **Rol en la arquitectura:** almacenamiento de datos operacionales, ficheros y/o datasets analíticos.
- **Objetivo en DR:** mantener disponibilidad de datos requeridos por aplicaciones y analítica.
- **Tratamiento DR:** redundancia geográfica o cuenta secundaria con acceso desde DR.
- **Estado esperado en DR:** replicando o preparado para promoción.
- **Riesgo principal:** dependencia de endpoints, permisos o promoción no transparente.

#### Backup y restauración **(MSFT validated)**
- **Rol en la arquitectura:** mecanismo de recuperación adicional ante corrupción, borrado o fallo lógico.
- **Objetivo en DR:** permitir recuperación alternativa cuando la réplica no sea suficiente o no sea viable.
- **Tratamiento DR:** políticas de backup, retención y restore tests periódicos.
- **Estado esperado en DR:** operativo y validado.
- **Riesgo principal:** backups no restaurados periódicamente o cobertura incompleta.

### 4.4 Analítica y procesamiento

#### Azure Databricks
- **Rol en la arquitectura:** procesamiento analítico, pipelines de datos y ejecución de cargas analíticas.
- **Objetivo en DR:** reanudar los procesos analíticos críticos tras el failover.
- **Tratamiento DR:** workspace secundario desplegado con conectividad, secretos y objetos críticos preparados.
- **Estado esperado en DR:** preaprovisionado, con activación controlada.
- **Riesgo principal:** objetos no versionados, dependencias de catálogo o secretos no sincronizados.

#### Jobs, notebooks y políticas analíticas
- **Rol en la arquitectura:** definición de procesos de negocio y explotación de datos.
- **Objetivo en DR:** reactivar en orden de prioridad los procesos analíticos esenciales.
- **Tratamiento DR:** versionado y replicación lógica de jobs, notebooks, configuraciones y dependencias.
- **Estado esperado en DR:** preparado para ejecución.
- **Riesgo principal:** dependencia manual o configuración divergente.

### 4.5 Seguridad y operación

#### Azure Key Vault **(MSFT validated)**
- **Rol en la arquitectura:** custodia de secretos, certificados y claves.
- **Objetivo en DR:** asegurar que los componentes en DR acceden a sus secretos y materiales criptográficos.
- **Tratamiento DR:** Key Vault secundario o estrategia equivalente con replicación/control de secretos.
- **Estado esperado en DR:** operativo antes de la activación.
- **Riesgo principal:** secretos no sincronizados o referencias al vault primario.

#### Managed Identities **(MSFT validated)**
- **Rol en la arquitectura:** autenticación de servicios sin credenciales embebidas.
- **Objetivo en DR:** mantener acceso seguro entre componentes.
- **Tratamiento DR:** asignación de identidades equivalentes y permisos explícitos en entorno secundario.
- **Estado esperado en DR:** preparado y validado.
- **Riesgo principal:** permisos incompletos o reutilización no controlada.

#### Observabilidad y monitorización
- **Rol en la arquitectura:** supervisión técnica, alertado y trazabilidad operativa.
- **Objetivo en DR:** mantener visibilidad completa del entorno secundario durante contingencia.
- **Tratamiento DR:** continuidad de métricas, logs y alertas en DR.
- **Estado esperado en DR:** activo y accesible.
- **Riesgo principal:** pérdida de trazabilidad o cobertura incompleta de logs en DR.

## 5. Estrategia de Continuidad Propuesta

Se propone una estrategia **activo-pasivo en modo warm-standby** **(MSFT validated)**.

### 5.1 Modelo operativo
- La **región primaria** soporta la operación habitual.
- La **región secundaria (Spain Central)** mantiene componentes desplegados y preparados para activación.
- El failover se realizará de forma controlada mediante procedimientos operativos documentados.

### 5.2 Capacidades esperadas en DR
En Spain Central deberán mantenerse:
- red y seguridad base desplegadas,
- servicios críticos de datos replicados,
- componentes de aplicación con capacidad mínima o lista para escalado,
- mecanismos de publicación listos para conmutación,
- automatización y runbooks operativos.

### 5.3 Alcance del warm-standby
El entorno de DR no se considera un entorno activo-activo ni un duplicado a plena capacidad.  
Su objetivo es reducir sustancialmente el tiempo de recuperación manteniendo costes razonables.

## 6. Justificación de la Estrategia

La estrategia warm-standby se considera adecuada porque proporciona equilibrio entre:
- resiliencia,
- coste,
- simplicidad operativa,
- capacidad de prueba,
- velocidad de recuperación.

### 6.1 Frente a cold-standby
Reduce:
- el tiempo de recuperación,
- el riesgo de error humano,
- la dependencia de reconstrucción manual.

### 6.2 Frente a active-active
Evita:
- sobrecostes estructurales elevados,
- complejidad de sincronización continua,
- incremento del esfuerzo operativo y de gobierno.

## 7. Objetivos de Recuperación

Los siguientes objetivos deberán validarse con los propietarios de servicio y negocio:

| Indicador | Objetivo inicial propuesto |
|---|---|
| RTO plataforma crítica | 1 a 4 horas |
| RTO Azure SQL | 5 a 30 minutos |
| RPO Azure SQL | 5 a 30 minutos |
| RTO AKS | 30 a 90 minutos |
| RTO Databricks | 30 a 120 minutos |
| RPO Storage | Según mecanismo de replicación |

## 8. Beneficios Esperados

La implantación del DR permitirá:
- reducir el riesgo de indisponibilidad prolongada,
- mejorar la recuperabilidad de los servicios críticos,
- minimizar la dependencia de tareas manuales,
- incrementar la trazabilidad y auditabilidad,
- facilitar pruebas recurrentes de continuidad,
- fortalecer la confianza operativa de negocio y tecnología.

## 9. Riesgos Cubiertos

La estrategia está orientada a mitigar escenarios como:
- caída completa de la región primaria,
- degradación severa de servicios regionales,
- fallos de conectividad regional,
- indisponibilidad de componentes de datos,
- indisponibilidad operativa de AKS,
- incidentes que requieran aislamiento regional.

## 10. Riesgos Residuales

Permanecerán riesgos residuales como:
- pérdida de datos dentro de la ventana de replicación asíncrona,
- divergencias entre configuración real y documentada,
- dependencias externas no redundadas,
- limitaciones de cuota o capacidad regional,
- errores en configuraciones no automatizadas,
- tiempos de propagación DNS.

## 11. Principios Rectores

1. **Automatización por defecto**  
   Toda la infraestructura replicable deberá desplegarse mediante IaC.

2. **Datos como prioridad**  
   La protección y recuperabilidad de los datos son prioritarias.

3. **Configuración versionada**  
   Todo cambio crítico deberá estar trazado y versionado.

4. **Pruebas periódicas**  
   El DR deberá validarse mediante simulacros recurrentes.

5. **Operación simple y auditada**  
   El failover debe ser repetible, entendible y controlado.

## 12. Componentes Estratégicos

Los principales habilitadores de la solución serán:
- red secundaria preaprovisionada,
- replicación de Azure SQL **(MSFT validated)**,
- estrategia de storage con protección regional,
- AKS secundario,
- Databricks secundario,
- Key Vault y secretos disponibles en DR,
- publicación y routing preparados,
- monitorización y runbooks operativos.

## 13. Roadmap de Implantación

### 13.1 Fase 1 — Assessment y diseño
- inventario de activos,
- mapa de dependencias,
- clasificación de criticidad,
- definición final de RTO/RPO,
- diseño técnico objetivo.

### 13.2 Fase 2 — Landing Zone DR
- red,
- conectividad,
- seguridad,
- DNS,
- componentes base.

### 13.3 Fase 3 — Protección de datos
- replicación SQL **(MSFT validated)**,
- backups **(MSFT validated)**,
- estrategia de storage,
- restore testing.

### 13.4 Fase 4 — Plataforma de aplicación
- despliegue de AKS DR,
- pipelines,
- publicación,
- pruebas de aplicación.

### 13.5 Fase 5 — Plataforma analítica
- Databricks DR,
- conectividad privada,
- automatización de despliegues,
- validación funcional.

### 13.6 Fase 6 — Operación y gobierno
- runbooks,
- simulacros,
- métricas,
- mejora continua.

## 14. Requisitos Organizativos

Será necesario disponer de:
- comité de crisis,
- responsables por dominio tecnológico,
- gestión formal del cambio,
- accesos break-glass,
- calendario de pruebas,
- repositorio documental unificado,
- proceso de mejora posterior a simulacros.

## 15. Indicadores de Éxito

Se recomienda medir:
- cumplimiento real de RTO/RPO,
- porcentaje de infraestructura desplegada por IaC,
- porcentaje de automatización del failover,
- número de hallazgos críticos en pruebas,
- tiempo medio real de recuperación,
- cobertura de servicios críticos en DR.

## 16. Cumplimiento de requisitos de seguridad

### 16.1 Criterio de evaluación

Los estados utilizados en esta sección son:
- **Cumple**
- **Cumple parcialmente**
- **No evidenciado**
- **No cumple**
- **Pendiente de validación**

La evaluación se basa en la arquitectura compartida, en los documentos de DR definidos y en el contraste parcial realizado con capacidades conocidas de Azure. Cuando un requisito no puede demostrarse con la información disponible, se clasifica como **No evidenciado** o **Pendiente de validación**.

### 16.2 Resumen ejecutivo de cumplimiento

| Requisito | Estado | Motivo | Mitigación propuesta |
|---|---|---|---|
| mTLS o canales dedicados cifrados en replicación entre CPDs | No evidenciado | La arquitectura no demuestra mTLS extremo a extremo ni cifrado explícito en todos los canales entre CPDs; además se indica que MPLS sin cifrado no sería válido | Documentar cada flujo de replicación, exigir cifrado en tránsito por servicio, usar VPN/ExpressRoute con cifrado complementario cuando aplique y registrar excepción si un canal no soporta mTLS |
| Identidades de servicio únicas por componente | Cumple parcialmente | Existen Managed Identities en la arquitectura, pero no se evidencia unicidad por cada componente ni prohibición de credenciales compartidas | Definir política de una identidad por componente crítico y eliminar secretos/credenciales compartidas |
| MFA obligatorio para consolas de clúster y orquestación | No evidenciado | La arquitectura no muestra políticas de acceso condicional ni MFA | Implementar MFA obligatorio con Microsoft Entra ID, Conditional Access y PIM para accesos privilegiados |
| Matriz de accesos en modo normal, contingencia y desastre | No cumple | No estaba definida explícitamente en la documentación actual | Añadir matriz de accesos formal por rol, entorno y modo operativo |
| Secretos en bóveda centralizada con rotación | Cumple parcialmente | Se observa uso de Key Vault, pero no se acredita que todos los secretos estén centralizados ni que exista rotación formal | Consolidar todos los secretos en Azure Key Vault y definir política de rotación y custodia |
| Control de versiones y acceso auditado de runbooks/playbooks | Cumple parcialmente | Los runbooks están versionados en GitHub, pero no se ha documentado aún control de acceso auditado ni gobierno de cambios | Activar branch protection, PR obligatoria, CODEOWNERS y auditoría de cambios |
| Hardening documentado de imágenes base en nodos HA | No evidenciado | No se aporta baseline ni procedimiento de hardening | Definir hardening CIS o baseline corporativa, imágenes golden y evidencias de escaneo |
| Pruebas de seguridad durante conmutación | No cumple | Los documentos no incluían todavía pruebas específicas de cifrado, logs y accesos tras failover | Añadir casos de prueba de seguridad en simulacros DR |
| TLS 1.3 en todos los canales de replicación | Pendiente de validación | No puede garantizarse con la información disponible ni para todos los servicios gestionados | Inventariar por flujo y servicio el protocolo soportado; si no se garantiza TLS 1.3, registrar excepción y control compensatorio |
| Cifrado en reposo con claves gestionadas por QS | Cumple parcialmente | Hay evidencia de claves y cifrado, pero no cobertura demostrada para todos los servicios y nodos DR | Definir mapa de cobertura de CMK/BYOK y cerrar gaps por componente |
| Anonimización o datos sintéticos antes de pruebas de carga | No cumple | No existe proceso documentado en el plan actual | Definir procedimiento aprobado de anonimización o datos sintéticos para proyecto 4.2 |
| Anonimización en origen antes del movimiento de datos | No cumple | No está recogido en la documentación actual | Incorporar control obligatorio en origen antes de mover datos a otros entornos |
| Logs de auditoría centralizados en plataforma inmutable en todos los nodos | Cumple parcialmente | La arquitectura contempla observabilidad, pero no inmutabilidad demostrada para todos los logs | Centralizar logs en plataforma corporativa con retención, control de acceso e inmutabilidad donde aplique |

### 16.3 Posicionamiento ejecutivo

A nivel ejecutivo, la arquitectura propuesta presenta una **base razonable para cumplir parcialmente** los requisitos de seguridad gracias al uso de capacidades como **Azure Key Vault**, **Managed Identity**, **Private Endpoints** y mecanismos de cifrado y replicación en servicios gestionados de Azure. No obstante, el cumplimiento **no puede considerarse completo** con la evidencia actualmente disponible.

Los principales gaps se concentran en:
- gobierno de accesos privilegiados y MFA,
- matriz formal de accesos por modo operativo,
- definición y evidencia de hardening,
- pruebas de seguridad en escenarios de conmutación,
- anonimización de datos para pruebas,
- demostración formal del cifrado exigido en todos los canales de replicación.

### 16.4 Recomendación ejecutiva

Se recomienda tratar estos requisitos como un **workstream específico de seguridad DR**, con entregables mínimos obligatorios antes de la aceptación final del plan:
- matriz de accesos normal / contingencia / desastre,
- política de identidades únicas por componente,
- política MFA/PIM para operación,
- procedimiento de rotación de secretos,
- baseline de hardening,
- plan de pruebas de seguridad DR,
- catálogo de cifrado en tránsito y en reposo por componente,
- procedimiento formal de anonimización en origen.

## 17. Conclusión

La adopción de un modelo **warm-standby en Spain Central** constituye una estrategia adecuada para incrementar la resiliencia de la plataforma Azure analizada.

Se recomienda avanzar hacia la implantación técnica priorizando:
- datos,
- red y DNS,
- secretos e identidades,
- AKS,
- Databricks,
- automatización,
- pruebas operativas,
- y el cierre de los gaps de seguridad identificados.
