# Fault Injection Log

This table records the behavior of the pipeline when each major stage is intentionally broken, confirming correct stage isolation and failure handling.

| Stage Faulted | How the Fault Was Introduced | Observed Pipeline Behavior | Why This Behavior Is Correct |
|---------------|------------------------------|----------------------------|-------------------------------|
| **Lint** | Introduced a syntax error in `index.js` | Pipeline stopped immediately after Lint; Build, Verify, Archive, Publish did not run. | Linting is a fail-fast step; stopping early prevents wasting time on invalid code. |
| **Test (Verify)** | Modified test script to exit with code 1 | Test branch failed, causing the Verify stage to fail and the pipeline to stop before Archive/Publish. | Verification protects quality; no artifact should be archived or published if tests fail. |
| **Build** | Removed the `build` folder creation | Build stage failed; Verify, Archive, Publish were skipped. | Build is a prerequisite for all downstream work, so failure prevents producing invalid artifacts. |
| **Publish** | Provided an incorrect Jenkins credential ID | All earlier stages passed; Publish failed; post-failure block triggered. | Publishing must authenticate properly; failing here prevents an unverified or unauthorized deployment. |

All faults were reversed afterward and the pipeline returned to a green state.