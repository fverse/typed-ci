# Typed CI

Type-safe, validated CI/CD pipeline configuration for GitLab CI and GitHub Actions using [Pkl](https://pkl-lang.org).

## Overview

`typed-ci` provides Pkl templates that allow you to write your CI/CD configurations in Pkl and then compile them (using the Pkl CLI) to YAML. Pkl will type-check, validate the configuration before it successfully compiles. This way you can configuration errors at evaluation time, not at runtime.

## Templates

### [typed-gitlab-ci](./packages/typed-gitlab-ci)

Write your GitLab CI pipelines in Pkl with full type safety and validation. Compile to YAML and catch errors before they break your pipeline.

```pkl
amends "package://pkl-lang.org/typed-gitlab-ci@0.1.0/GitLabCI.pkl"

stages { "build", "test", "deploy" }

jobs {
  ["build-job"] {
    stage = "build"
    script { "make build" }
  }
}
```

**[View Documentation →](./packages/typed-gitlab-ci/README.md)**

### [typed-github-actions](./packages/typed-github-actions)

Write your GitHub Actions workflows in Pkl with full type safety and validation. Compile to YAML and catch errors before they break your workflow.

```pkl
amends "package://pkl-lang.org/typed-github-actions@0.1.0/GitHubActions.pkl"

name = "CI"
on { push { branches { "main" } } }

jobs {
  ["build"] {
    `runs-on` = "ubuntu-latest"
    steps {
      new { uses = "actions/checkout@v4" }
      new { run = "make build" }
    }
  }
}
```

**[View Documentation →](./packages/typed-github-actions/README.md)**

## Getting Started

### Prerequisites

- [Pkl CLI](https://pkl-lang.org/main/current/pkl-cli/index.html) installed

### Installation

Choose the package for your platform and add it to your `PklProject`:

```pkl
amends "pkl:Project"

dependencies {
  ["typed-gitlab-ci"] {
    uri = "package://pkl-lang.org/typed-gitlab-ci@0.1.0"
  }
  // or
  ["typed-github-actions"] {
    uri = "package://pkl-lang.org/typed-github-actions@0.1.0"
  }
}
```

Then resolve dependencies:

```bash
pkl project resolve
```
For more information, check out the template-specific READMEs.

## License

MIT
