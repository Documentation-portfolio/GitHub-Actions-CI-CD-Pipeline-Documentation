# Troubleshoot Workflows
## Table of Contents
- [1. Troubleshooting Flow](#1-troubleshooting-flow)
- [2. Common Failure Scenarios and Resolutions](#2-common-failure-scenarios-and-resolutions)
- [3. Advanced Debugging Techniques](#3-advanced-debugging-techniques)

The purpose of this document is to guide you through failed workflows and help you identify failure sources, interpret logs, and apply fixes. 

The primary tool for workflow failure analysis is **Run Log**, **Status Check**, and **job-level step outputs**.

### Troubleshoot Workflows: Action Logs and Status Checks

#### Step 1: Identify the failed job.
- Open the workflow run.
- In the **Summary** tab, view the failed job with a red X.
- To view the individual steps, expand the job.
#### Step 2: Locate the failed step.
- Within the failed job, find the failed step with a red 'X'.
- Click to expand logs for the failed step.
#### Step 3: Analyze the error.
Analyze: 
- Stack traces
- Error codes
- Runtime exceptions
- Exit codes (For example, exited with code 1)

>**Tip:**The root cause of failure is mostly in the last 5-10 log lines before the failure message.

Example signals:
- *Error: Cannot find module...*
- *npm ERR! code ELIFECYCLE*
- *Process completed with exit code 1*

### Common Failures and Resolutions
The following table lists the common GitHub Actions workflow failures and resolutions:

| Failure Scenario                    | Log Indication (Example)                                                                                   | Resolution Steps                                                                                                                                                                                                       |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Dependency Installation Failure** | `npm ERR! code ELIFECYCLE` <br> `ERROR: Could not find a version that satisfies the requirement...`        | • Validate package installation command (`npm ci`, `pip install -r requirements.txt`). <br>• Ensure correct runtime version in `actions/setup-node`, `actions/setup-python`, etc. <br>• Check for corrupted lockfiles. |
| **Test or Linter Failure**          | `Failing tests: 2` <br> `Linter found 10 errors.` <br> `Exited with code 1`                                | • Review test output for assertion failures. <br>• Fix code quality violations. <br>• Re-run locally to confirm.                                                                                                       |
| **Artifact Not Found**              | `Error: The artifact 'app-dist' could not be downloaded.` <br> `Unable to find any artifacts for the run.` | • Ensure **artifact name matches** in upload and download steps. <br>• Confirm the upload step ran successfully. <br>• For matrix jobs, validate artifact naming per instance.                                            |
| **Secret / Authentication Error**   | `Error 403: Forbidden` <br> `Unauthorized` <br> `Authentication failed for...`                             | • Verify secret exists in repository or environment. <br>• Check correct syntax (`${{ secrets.API_KEY }}`). <br>• Ensure job uses the correct environment (required for protected secrets).                                     |
| **Resource Constraints**            | `No space left on device` <br> `Out of memory error`                                                       | • Switch to a larger hosted runner (example, from `ubuntu-latest` to `ubuntu-24.04` or self-hosted). <br>• Clear cache, reduce logs, and compress build output. <br>• Split heavy steps into smaller tasks.                         |

### Advanced Debugging Techniques
Use the following techniques for intermittent or hard-to-reproduce workflow failures.

#### Allow Verbose Step Debugging
GitHub Actions support internal debug logs. The verbose step helps expand tool invocation logs, internal error messages, and more detailed environment information.
1. Select **Repo Settings**, select **Secrets**, and then select **Actions**.
2. Add a secret named:
```
ACTIONS_STEP_DEBUG = true

```
3. Re-run the workflow.

>**Important:** Remove the secret after debugging to reduce log noise.

#### Insert Diagnostic Steps
Add a temporary diagnostic step before a suspicious step to inspect environment and filesystem state. The diagnostic steps helps validate file paths, confirm environment variable availability, and debug missing dependencies or build artifacts.
```
- name: Debug Environment and Filesystem
  if: always()
  run: |
    echo "Directory Structure:"
    ls -la
    echo "Environment Variables:"
    env | grep API
```
#### Use Conditional Steps for Cleanup
When failure occurs, you may need cleanup tasks such as remove temp files, notify systems, delete preview resources. The conditional steps ensure no partial state is left behind and external systems are not polluted with incomplete preview builds or stale resources.
Use:
```
if: always()
```
Example:
```
- name: Cleanup temporary files
  if: always()
  run: rm -rf temp/

```

