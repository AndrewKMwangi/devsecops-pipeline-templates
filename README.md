# DevSecOps Pipeline Templates

## 1. Project Overview

DevSecOps Pipeline Templates is a reusable CI/CD framework built with GitHub Actions. It provides standardized, security-focused automation for building, testing, scanning, packaging, and delivering applications.

The framework is designed to be consumed by multiple application repositories rather than maintaining a separate, independently implemented pipeline for every project.

The initial implementation will support Java/Spring Boot and Node.js applications, with container delivery and Kubernetes GitOps deployment capabilities.

## 2. Project Objectives

The project aims to:

- Build reusable GitHub Actions workflows that can be called by multiple repositories.
- Standardize application build, test, and quality validation.
- Integrate security checks throughout the CI/CD lifecycle.
- Build and scan container images before publication.
- Generate Software Bill of Materials (SBOM) artifacts.
- Publish approved container images to GitHub Container Registry (GHCR).
- Support GitOps-based Kubernetes deployments.
- Establish consistent security policies, permissions, and release controls.
- Provide documentation and examples that make pipeline adoption straightforward.

## 3. Architecture

The framework separates pipeline implementation from application source code.

**Central framework repository**

`AndrewKMwangi/devsecops-pipeline-templates`

Contains reusable workflows, composite actions, security policies, examples, and documentation.

**Consumer repositories**

Application repositories call the central workflows and provide their own application-specific configuration.

Initial consumer:

`AndrewKMwangi/baraza-banking-platform`

Future validation will use a second application to demonstrate that the framework is reusable beyond Baraza.

## 4. Supported Technologies

| Area | Initial technology |
|---|---|
| CI/CD orchestration | GitHub Actions |
| Java applications | Java 21, Maven, Spring Boot |
| Frontend applications | Node.js, React, Vite |
| Containerization | Docker |
| Container registry | GitHub Container Registry |
| Kubernetes | Kubernetes, Helm |
| GitOps | Argo CD |
| Infrastructure security | Checkov |
| Secret detection | Gitleaks |
| Container security | Trivy |
| Static application security testing | CodeQL |
| Dependency security | Dependabot, Dependency Review |
| SBOM generation | Syft |
| Container signing | Cosign |

Tool selection and enforcement will be configurable where appropriate, rather than requiring every consumer to adopt every tool.

## 5. Pipeline Lifecycle

The intended delivery lifecycle is:

1. Source code is pushed or a pull request is opened.
2. CI checks out the source and validates the application build.
3. Automated tests and quality checks execute.
4. Security checks identify vulnerabilities, insecure configurations, and exposed secrets.
5. The application is packaged into a container image.
6. Container security scanning and SBOM generation take place.
7. Approved images are published to GHCR.
8. GitOps deployment configuration is updated.
9. Argo CD reconciles the desired state with the Kubernetes environment.
10. Deployment health and application availability are verified.

The framework will distinguish successful CI validation from successful deployment. A passing build alone will not be treated as proof of a healthy production deployment.

## 6. Security Principles

The framework will apply the following principles:

- Least-privilege GitHub Actions permissions.
- Secure handling of credentials and secrets.
- Pinned third-party GitHub Actions dependencies.
- Automated source, dependency, infrastructure, and container security checks.
- Configurable vulnerability thresholds and security gates.
- Artifact traceability and versioned releases.
- Image signing and provenance where implemented.
- Environment-based deployment controls and approvals.
- Auditable deployment and release processes.

## 7. Repository Structure

The repository will evolve towards the following structure:

```text
devsecops-pipeline-templates/
├── .github/
│   └── workflows/
├── actions/
├── templates/
│   ├── java/
│   ├── node/
│   ├── docker/
│   └── kubernetes/
├── policies/
├── examples/
│   ├── spring-boot/
│   └── react/
├── docs/
└── README.md
```

The final structure will be refined as the framework is implemented.

## 8. Implementation Roadmap

| Phase | Deliverable |
|---|---|
| 1 | Architecture, repository structure, and engineering standards |
| 2 | Reusable Java/Maven and Node.js CI workflows |
| 3 | Integrated DevSecOps security controls |
| 4 | Container build, scan, SBOM, and GHCR publication |
| 5 | GitOps deployment integration with Argo CD |
| 6 | Deployment verification, release governance, and recovery |
| 7 | Documentation and validation with a second application |

## 9. Design Philosophy

The framework prioritizes reusability, security, maintainability, and traceability.

It is intended to provide a consistent engineering standard while allowing individual application repositories to configure the workflows according to their technology stack and delivery requirements.

The framework will be validated through actual application repositories, beginning with Baraza Banking Platform.

---

**Project status:** Initial repository setup. Implementation is in progress.