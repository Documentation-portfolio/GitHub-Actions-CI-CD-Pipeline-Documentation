---
id: configuration-documentation
title: Pipeline Control and Configuration
sidebar_label: Configuration
---

<div class="github-toc">
  <strong>Table of contents</strong>
  <ul>
    <li><a href="#workflow-overview">Workflow Overview</a></li>
    <li><a href="#1-job-execution-configuration">1. Job Execution Configuration</a>
      <ul>
        <li><a href="#a-runner-environment-runs-on">A. Runner Environment</a></li>
        <li><a href="#b-parallel-testing-with-matrix-strategy">B. Matrix Strategy</a></li>
      </ul>
    </li>
    <li><a href="#2-step-configuration-and-data-flow">2. Step Configuration & Data Flow</a>
      <ul>
        <li><a href="#a-action-input-parameters-with">A. Action Input Parameters</a></li>
        <li><a href="#b-passing-data-between-steps-output-variables">B. Passing Data Between Steps</a></li>
      </ul>
    </li>
    <li><a href="#3-secure-and-dynamic-variable-management">3. Secure & Dynamic Variable Management</a>
      <ul>
        <li><a href="#example-using-scoped-variables-and-secrets">Example: Scoped Variables & Secrets</a></li>
      </ul>
    </li>
  </ul>
</div>

This document details the critical configuration settings used to precisely control the execution environment, input management, variable flow, and security for GitHub Actions CI/CD pipelines.

Mastering these configurations is essential for building **secure, scalable, and cost-optimized** CI/CD workflows.

---

## Workflow Overview

A **workflow** is an automated process in GitHub Actions that runs one or more jobs.  
Workflows are defined in `.yaml` files stored in:



A workflow can run when:

- Triggered manually  
- Scheduled  
- Initiated by repository events (push, pull_request, release, etc.)

You can define multiple workflows to automate tasks like building, testing, deploying, scanning security, and environment promotions.

---

## 1. Job Execution Configuration

Job configuration determines *where* and *how* tasks execute.

---

### A. Runner Environment (`runs-on`)

The `runs-on` keyword defines the machine image used for job execution.

| Configuration | Description | Example Use Case |
| --- | --- | --- |
| `ubuntu-latest` | Default GitHub-hosted Linux runner. Fast & cost-efficient. | General builds & tests. |
| `windows-latest` | GitHub-hosted Windows runner. | Windows-specific testing. |
| `self-hosted` | Custom runner managed by you. | Deployments inside private networks / IDP-restricted workloads. |

---

### B. Parallel Testing with Matrix Strategy

Matrix builds allow automated test coverage across multiple runtimes, OS combinations, and architecture variations.

**Key Concepts**

- Eliminates duplicated YAML  
- Automatically creates parallel jobs  
- Supports exclusions to optimize runtime costs

**Example**

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        node: [18, 20, 22]
        os: [ubuntu-latest, macos-latest]
        exclude:
          - os: macos-latest
            node: 18

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Run Tests
        run: npm test


