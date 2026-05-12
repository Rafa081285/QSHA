# PLAN DIRECTOR DE DISASTER RECOVERY (DR) — VISIÓN EJECUTIVA

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
| Documento | Plan Director de DR — Visión Ejecutiva |
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
16. Conclusión  

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

El presente plan cubre, a nivel ejecutivo, los siguientes dominios:

### 4.1 Conectividad y red
- Azure Virtual WAN
- hub/spoke networking
- subredes, rutas y segmentación
- firewalling
- DNS privado **(MSFT validated)**
- private endpoints **(MSFT validated)**

### 4.2 Plataforma de aplicación
- AKS
- publicación e ingreso
- dependencias con servicios PaaS

### 4.3 Datos
- Azure SQL **(MSFT validated)**
- almacenamiento asociado
- backups y restauración **(MSFT validated)**

### 4.4 Analítica
- Azure Databricks
- accesos privados a datos
- procesamiento crítico

### 4.5 Seguridad y operación
- Key Vault **(MSFT validated)**
- Managed Identity **(MSFT validated)**
- claves de cifrado **(MSFT validated)**
- monitorización y alertado

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

## 16. Conclusión

La adopción de un modelo **warm-standby en Spain Central** constituye una estrategia adecuada para incrementar la resiliencia de la plataforma Azure analizada.

Se recomienda avanzar hacia la implantación técnica priorizando:
- datos,
- red y DNS,
- secretos e identidades,
- AKS,
- Databricks,
- automatización,
- pruebas operativas.
