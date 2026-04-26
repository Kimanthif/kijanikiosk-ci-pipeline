# KijaniKiosk Continuous Integration Pipeline: Board-Facing Overview

This document explains how our automated build pipeline works from the moment a developer updates the shared codebase to the moment a versioned package appears in our internal software registry. The process is intentionally simple, transparent, and predictable. Its purpose is to ensure that every software change entering our financial services platform is tested, validated, and recorded with full traceability. The pipeline reduces manual work, eliminates avoidable mistakes, and makes sure that every version we store can be trusted.

## How the Pipeline Starts

Any time a developer pushes new code to the central repository, the automated system immediately begins a review process. This review does not involve people; it is fully machine-driven. The idea is that quality checks happen the same way every time, regardless of who wrote the code or when they submitted it. This consistency is especially important for a financial environment where correctness and reproducibility matter.

Once triggered, the pipeline runs inside a controlled environment using a fixed version of Node.js. This locked-in environment eliminates “works on my machine” problems. Every run happens with the same tools, the same operating system, and the same configuration. This is a foundation for predictable results.

## What Each Stage Does

The pipeline includes several stages. Each stage answers one question before the system moves on to the next.

| Stage | Purpose | What It Confirms |
|-------|---------|------------------|
| **Lint** | Quick, automated review of code formatting and syntax | The code is clean, readable, and free of basic mistakes |
| **Build** | Creates the actual software output | The version is valid and the system can generate a functioning package |
| **Verify (Test + Security Audit)** | Runs tests and checks for high-risk vulnerabilities | The software behaves correctly and passes minimum security expectations |
| **Archive** | Packages and fingerprints the build output | We have a certified copy stored for long-term traceability |
| **Publish** | Uploads the versioned package to the Nexus registry | The artifact is available for downstream deployment |

Each stage moves us closer to a trusted, versioned output. If any stage raises a concern, the pipeline stops immediately so the issue can be corrected before the work goes further.

## Versioning and Traceability

Every package produced by the pipeline is assigned a version that includes both a human-friendly semantic version and a short code linked to the exact commit that produced it. This means that anyone reviewing an issue can trace a deployed artifact back to the exact line of code that generated it. The artifact is stored in Nexus, our internal registry, where we maintain at least two distinct versions. This proves that the versioning system produces unique outputs for each new change.

Version traceability is especially important for financial systems, where auditing and reproducibility are part of normal operational requirements. If a problem ever arises in production, we can quickly identify when the faulty code was introduced and which systems are using it.

## What Happens When Something Goes Wrong

If a developer introduces a problem at any point—whether a simple formatting error, a failing test, or an issue with the build—the pipeline stops immediately. This prevents flawed code from moving forward. The system then records exactly what failed and why. Earlier results remain visible, but later stages are intentionally skipped. This protects the integrity of the artifacts and makes debugging efficient. Since the environment is consistent from run to run, the developer can reproduce the issue locally, fix it, and push an update. The next pipeline run will pick up the correction and continue normally.

Stopping early also reduces wasted time. For example, if the code cannot pass a basic lint check, there is no reason to run tests or build anything. The pipeline applies the principle of “fail fast”: errors should be caught as early and cheaply as possible.

## What This Pipeline Does Not Yet Do

This pipeline handles building, validating, fingerprinting, and publishing artifacts, but it does not yet manage automated deployment into live environments. It does not run long-form security scans, performance tests, or compliance checks. These are part of the work planned for future weeks. For now, the focus is on ensuring that every version stored in our registry is clean, consistent, and ready for safe use in the next phase of the platform.