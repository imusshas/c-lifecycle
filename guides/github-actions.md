# Running Workflows with GitHub Actions

This guide explains how to configure a GitHub repository and run workflows using GitHub Actions.

A GitHub Actions workflow can automatically run commands on a temporary virtual machine provided by GitHub.

For example, a repository can run the same C program on:

```text
Windows
Linux
macOS
```

without requiring all three operating systems on your own computer.

---

## 1. Repository Structure

GitHub workflow files must be stored inside:

```text
.github/workflows/
```

For example:

```text
c-lifecycle/
│
├── hello.c
├── README.md
│
├── guides/
│   └── github-action.md
│
└── .github/
    └── workflows/
        ├── windows-test.yaml
        ├── linux-test.yaml
        └── mac-test.yaml
```

Workflow files normally use either:

```text
.yaml
```

or:

```text
.yml
```

---

## 2. Enable GitHub Actions

GitHub Actions is normally enabled for repositories.

To check:

```text
Repository
→ Settings
→ Actions
→ General
```

Under **Actions permissions**, make sure the repository is allowed to run the actions required by its workflows.

For a normal personal repository, the default configuration is usually sufficient.

---

## 3. Configure Git Authentication

If you push the repository using HTTPS and a Personal Access Token, the token must have sufficient permission.

Go to:

```text
GitHub
→ Profile Picture
→ Settings
→ Developer settings
→ Personal access tokens
```

There are two types of tokens:

```text
Fine-grained tokens
Tokens (classic)
```

### Using a Classic Token

If using a classic token for Git operations, commonly required scopes are:

```text
repo
workflow
```

`repo` allows repository operations.

`workflow` allows the token to create or modify files inside:

```text
.github/workflows/
```

Without the `workflow` scope, pushing a workflow file may produce an error similar to:

```text
refusing to allow a Personal Access Token to create or update workflow
without workflow scope
```

Do not give a token more permissions than necessary.

---

## 4. Create a Workflow

Create the workflow directory if it does not already exist:

```bash
mkdir -p .github/workflows
```

Then create a workflow file.

For example:

```bash
nano .github/workflows/linux-test.yaml
```

A minimal workflow looks like this:

```yaml
name: Linux Test

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Show operating system
        run: uname -a

      - name: Compile program
        run: gcc hello.c -o hello

      - name: Run program
        run: ./hello
```

---

## 5. Understand the Workflow Structure

Consider:

```yaml
name: Linux Test
```

This is the name shown in the GitHub **Actions** page.

---

The following:

```yaml
on:
  workflow_dispatch:
```

means:

```text
Run this workflow manually.
```

GitHub will provide a **Run workflow** button.

---

The following:

```yaml
jobs:
```

defines the jobs that the workflow will execute.

---

The following:

```yaml
runs-on: ubuntu-latest
```

selects the operating system.

Common options are:

```yaml
runs-on: ubuntu-latest
```

```yaml
runs-on: windows-latest
```

```yaml
runs-on: macos-latest
```

---

The following:

```yaml
steps:
```

contains the individual operations performed by the job.

For example:

```yaml
- name: Compile program
  run: gcc hello.c -o hello
```

executes:

```bash
gcc hello.c -o hello
```

on the GitHub-hosted machine.

---

## 6. Checkout the Repository

Most workflows should contain:

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

This copies the repository contents into the temporary machine.

Without this step, files such as:

```text
hello.c
```

will normally not be available to later commands.

---

## 7. Commit the Workflow

After creating or modifying the workflow:

```bash
git status
```

Add the files:

```bash
git add .github/workflows/
```

Commit:

```bash
git commit -m "Add GitHub Actions workflows"
```

Push:

```bash
git push
```

---

## 8. Run a Workflow

Open the repository on GitHub.

Go to:

```text
Repository
→ Actions
```

The workflows appear in the left sidebar.

For example:

```text
Test C Compilation on Windows
Test C Compilation on Linux
Test C Compilation on macOS
```

Select the workflow you want to execute.

Then click:

```text
Run workflow
```

Select the branch:

```text
main
```

and click:

```text
Run workflow
```

again.

---

## 9. Workflow Status

After starting the workflow, GitHub creates a workflow run.

Typical status indicators are:

```text
Yellow / Orange → Running
Green           → Successful
Red             → Failed
```

Click the workflow run to inspect it.

---

## 10. View Individual Steps

Inside the workflow run, select the job.

For example:

```text
test-c-lifecycle
```

You will see its steps:

```text
Set up job

Checkout repository

Check compiler

1 - Preprocessing

2 - Compilation

3 - Assembly

4 - Linking

Run executable

Save generated files

Complete job
```

Click any step to see its terminal output.

For example, if a step contains:

```yaml
run: |
  gcc -S hello.i -o hello.s
  cat hello.s
```

the GitHub Actions log will display the generated assembly code.

---

## 11. Running on Different Operating Systems

The runner determines which operating system GitHub provides.

### Windows

```yaml
runs-on: windows-latest
```

Example output executable:

```text
hello.exe
```

---

### Linux

```yaml
runs-on: ubuntu-latest
```

Example output executable:

```text
hello
```

---

### macOS

```yaml
runs-on: macos-latest
```

Example output executable:

```text
hello
```

Each job runs on a GitHub-hosted machine rather than on your own computer.

---

## 12. Save Generated Files

Files generated during a workflow disappear after the job finishes unless they are saved as artifacts.

For example:

```yaml
- name: Save generated files
  uses: actions/upload-artifact@v4
  with:
    name: c-lifecycle-output
    path: |
      hello.i
      hello.s
      hello.o
      hello
```

For Windows:

```yaml
- name: Save generated files
  uses: actions/upload-artifact@v4
  with:
    name: c-lifecycle-windows-output
    path: |
      hello.i
      hello.s
      hello.o
      hello.exe
```

---

## 13. Download Workflow Artifacts

After the workflow finishes:

```text
Repository
→ Actions
→ Select workflow
→ Select workflow run
```

Find the **Artifacts** section.

For example:

```text
c-lifecycle-linux-output
```

Click the artifact to download it.

The downloaded archive may contain:

```text
hello.i
hello.s
hello.o
hello
```

or, on Windows:

```text
hello.i
hello.s
hello.o
hello.exe
```

---

## 14. Update a Workflow

Modify the workflow file normally.

For example:

```bash
nano .github/workflows/linux-test.yaml
```

Then:

```bash
git add .github/workflows/linux-test.yaml
git commit -m "Update Linux workflow"
git push
```

Go back to:

```text
Repository
→ Actions
```

and run the workflow again.

---

## 15. Common Problem: No Workflow Found

If GitHub shows:

```text
Found 0 workflows
```

check that the workflow is located exactly inside:

```text
.github/workflows/
```

For example:

```text
.github/workflows/linux-test.yaml
```

Also make sure that the file has been:

```text
added
committed
pushed
```

to GitHub.

Check locally with:

```bash
git status
```

and:

```bash
git log --oneline
```

---

## 16. Common Problem: Workflow Is Only on the Local Computer

Creating:

```text
.github/workflows/linux-test.yaml
```

on your computer is not enough.

It must be pushed to GitHub:

```bash
git add .
git commit -m "Add workflow"
git push
```

Then verify it from the GitHub **Code** page.

You should be able to navigate to:

```text
.github
└── workflows
    └── linux-test.yaml
```

---

## 17. Common Problem: PAT Cannot Push Workflow

An error such as:

```text
refusing to allow a Personal Access Token to create or update workflow
```

usually means the token used for HTTPS Git authentication does not have sufficient workflow permission.

For a classic Personal Access Token, check:

```text
Settings
→ Developer settings
→ Personal access tokens
→ Tokens (classic)
→ Select token
```

Make sure the required permissions include:

```text
repo
workflow
```

Then try:

```bash
git push
```

again.

---

## 18. Common Problem: Workflow Fails

A red workflow does not necessarily mean GitHub Actions itself is broken.

Open:

```text
Actions
→ Failed workflow
→ Failed job
```

Find the step marked with:

```text
X
```

Expand that step.

The terminal output usually shows the actual error.

For example:

```text
gcc: command not found
```

means GCC is not available in that environment.

A workflow can install the required software before using it.

---

## 19. Common Problem: `Run workflow` Button Is Missing

For a manually executed workflow, make sure it contains:

```yaml
on:
  workflow_dispatch:
```

Also make sure the workflow has been pushed to the repository's default branch, normally:

```text
main
```

Then open:

```text
Repository
→ Actions
→ Select workflow
```

The **Run workflow** button should appear there.

---

## 20. GitHub Actions Terminology

### Workflow

The complete automation defined by a YAML file.

Example:

```text
linux-test.yaml
```

---

### Event

Something that starts a workflow.

For example:

```yaml
workflow_dispatch
```

means manual execution.

Other events can automatically start workflows after actions such as:

```text
push
pull request
scheduled time
```

---

### Job

A collection of steps executed on one runner.

Example:

```yaml
jobs:
  test:
```

---

### Runner

The machine that executes the job.

Examples:

```text
Windows runner
Linux runner
macOS runner
```

---

### Step

One operation inside a job.

For example:

```yaml
- name: Compile
  run: gcc hello.c -o hello
```

---

### Action

A reusable operation provided by GitHub or another developer.

For example:

```yaml
uses: actions/checkout@v4
```

---

### Artifact

A file produced by a workflow and saved so that it can be downloaded after the job finishes.

Examples:

```text
hello.i
hello.s
hello.o
hello.exe
```

---

## 21. C Lifecycle Repository Example

This repository uses three workflows:

```text
.github/workflows/
├── windows-test.yaml
├── linux-test.yaml
└── mac-test.yaml
```

Each workflow demonstrates:

```text
hello.c
   │
   ▼
Preprocessor
   │
   ▼
hello.i
   │
   ▼
Compiler
   │
   ▼
hello.s
   │
   ▼
Assembler
   │
   ▼
hello.o
   │
   ▼
Linker
   │
   ▼
Executable
```

The same C source can therefore be compiled and inspected on three different operating systems using GitHub-hosted runners.

---

## 22. Basic Workflow Process

The complete process is:

```text
Create repository
      │
      ▼
Create .github/workflows/
      │
      ▼
Create workflow YAML
      │
      ▼
git add
      │
      ▼
git commit
      │
      ▼
git push
      │
      ▼
GitHub → Actions
      │
      ▼
Select workflow
      │
      ▼
Run workflow
      │
      ▼
Inspect logs
      │
      ▼
Download artifacts
```
