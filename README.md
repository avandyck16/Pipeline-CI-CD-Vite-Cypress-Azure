### Pipeline CI/CD (Vite + Cypress + Azure Static Web Apps)

Este framework de automatización fue integrado dentro del flujo CI/CD mediante pipelines YAML de Azure DevOps, incorporando la ejecución de pruebas Cypress E2E como un quality gate previo al despliegue.

Este archivo YAML define un flujo automatizado que construye tu aplicación, corre pruebas end‑to‑end con Cypress y despliega a Azure Static Web Apps dependiendo de la rama (dev o main).
</br>
La idea es que cada vez que alguien hace un push o merge, el sistema se encargue de todo sin intervención manual.

### 🧭 Resumen del flujo

El pipeline sigue esta lógica:

| Situación      | Build inicial | Cypress | Build PROD | Deploy |
| -------------- | ------------- | ------- | ---------- | ------ |
| PR → `dev`     | DEV           | DEV     | —          | —      |
| Merge → `dev`  | DEV           | DEV     | —          | DEV    |
| PR → `main`    | DEV           | DEV     | —          | —      |
| Merge → `main` | DEV           | DEV     | PROD       | PROD   |

**En palabras simples:**

- Siempre se construye la app en modo dev.
- Siempre se ejecutan las pruebas de Cypress.
- Solo si se hace merge a main:
  - Se construye la versión prod.
  - Se despliega a producción.
- Solo si se hace merge a dev:
  - Se despliega a ambiente de desarrollo.

**¿Cuál es el impacto en calidad?**

- Con esto evitamos que si se encuentra un bug, éste llegue a producción.
- Se puede identificar rápidamente gracias a los reportes HTML.
- Corrobora que las plataformas estén limpias desde los Pull Requests.
- Previene deploys a Dev y a Main en estos casos.




### Fragmento que integra Cypress a Pipeline:
