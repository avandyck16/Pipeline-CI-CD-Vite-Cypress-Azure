# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
>
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
>
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# CI/CD Pipeline — Cypress Quality Gate

**Vite + Cypress + Azure DevOps + Azure Static Web Apps**

Implementation of an automated CI/CD pipeline integrating Cypress E2E testing as a quality gate for a Vite application deployed through Azure Static Web Apps.

The implementation was developed from an existing pipeline that did not include automated Cypress validation, establishing the initial Build → Test → Deploy workflow and branch-based deployment strategy.

---

# Project Overview

**Beyond test automation:** I am not a Pipeline or DevOps Architect, but I wanted to go beyond simply writing tests. I took the initiative to understand how CI/CD pipelines work, how the different stages interact, and how QA automation could become part of that process. This project was the result of putting that knowledge into practice and building the initial testing pipeline for the team.


## Challenge

The existing CI/CD process did not include an automated E2E testing stage or a controlled relationship between test validation and deployment.

The pipeline needed to:

* Build a controlled application artifact.
* Execute Cypress against that artifact.
* Preserve test evidence when failures occurred.
* Prevent invalid changes from progressing through the deployment flow.
* Generate production builds only when required.
* Deploy according to the target branch.

## Objective

Implement Cypress E2E validation within Azure DevOps and establish a controlled CI/CD flow where automated test results determine whether the pipeline can continue toward deployment.

---

# My Contribution

* Designed and implemented the initial Cypress CI/CD integration.
* Developed the Azure DevOps YAML stages and execution conditions.
* Integrated Cypress into the existing build process.
* Configured application artifacts to be consumed by the test stage.
* Implemented local application serving for E2E execution.
* Integrated Mochawesome and JUnit reporting.
* Configured screenshots and test reports as pipeline artifacts.
* Implemented branch-based deployment conditions.
* Separated development and production build processes.
* Configured environment variables and deployment tokens.
* Established Cypress as a pre-deployment quality gate.

> This implementation focused on the QA/testing integration layer of the CI/CD process rather than acting as a replacement for dedicated DevOps or infrastructure architecture responsibilities.

---

# Technology Stack

* Azure DevOps
* YAML Pipelines
* Cypress
* JavaScript
* Node.js
* Vite
* Azure Static Web Apps
* Mochawesome
* GitHub

---

# Pipeline Architecture

The first implementation established the following flow:

```text
Build
  ↓
Cypress Validation
  ↓
Build Production (main only)
  ↓
Deploy
```

The application was initially built in Development mode for validation. Production artifacts were generated only after successful validation and only when the change was merged into `main`.

---

# Branch-Based Flow

| Situation      | Initial Build | Cypress | Build PROD | Deploy |
| -------------- | ------------- | ------- | ---------- | ------ |
| PR → `dev`     | DEV           | DEV     | —          | —      |
| Merge → `dev`  | DEV           | DEV     | —          | DEV    |
| PR → `main`    | DEV           | DEV     | —          | —      |
| Merge → `main` | DEV           | DEV     | PROD       | PROD   |

### Development

```text
Merge → dev
   ↓
Build DEV
   ↓
Cypress Validation
   ↓
Tests Passed
   ↓
Deploy DEV
```

### Production

```text
Merge → main
   ↓
Build DEV
   ↓
Cypress Validation
   ↓
Tests Passed
   ↓
Build PROD
   ↓
Deploy PROD
```

This design ensured that the production build was generated only after the development build had passed automated validation.

---

# Cypress Pipeline Integration

The Cypress stage downloads the application artifact generated during the Build stage, serves it locally, and executes the E2E suite against that exact artifact.

```yaml
- stage: Test
  displayName: "Run Cypress Tests"
  dependsOn: Build
  condition: succeeded()

  jobs:
    - job: CypressTests
      displayName: "Execute Cypress E2E Tests"

      pool:
        vmImage: $(vmImageName)

      steps:
        - task: NodeTool@0
          inputs:
            versionSpec: "20.x"

        - download: current
          artifact: vite-dist

        - script: npm install -g serve wait-on
          displayName: "Install test utilities"

        - script: |
            serve -s $(Pipeline.Workspace)/vite-dist -l 8080 &
          displayName: "Start application"

        - script: |
            wait-on http://localhost:8080
          displayName: "Wait for application"

        - script: |
            cd automated-tests
            npm ci
            npx cypress run \
              --reporter mochawesome \
              --reporter-options "reportDir=cypress/reports,html=true,json=true"
          displayName: "Run Cypress Tests"
          continueOnError: false

        - publish: $(System.DefaultWorkingDirectory)/automated-tests/cypress/reports
          artifact: Report
          condition: succeededOrFailed()

        - publish: $(System.DefaultWorkingDirectory)/automated-tests/cypress/screenshots
          artifact: Screenshots
          condition: failed()

        - task: PublishTestResults@2
          condition: always()
          inputs:
            testResultsFormat: "JUnit"
            testResultsFiles: "automated-tests/results/*.xml"
            failTaskOnFailedTests: true
```

### Why this implementation matters

Cypress does not validate the source code directly. It validates the **built application artifact** served inside the pipeline.

This creates a controlled validation point between application build and deployment.

---

# Artifact-Based Validation

The pipeline uses artifacts to transfer the application between stages.

```text
Build
  ↓
Vite Build
  ↓
vite-dist artifact
  ↓
Cypress Stage
  ↓
Serve artifact locally
  ↓
Run E2E tests
```

This approach ensures that the application being tested is the same build artifact produced by the pipeline rather than a separately generated local version.

---

# Reporting & Evidence

The pipeline generates and preserves execution evidence through:

* Mochawesome HTML reports.
* Mochawesome JSON reports.
* JUnit test results.
* Failure screenshots.
* Azure DevOps pipeline artifacts.

Reports and screenshots remain available even when a test execution fails, allowing the failure to be investigated without losing execution evidence.

### Quality Gate Behavior

`continueOnError: false` ensures that Cypress failures stop the pipeline and prevent subsequent deployment stages from executing.

This separates two concerns:

```text
Test Failure
    ↓
Preserve Evidence
    ↓
Publish Results
    ↓
Quality Gate
    ↓
Stop Deployment
```

---

# Production Build Strategy

Production artifacts are generated only after the Cypress validation stage succeeds and only when the target branch is `main`.

```yaml
- stage: BuildProd
  displayName: "Build Vite App (Prod)"
  dependsOn: Test
  condition: and(
    succeeded(),
    eq(variables['Build.SourceBranch'], 'refs/heads/main')
  )

  jobs:
    - job: BuildProdJob

      steps:
        - task: NodeTool@0
          inputs:
            versionSpec: "20.x"

        - script: npm ci
          displayName: "Install dependencies"

        - script: npm run build:prod
          displayName: "Build Production"

        - publish: dist
          artifact: vite-dist-prod
          displayName: "Publish production artifact"
```

This prevents unnecessary production builds for Development changes and ensures that the production artifact is created only after the validation stage.

---



## Development Deployment

A merge into `dev` triggers deployment using the previously validated Development artifact.

```text
Merge → dev
   ↓
Build DEV
   ↓
Cypress
   ↓
Deploy DEV
```

## Production Deployment

A merge into `main` triggers the production build only after Cypress validation succeeds.

```text
Merge → main
   ↓
Build DEV
   ↓
Cypress
   ↓
Build PROD
   ↓
Deploy PROD
```

Deployment stages are controlled through branch-specific conditions and Azure Static Web Apps deployment tokens.

---

# Technical Decisions

| Decision                         | Purpose                                             |
| -------------------------------- | --------------------------------------------------- |
| Build before Cypress             | Validate a real application artifact                |
| Artifact-based testing           | Keep Build and Test stages consistent               |
| Cypress as quality gate          | Prevent failed validation from progressing          |
| DEV build for initial validation | Maintain a controlled and repeatable test target    |
| PROD build after validation      | Avoid generating production artifacts unnecessarily |
| Branch-based conditions          | Control DEV vs PROD deployment                      |
| Published reports on failure     | Preserve evidence for investigation                 |
| Screenshots on failure           | Provide visual execution evidence                   |

---

# Results

The implementation established the first automated testing layer within the project's CI/CD process.

### Key Outcomes

* Cypress E2E testing integrated into Azure DevOps.
* Automated Build → Test → Deploy workflow established.
* E2E validation introduced as a CI/CD quality gate.
* Application artifacts transferred between pipeline stages.
* Automated reporting and failure evidence preserved.
* Development and production deployment paths separated.
* Production builds restricted to validated `main` changes.
* Branch-based deployment conditions implemented.
* Manual intervention reduced across the delivery process.

---

# Capabilities Demonstrated

* Azure DevOps YAML pipeline implementation.
* CI/CD testing integration.
* Cypress E2E execution in pipeline environments.
* Quality gate implementation.
* Artifact management between stages.
* Branch-based pipeline conditions.
* Automated test reporting.
* Deployment control through test results.
* Environment and secret configuration.
* Integration of QA validation into software delivery workflows.

---

# Evolution of the Pipeline

This implementation represents the **initial CI/CD testing architecture**.

Its main purpose was to establish automated E2E validation within an existing deployment process.

The architecture subsequently evolved toward a more controlled environment-specific strategy, where the build corresponding to the target branch is validated before its final deployment.

This evolution is documented separately in the project's later CI/CD implementations.

---

# Conclusion

This project established the initial automated testing infrastructure within the application's CI/CD process, integrating Cypress E2E validation into Azure DevOps from an environment where no automated Cypress pipeline stage existed.

The implementation introduced a controlled **Build → Test → Deploy** workflow, artifact-based validation, automated reporting, branch-based deployment conditions, and production-build control.

The result was a CI/CD process where automated QA validation became an integrated part of software delivery rather than a separate manual activity.

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>
