# Automation Pipeline-Infrastructure Guidelines
Azure DevOps pipeline directives for Cypress execution, reporting and environment management. 

# Directivas de Infraestructura Inicial para Automatización

## Descripción

Proyecto enfocado en la primera integración de suites automatizadas de Cypress para la empresa dentro de Azure DevOps Pipelines, permitiendo la ejecución automatizada de pruebas, administración de variables de entorno, generación de reportes y publicación de artifacts para equipos de desarrollo y QA.

## Objetivo

Estandarizar la ejecución de pruebas automatizadas dentro del pipeline CI/CD, facilitando la configuración de entornos, la generación de evidencia de ejecución y la escalabilidad de las suites de prueba.

## Tecnologías Utilizadas

```text
Azure DevOps
YAML Pipelines
Cypress
Mochawesome
Node.js
Azure Artifacts
Docker Containers
Cross-Browser Testing
```

## Mi Participación

* Implementación de variables de entorno para automatización.
* Configuración de ejecución Cypress dentro del pipeline.
* Integración de reportes HTML y JSON mediante Mochawesome.
* Publicación automática de artifacts.
* Diseño de estrategia cross-browser mediante matrices.
* Documentación técnica para adopción por equipos de QA y Desarrollo.
* Validación de compatibilidad con arquitecturas CI/CD existentes.

## Logros y Resultados

* Ejecución automatizada de pruebas en Azure DevOps.
* Eliminación de configuraciones manuales repetitivas.
* Generación automática de evidencia de ejecución.
* Reportes accesibles directamente desde el pipeline.
* Estrategia preparada para ejecución multi-browser.
* Documentación reutilizable para futuros proyectos.

## Desafíos Técnicos

### Variables de Entorno

**Problema**

Las pruebas automatizadas dependían de variables que no estaban disponibles durante la ejecución en el pipeline.

**Solución**

Se implementó la exportación controlada de variables desde Azure DevOps Pipeline Variables hacia Cypress mediante directivas documentadas dentro del YAML.


### Reporting Automatizado

**Problema**

Los resultados de ejecución no generaban evidencia accesible para análisis posterior.

**Solución**

Se integró Mochawesome para generar reportes HTML y JSON publicados automáticamente como artifacts del pipeline.

### Ejecución Multi-Browser

**Problema**

Se requería una estrategia escalable para validar compatibilidad entre navegadores.

**Solución**

Se documentó una implementación basada en matrices de Azure DevOps utilizando contenedores oficiales de Cypress para ejecutar pruebas en Chrome, Firefox y Edge.

## Conclusión

Esta implementación permite integrar suites Cypress dentro de arquitecturas CI/CD modernas, proporcionando trazabilidad, evidencia automática y una base escalable para futuras estrategias de automatización y ejecución multi-browser.

## Estrategias Adicionales Documentadas

Además de la configuración base para automatización, se documentaron alternativas de implementación para escenarios futuros:

### Ejecución Cross-Browser

Se definió una estrategia basada en matrices de Azure DevOps para ejecutar las mismas suites en múltiples navegadores utilizando contenedores oficiales de Cypress.

**Navegadores considerados:**

* Chrome
* Firefox
* Edge

### Reporting Multi-Browser

Se documentó una configuración para generar artifacts independientes por navegador, permitiendo analizar resultados y evidencias de ejecución de forma aislada.

### Escalabilidad de Pipeline

Las directivas fueron diseñadas para adaptarse a distintas arquitecturas CI/CD, incluyendo escenarios donde la validación automatizada ocurre antes o después del deployment.

**Ejemplos compatibles:**

| Arquitectura                                                    |
| --------------------------------------------------------------- |
| Build → Test → Deploy                                           |
| Build → Deploy → Test                                           |
| Build Dev → Deploy Dev → Test → Build Prod → Deploy Prod → Test |

Estas alternativas permiten reutilizar la misma base de automatización sin modificaciones significativas en las suites de prueba.

**Screenshots**

<img width="439" height="357" alt="image" src="https://github.com/user-attachments/assets/d8cbc560-6e6f-459d-9b8b-530662d899d9" />
<img width="439" height="357" alt="image" src="https://github.com/user-attachments/assets/dd8296c5-b3f6-46cd-a941-556570dd9e8a" />
<img width="439" height="357" alt="image" src="https://github.com/user-attachments/assets/e6fe69a0-2ef9-4107-9f7c-df5963160841" />






