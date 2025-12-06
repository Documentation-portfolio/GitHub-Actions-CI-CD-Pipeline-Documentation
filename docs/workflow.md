# CI/CD Pipeline Workflow 

import TOCInline from '@theme/TOCInline';

<TOCInline toc={toc} minHeadingLevel={2} maxHeadingLevel={4} />

The purpose of this document is to outline the workflow of the Continuous Integration and Continuous Deployment (CI/CD) pipeline for a sample application, defined in the GitHub Actions workflow file: `ci-cd.yml`.

### Workflow Overview
A workflow is an automated process that can run one or more jobs. A workflow runs when triggered mannually, at a scheduled time, or by an event in your repository. You can configure and set up a workflow in a repository to build, test, package, release, or deply a project on GitHub. 
Create a `.yaml` file to define your workflow configuration and save the workflow in the `.github/workflows` directory in your repository.

You can have multiple workflows in your repository to automate tasks such as build and test a pull request, or deploy an application every time a release is created.

### Workflow Triggers
A workflow trigger is an event that causes a workflow to execute. 

##### Examples 
The following workflow runs when a `push` event targets a branch named `main`.
```
on:
  push:
    branches: [ main ]
```
The following workflow runs when a `pull_request` event targets a branch named `main`.
```
on:
  pull_request:
    branches: [ main ]
```

### Workflow Structure: Jobs and Dependencies

A workflow consists of one or more jobs that run in parallel by default. The CI/CD pipeline uses dependencies to ensure sequential execution of the workflow jobs. The sequencing of jobs is defined through `needs:`.

| Job Name | `runs-on` | Dependencies (needs) | Description |
| --- | --- | --- | --- |
| `build` | `ubuntu-latest` | None | Compiles the code and creates an artifact. |
| `test` | `ubuntu-latest` | `build` | Runs unit and integration tests against the built code. |
| `deploy` | `ubuntu-latest` | `test` | Pushes the validated artifact to the staging environment. |

### Workflow Execution Details

**Job 1**: `build`
Sets up the enviornment, installs dependencies, and compiles the application.
The final step of the job uses action to save the compipled code for use in subsequent jobs.
##### Key Concepts
- Uses `actions/checkout@v4` to pull source code
- Uses `actions/setup-node@v4` to configure Node.js
- Uses `upload-artifact` to saves build output as an artifact `upload-artifact`
- Produces compiled build files under `dist/`
```
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install dependencies
        run: npm ci
      - name: Build application
        run: npm run build
      - name: Upload Build Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-dist
          path: dist/ # Path to the compiled output directory
```
**Job 2**: `test`
Consumes the artifact from the `build` job and runs tests against the compiled code.

##### Key Concepts
- Uses `needs: build` to enforce ordering
- Downloads artifact using `download-artifact`
- Tests run against build output intended for deployment
```
test:
  needs: build
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Download Build Artifact
      uses: actions/download-artifact@v4
      with:
        name: app-dist

    - name: Run Unit Tests
      run: npm test -- --coverage

```
**Job 3**: `deploy`
Deployment occurs only when:
- The workflow event is `push`
- The branch is `main`
- All previous jobs are successful

##### Key Concepts
- Uses `if` conditions to prevent PR deployments
- Deploys to a protected GitHub environment (`Staging`)
- Uses secret `DEPLOY_KEY` for secure authentication
- Downloads the artifact generated from the build job
```
deploy:
  if: github.event_name == 'push' && github.ref == 'refs/heads/main'
  needs: test
  runs-on: ubuntu-latest
  environment: Staging
  steps:
    - name: Download Artifact for Deployment
      uses: actions/download-artifact@v4
      with:
        name: app-dist

    - name: Deploy to Staging
      run: |
        ./deploy-script.sh --api-key ${{ secrets.DEPLOY_KEY }}
        echo "Deployment to staging complete!"
```
### Advanced Workflow Features

**Artifact Management**
Artifacts ensure consistency as the build job, the test job, and the deploy job use the same compiled output.

**Sequential Job Execution**
`needs:` enforces strict pipeline flow.

**Status Reporting**
After execution, the final status of the workflow (Success, Failure, Cancelled) appears on **Pull Requests**, **Commit** history, or **Actions** dashboard.