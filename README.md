> ### Confidentiality Notice
> This repository contains an anonymized portfolio case study based on a real-world project.
> 
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
> 
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# Directivas de Infraestructura Inicial para Automatización

## Descripción

Adaptación de pruebas automatizadas con Cypress a CI/CD en Azure DevOps, enfocada en la primera integración de suites automatizadas dentro de pipelines de la organización, asegurando su ejecución en despliegues a entornos DEV y PROD y validando el flujo BUILD → TEST → DEPLOY mediante la definición de estándares de configuración YAML para una integración estable y confiable.

## Objetivo

Estandarizar la ejecución de pruebas automatizadas dentro de la arquitectura de pipelines existente, facilitando la configuración de entornos, la generación de evidencia de ejecución y la incorporación de nuevas suites de prueba bajo un modelo de integración reutilizable y escalable.


## Tecnologías Utilizadas

```text
- Azure DevOps
- YAML Pipelines
- Cypress
- Mochawesome
- Azure Artifacts
- Git / GitHub Workflow
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

* Detecté que los pipelines YAML ya existían, pero estaban configurados de forma inestable.

* Adapté suites automatizadas con Cypress para integrarse correctamente al flujo existente.

* Validé la ejecución de pruebas automatizadas en entornos DEV y PROD.

* Contribuí a establecer las bases para la integración repetible de suites de Cypress en Azure DevOps Pipelines.

---

## Desafíos Técnicos

### Variables de Entorno

**Problema**

Las pruebas automatizadas dependían de variables que no estaban disponibles durante la ejecución en el pipeline.

**Solución**

Se implementó la exportación controlada de variables desde Azure DevOps Pipeline Variables hacia Cypress mediante directivas documentadas dentro del YAML.

**Ejemplo Visual**
```yml
 - stage: Test
    displayName: "Run Cypress Tests"
    dependsOn: Build
    condition: succeeded()

    jobs:
      - job: CypressTests
        displayName: "Execute Cypress E2E Tests"

        pool:
          [Redacted for Privacy]

          - script: |
              cd automated-tests
              npm ci
              npm install mochawesome
            displayName: "Install Cypress dependencies"

          - script: |
              cd automated-tests
              export CYPRESS_BASE_URL=http://localhost:8080
              export CYPRESS_USER=$(CYPRESS_USER)
              export CYPRESS_PASSWORD=$(CYPRESS_PASSWORD)
              export CYPRESS_API_URL=$(CYPRESS_API_URL)
              npx cypress run --reporter mochawesome --reporter-options "reportDir=cypress/reports,reportFilename=$(browser)Report,timestamp=mmddyyyy_HHMMss,overwrite=true,html=true,json=true"
            displayName: "Run Cypress Tests"
            continueOnError: true
```


### Reporting Automatizado

**Problema**

Los resultados de ejecución no generaban evidencia accesible para análisis posterior.

**Solución**

Se integró Mochawesome para generar reportes HTML y JSON publicados automáticamente como artifacts del pipeline.

```yml
npx cypress run --reporter mochawesome --reporter-options "reportDir=cypress/reports,reportFilename=$(browser)Report,timestamp=mmddyyyy_HHMMss,overwrite=true,html=true,json=true"
```


## Estrategias Adicionales Documentadas

Además de la configuración base para automatización, se documentaron alternativas de implementación para escenarios futuros. 

### *️⃣ Ejecución Cross-Browser (Opcional)

Como parte de las directivas del framework, también se documentó una estrategia opcional para ejecutar las mismas suites de automatización en múltiples navegadores mediante matrices de Azure DevOps y contenedores oficiales de Cypress.


```yml
### 1. Add the Cypress Browser Container
→ Under **Jobs → Pool**, immediately below:

vmImage: $(vmImageName)

Add:

container: cypress/browsers:latest
# Use latest, not version number
```
```yml
### 2. Add the Matrix Strategy
→ Immediately below `container: cypress/browsers:latest`, add:

strategy:
  matrix:
    chromeRun:
      browser: chrome
    firefoxRun:
      browser: firefox
    edgeRun:
      browser: edge
```
```yml
### 3. Configure Browser Execution
→ In the step where Cypress variables are exported and tests are executed, update the command:

npx cypress run

To:

npx cypress run --browser $(browser)
```
> Con esta configuración, Azure DevOps genera automáticamente una ejecución independiente para cada navegador definido en la matriz.
>
> Como resultado, la misma suite de pruebas de Cypress se ejecuta en Chrome, Firefox y Edge sin necesidad de crear jobs adicionales ni duplicar código de automatización.

**Navegadores considerados:**

* Chrome
* Firefox
* Edge

→ Esta configuración no fue implementada como parte de la solución base, ya que la arquitectura utilizada durante el proyecto ejecutaba las pruebas contra una instancia local de la aplicación dentro del pipeline:
```yml
Build de la aplicación
↓
Publicación del build en localhost:8080
↓
Ejecución de pruebas Cypress
↓
Despliegue a Dev o Prod
```

→ En este tipo de escenarios, ejecutar varios navegadores en paralelo sobre una misma instancia local puede provocar inestabilidad o resultados inconsistentes durante la ejecución:
> **Warning**
>
> Running multiple browsers in parallel against a single localhost instance may cause test instability or intermittent failures.

Por ello, la estrategia quedó documentada como una guía de implementación futura para equipos que requieran ampliar la cobertura de validación o adaptar el framework a arquitecturas CI/CD más avanzadas donde las pruebas se ejecuten sobre ambientes previamente desplegados.


---


## Conclusión

Esta implementación permite integrar suites Cypress dentro de arquitecturas CI/CD modernas, proporcionando trazabilidad, evidencia automática y una base escalable para futuras estrategias de automatización y ejecución multi-browser.


### *️⃣ Reporting Multi-Browser (Opcional)

Se documentó una configuración para generar artifacts independientes por navegador, permitiendo analizar resultados y evidencias de ejecución de forma aislada.
```yml
### 1. If Reporting and Cross-Browser execution will be used together
- Add the `container`.
- Add the `strategy`.
```
```yml
### 2. Update the Cypress Command
→ Adjust the execution command to include both the browser parameter and reporter configuration:
--reporter mochawesome --reporter-options "reportDir=cypress/reports,reportFilename=$(browser)Report,timestamp=mmddyyyy_HHMMss,overwrite=true,html=true,json=true"
```
```yml
### 3. Continue Pipeline Execution on Failures
→ Immediately below:
displayName: "Run Cypress Tests"

add:
continueOnError: true

This is required because the following steps must run regardless of whether the tests pass or fail.
```
```yml
### 4. Publish Browser-Specific Artifacts
→ Add the following publish tasks, including the `$(browser)` suffix in the artifact names:

- publish: $(System.DefaultWorkingDirectory)/automated-tests/cypress/reports
  artifact: Report_$(browser)
  condition: succeededOrFailed()

- publish: $(System.DefaultWorkingDirectory)/automated-tests/cypress/screenshots
  artifact: Screenshots_$(browser)
  condition: succeededOrFailed()
```

### Escalabilidad del Pipeline

Estas alternativas permiten reutilizar la misma base de automatización sin modificaciones significativas en las suites de prueba.


**Ejemplos compatibles:**

| Arquitectura                                                                                |
| ------------------------------------------------------------------------------------------- |
| Build → Test → Deploy                                                                       |
| Build → Deploy → Test                                                                       |
| Build → Deploy Dev → Cross-Browser Test                                                     |
| Build Dev → Deploy Dev → Cross-Browser Test → Build Prod → Deploy Prod → Cross-Browser Test |

> Escenarios que ejecutan pruebas sobre ambientes previamente desplegados (en lugar de una instancia local de la aplicación) permiten incorporar fácilmente estrategias de validación Cross-Browser mediante matrices de Azure DevOps, reutilizando las mismas suites de automatización.
>

Este enfoque permite reutilizar la misma base de automatización en distintos flujos de integración y despliegue.

En arquitecturas donde las pruebas se ejecutan después del deployment, la estrategia Cross-Browser documentada puede incorporarse fácilmente para ampliar la cobertura de validación sobre múltiples navegadores.


**Screenshots**

<img width="439" height="357" alt="image" src="https://github.com/user-attachments/assets/d8cbc560-6e6f-459d-9b8b-530662d899d9" />
<img width="439" height="357" alt="image" src="https://github.com/user-attachments/assets/dd8296c5-b3f6-46cd-a941-556570dd9e8a" />
<img width="439" height="357" alt="image" src="https://github.com/user-attachments/assets/e6fe69a0-2ef9-4107-9f7c-df5963160841" />






