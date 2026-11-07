# DevSecOps Pipeline Templates — Architecture

## 1. Purpose

This document defines the technical architecture of the DevSecOps Pipeline Templates framework.

The framework provides reusable, security-focused CI/CD capabilities through GitHub Actions. It separates pipeline implementation from application source code so that multiple application repositories can consume a centrally maintained set of workflows and actions.

The initial implementation targets Java/Spring Boot and Node.js applications, with Docker-based container delivery and Kubernetes GitOps deployment.

## 2. Architectural Goals

The architecture is designed around the following goals:

- **Reusability:** Implement pipeline capabilities once and consume them across multiple repositories.
- **Modularity:** Separate build, test, security, container delivery, and deployment responsibilities.
- **Security:** Integrate security controls throughout the software delivery lifecycle.
- **Consistency:** Apply common engineering standards across different applications.
- **Configurability:** Allow consumer repositories to provide application-specific inputs and policies.
- **Traceability:** Preserve links between source revisions, builds, security results, container images, and deployments.
- **Maintainability:** Centralize workflow improvements and version them for controlled adoption.

## 3. High-Level Architecture

The architecture has three main areas:

1. Central pipeline framework.
2. Consumer application repositories.
3. Delivery and runtime infrastructure.

### 3.1 Central Pipeline Framework

Repository:

`AndrewKMwangi/devsecops-pipeline-templates`

This repository owns the reusable pipeline implementation.

Its responsibilities include:

- Providing reusable GitHub Actions workflows.
- Providing composite actions for repeated pipeline operations.
- Maintaining shared security controls and policies.
- Defining supported workflow inputs and outputs.
- Publishing versioned releases.
- Providing examples and adoption documentation.

The framework should not contain application-specific business logic.

### 3.2 Consumer Application Repositories

Consumer repositories contain application source code and invoke the reusable workflows exposed by the central framework.

Initial consumer:

`AndrewKMwangi/baraza-banking-platform`

A future second application will validate that the framework can support a different application without copying or rewriting its pipeline implementation.

Consumer repositories are responsible for:

- Maintaining application source code.
- Defining application-specific build configuration.
- Providing required workflow inputs.
- Managing application-specific secrets and permissions.
- Maintaining Dockerfiles and deployment configuration where applicable.
- Invoking framework workflows at approved versions.

### 3.3 Delivery and Runtime Infrastructure

The delivery architecture initially uses:

- GitHub Actions for CI/CD orchestration.
- GHCR for container image storage.
- A separate GitOps repository for Kubernetes desired-state configuration.
- Argo CD for Kubernetes reconciliation.
- Kubernetes as the application runtime environment.

The framework coordinates delivery but does not replace Kubernetes, Argo CD, or the application repositories.

## 4. Reusable Workflow Architecture

Reusable workflows are the primary mechanism for exposing pipeline capabilities to consumer repositories.

They are defined in the framework repository under:

`.github/workflows/`

A reusable workflow declares:

`on: workflow_call`

This allows another GitHub Actions workflow to invoke it using the `uses` keyword.

The initial workflow capabilities are:

| Workflow | Responsibility |
|---|---|
| Java CI | Build and test Java/Maven applications |
| Node CI | Install dependencies, test, and build Node.js applications |
| Security Scan | Execute configured application and dependency security checks |
| Container Build | Build container images and prepare image metadata |
| Container Security | Scan images and generate SBOM artifacts |
| Release | Publish approved artifacts and container images |
| GitOps Deploy | Update GitOps configuration and coordinate deployment verification |

Each workflow should have a clearly defined responsibility.

Workflows should accept explicit inputs rather than relying on assumptions about consumer repository structure.

## 5. Composite Actions

Composite actions package repeated steps into reusable units.

They will be maintained under:

`actions/`

Potential examples include:

- Java environment setup.
- Node.js environment setup.
- Maven build and test execution.
- Docker build configuration.
- Gitleaks execution.
- Trivy scanning.
- SBOM generation.

A reusable workflow provides a complete pipeline capability, while a composite action provides a smaller reusable sequence of steps.

This distinction avoids creating large workflows filled with duplicated configuration.

## 6. Intended Pipeline Flow

The framework will support the following delivery lifecycle:

### Stage 1: Source Validation

A push or pull request triggers the consumer repository workflow.

The workflow checks out the source and establishes the required build environment.

### Stage 2: Application CI

The appropriate reusable workflow executes according to the application technology.

For Java applications:

- Configure the required Java version.
- Resolve Maven dependencies.
- Compile the application.
- Execute automated tests.
- Produce build artifacts.

For Node.js applications:

- Configure the required Node.js version.
- Install dependencies using the repository lockfile.
- Execute tests.
- Build the frontend application.

### Stage 3: Security Validation

Configured security workflows execute relevant checks.

These may include:

- Static Application Security Testing using CodeQL.
- Secret detection using Gitleaks.
- Dependency review and vulnerability assessment.
- Infrastructure-as-Code scanning using Checkov.

Security results are evaluated against the configured policy.

### Stage 4: Container Delivery

For containerized applications:

- Build the Docker image.
- Apply traceable image tags.
- Scan the image using Trivy.
- Generate an SBOM using Syft.
- Sign images using Cosign where enabled.
- Publish approved images to GHCR.

Container publication must be controlled by the required validation and security gates.

### Stage 5: GitOps Deployment

For applications using Kubernetes GitOps:

- Identify the approved image version.
- Update the relevant image reference in the GitOps repository.
- Commit or propose the desired-state change.
- Allow Argo CD to reconcile the change.
- Verify deployment and application health.

The framework should not treat a successful GitOps configuration update as proof that an application has deployed successfully.

### Stage 6: Deployment Verification

Deployment verification should check the resulting application state.

Depending on the application and environment, this may include:

- Kubernetes rollout status.
- Pod readiness.
- Health endpoints.
- Application smoke tests.
- Deployment failure detection.

Recovery and rollback procedures will be defined as the deployment capabilities mature.

## 7. Security Boundaries and Permissions

The framework will follow least-privilege principles.

Key requirements:

- Each workflow declares only the permissions it needs.
- Consumer repositories retain control of their application secrets.
- Credentials are not hardcoded into workflow files.
- Cloud authentication should use GitHub OIDC where supported.
- Third-party GitHub Actions should be pinned to verified commit SHAs.
- Security failures should be visible and handled according to explicit policy.
- Production deployments should use protected GitHub Environments where appropriate.

The central framework should not require unrestricted access to consumer repositories.

Cross-repository access must be explicitly configured and limited to the required operations.

## 8. Workflow Inputs and Outputs

Reusable workflows should expose documented inputs and outputs.

Examples of potential inputs:

| Input | Purpose |
|---|---|
| `java-version` | Select the Java runtime version |
| `node-version` | Select the Node.js runtime version |
| `working-directory` | Identify the application directory |
| `build-command` | Configure application build execution |
| `test-command` | Configure test execution |
| `dockerfile` | Identify the Dockerfile |
| `image-name` | Define the container image name |
| `registry` | Select the container registry |
| `enable-security-scan` | Enable or disable an optional security capability |

Potential outputs:

- Build status.
- Artifact location or identifier.
- Image name and immutable digest.
- Security scan result.
- SBOM artifact identifier.
- Published image reference.

The final input and output contracts will be defined during implementation. Security-critical controls must not be bypassed merely by changing ordinary consumer inputs.

## 9. Versioning Strategy

The framework will use versioned releases.

Consumer repositories should reference a stable release or immutable commit rather than depending on a moving development branch.

A proposed versioning convention is:

- `v1.0.0` — initial stable release.
- `v1.1.0` — backward-compatible feature addition.
- `v1.1.1` — backward-compatible correction.
- `v2.0.0` — breaking interface or behavior change.

Changes to reusable workflow inputs, outputs, security defaults, and permissions must be reviewed for compatibility.

## 10. Failure Handling

Pipeline stages should fail clearly and provide useful diagnostic information.

Examples:

- Compilation failure stops the relevant build.
- Test failure prevents progression through dependent stages.
- Secret detection failures are handled according to the security policy.
- Critical vulnerability findings can block container publication.
- Registry publication failure prevents the deployment stage from proceeding.
- Deployment health failure is reported independently from CI success.

The framework should preserve relevant logs, test results, scan reports, and SBOM artifacts for investigation.

## 11. Initial Implementation Scope

The first implementation will focus on:

1. Reusable Java/Maven CI workflow.
2. Reusable Node.js CI workflow.
3. Shared security scanning capabilities.
4. Container build and security scanning.
5. SBOM generation and GHCR publication.
6. GitOps integration with the Baraza platform.
7. Validation using a second application repository.

Advanced capabilities such as provenance enforcement, automated rollback, and comprehensive release governance will be introduced incrementally.

## 12. Architectural Principles

The framework will follow these principles:

- Build reusable capabilities instead of duplicating application pipelines.
- Keep workflows focused and composable.
- Make inputs, outputs, and permissions explicit.
- Fail safely when required validation is unsuccessful.
- Separate CI validation from release and deployment.
- Use immutable artifact references wherever possible.
- Keep security controls configurable but governed.
- Document implementation decisions and operational behavior.

---

**Document status:** Initial architecture proposal.

**Repository:** `AndrewKMwangi/devsecops-pipeline-templates`

**Architecture validation:** Pending implementation and consumer-repository testing.