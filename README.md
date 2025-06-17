## Overview

This repo is a template for creating .NET repos. You can do this by clicking the **Use this template** button in the top right corner. The new repo will have **main** as the default branch.

## git

Both `.gitattributes` and `.gitignore` are adapted for .NET development.

## VS Code

This repo contains `launch.json` and `tasks.json` used to configure debugging and other tasks. These files make use of the [Command Variable](https://marketplace.visualstudio.com/items?itemName=rioj7.command-variable) extension to prompt the user for build configuration (`Release` or `Debug`) as well as which test project to launch.  

## .NET

There is a `Directory.Build.props` file for common MSBuild properties to avoid copy-pasting across different C# projects. Depending on your setup, you may want to add some additional `Directory.Build.props` files in sub-folders.

It is recommended to use centralized package management as provided by `Directory.Packages.props`. If you choose not to do so, you can delete this file.

There is an `.editorconfig` that follows the default style and naming rules recommended by Microsoft for the most part.

[`global.json`](https://learn.microsoft.com/en-us/dotnet/core/tools/global-json) allows you to define which .NET SDK version is used when you run .NET CLI commands. This is useful so commands use the same SDK version whether they run locally or in CI scenarios.

### Layout

It is recommended that you keep the solution file(s) in the root directory, non-test code in `./src`, test code in `./tests`.

All build outputs from all projects are saved in `./artifacts/bin`, separated by project. For more information, see [Artifacts output layout](https://learn.microsoft.com/en-us/dotnet/core/sdk/artifacts-output).

## Dependabot

There is a config file for Dependabot to manage dependency updates for Nuget packages, GitHub actions and .NET SDK versions. Please modify this file as needed.

See [Dependabot options reference](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference) for details about config options.

## CI

There are several GitHub workflows for CI defined. Feel free to modify them as needed.

- If you wish to change the default branch from **main**, please update the Github workflows accordingly:

    ```yaml
    on:
      push:
        branches: [main] # Change to new name

      pull_request:
        branches: [main] # Change to new name
    ```

- Although each workflow runs fairly quickly, you may wish to combine some of them into a single workflow, e.g. on pushing to a branch, a single workflow that builds the code, performs SAST and runs tests (in that order) may be desired rather than having separate workflows for each task.

- These 2 GitHub Actions are used in some of the workflows:

    1. [`fkirc/skip-duplicate-actions`](https://github.com/fkirc/skip-duplicate-actions/tree/v5/): This prevents duplicate runs of the same workflow. This can happen for example if a workflow is triggered by more than one of the following events: `pull_request`, `push` and `workflow_dispatch`.

    2. [`dorny/paths-filter`](https://github.com/dorny/paths-filter): This lets you filter based on path. This is preferable to the builtin [path filters](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#onpushpull_requestpull_request_targetpathspaths-ignore) because they allow more flexibility -- you can use them to conditionally execute steps or jobs whereas the latter only work at the workflow level. One other nice thing is you can look at the step output to see which files were affected if any:

        ```text
        Run dorny/paths-filter@v3
        Get current git ref
        ...
        Detected 2 changed files
        Results:
        Filter code = false
        Matching files: none
        ...
        ```

- Some workflows require [creating repository secrets](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions).

- Also, some workflows update specific labels. Feel free to edit the labels in the workflows. If you wish to use the ones specified, you can manually create them or better yet, clone them from this repo:

    ```shell
    gh label clone AjayKMehta/template-dotnet --repo {UserName}/{Repo}
    ```

> [!NOTE]
> This requires the GItHub CLI app to be installed.

> [!TIP]
> If you are in the repo's working directory, you can omit `--repo {UserName}/{Repo}`.

### Workflows

#### Security + Linting

##### 🔍 *CodeQL Analysis*

Automated code analysis to identify vulnerabilities and coding errors in code.

**File**: [codeql-analysis.yml](.github/workflows/codeql-analysis.yml)

**Notes**:

> ℹ️ See [**Customizing your advanced setup for code scanning**](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning) for more details.<br />
> ℹ️ There are [3 build modes](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages#codeql-build-modes) available: `none`, `autobuild` or `manual`. In the workflow, build mode is set to `manual` but you may wish to change this as needed (see [**Building C#**](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages#building-c)).
>
> ❗There are some steps that are a workaround for an issue I have encountered with Dependabot not updating NuGet dependencies properly. You may wish to remove them if they are not applicable or no longer necessary *after testing the workflow without them*.

##### 📦 *Dependency Review*

Scans pull requests for dependency changes[^dep], highlighting security vulnerabilities and licensing issues.

[^dep]: Skips if PR is from dependabot.

**File**: [dependency-review.yml](.github/workflows/dependency-review.yml)

**Notes**:

>⚠️ Fails PR for high severity vulnerabilities.<br />
>ℹ️ Adds summary to PRs.<br />
>ℹ️ Uses dependency caching.

##### 🛡️ *Semgrep*

Performs static analysis security testing (SAST) to detect security vulnerabilities and bugs.

**File**: [semgrep.yml](.github/workflows/semgrep.yml)

**Notes**:

> ❗Requires setting up Semgrep account and saving Semgrep token as `SEMGREP_APP_TOKEN` secret for repository.<br/>
> ℹ️ For more information, please refer to [online help](https://semgrep.dev/docs/deployment/add-semgrep-to-ci).

#### PRs

##### 💥 *Label Merge Conflicts*

Automatically labels PRs that have merge conflicts with `💥 Conflicts` label.

**File**: [label_merge_conflicts.yml](.github/workflows/label_merge_conflicts.yml)

**Notes**:

> ℹ️ Retries up to 5 times with 5 second wait between retries

##### 🏷️ *Pull Request Labeler*

Automatically applies labels to pull requests based on the files changed.

**File**: [labeler.yml](.github/workflows/labeler.yml)

**Notes**:

> ℹ️ Labels are defined in [this config file](.github\labeler.yml).

##### 🔄 *Update PR*

Automatically updates PR branches when the **main** branch changes to prevent stale PRs.

**File**: [update_pr.yml](.github/workflows/update_pr.yml)

**Notes**:

> ❗Requires [saving GitHub app token or personal access token](https://github.com/adRise/update-pr-branch/pull/39) as `ACTION_USER_TOKEN` secret for repository.<br />
> ℹ️ Requires checks to pass.<br />
> ℹ️ Allows ongoing checks.

#### Tests

##### ✅ *Tests*

Runs tests, generates code coverage reports, and uploads code coverage results to [Codecov](https://www.codecov.io/).

**File**: [test.yml](.github/workflows/test.yml)

**Notes**:

> ❗Requires CodeCov account and token that is stored in a repository secret called `CODECOV_TOKEN`.<br />
> ℹ️ See [Codecov Tokens](https://docs.codecov.com/docs/codecov-tokens) for more information.<br />
> ℹ️ Adds coverage summary to PR and workflow runs.

#### Dependency management

##### ⬆️ *Version Sweeper*

Checks monthly for outdated .NET versions and creates automated upgrade PRs if needed.

**File**: [version-sweeper.yml](.github/workflows/version-sweeper.yml)

### Workflow Triggers

| Workflow              | Push | PR | Schedule | Manual |
|-----------------------|------|----|----------|--------|
| CodeQL Analysis       | ✅    | ✅  | -        | ✅      |
| Dependency Review     | ✅    | ✅  | -        | ✅      |
| Label Merge Conflicts | ✅    | -  | -        | -      |
| PR Labeler            | -    | ✅  | -        | -      |
| Semgrep               | ✅    | ✅  | Weekly   | ✅      |
| Tests                 | ✅    | ✅  | -        | -      |
| Update PR             | ✅    | -  | -        | -      |
| Version Sweeper       | -    | -  | Monthly  | ✅      |
