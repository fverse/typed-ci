# typed-gitlab-ci

Type-safe GitLab CI pipeline configuration using Pkl.

## Overview

`typed-gitlab-ci` provides a type-safe way to define GitLab CI pipelines using the Pkl configuration language. Write your `.gitlab-ci.yml` configurations with validation, and type checking.

## Installation

### Using Pkl CLI

Add `typed-gitlab-ci` as a dependency to your `PklProject`:

```pkl
amends "pkl:Project"

dependencies {
  ["typed-gitlab-ci"] {
    uri = "package://pkl-lang.org/typed-gitlab-ci@0.1.0"
  }
}
```

Then run:

```bash
pkl project resolve
```

### Direct Import

You can also import directly using the package URI:

```pkl
import "package://pkl-lang.org/typed-gitlab-ci@0.1.0/GitLabCI.pkl"
```

## Quick Start

Create a file `pipeline.pkl`:

```pkl
amends "package://pkl-lang.org/typed-gitlab-ci@0.1.0/GitLabCI.pkl"

output {
  renderer = new YamlRenderer {}
}

stages {
  "build"
  "test"
}

jobs {
  ["build-job"] {
    stage = "build"
    image = "node:18"
    script {
      "npm install"
      "npm run build"
    }
    artifacts {
      paths {
        "dist/"
      }
    }
  }
  
  ["test-job"] {
    stage = "test"
    image = "node:18"
    script {
      "npm install"
      "npm test"
    }
    needs {
      "build-job"
    }
  }
}
```

Render to YAML:

```bash
pkl eval -f yaml pipeline.pkl > .gitlab-ci.yml
```

## Examples

The `examples/` directory contains complete working examples:

- **basic-pipeline.pkl** - Simple build job with artifacts and caching
- **multi-stage.pkl** - Multi-stage pipeline with dependencies
- **deploy-pipeline.pkl** - Production deployment with environments

To try an example:

```bash
cd examples
pkl eval -f yaml basic-pipeline.pkl
```

## Validation

`typed-gitlab-ci` validates your configuration at evaluation time:

```pkl
// This will fail validation - invalid stage reference
jobs {
  ["my-job"] {
    stage = "nonexistent-stage"  // Error: Stage not defined in stages list
    script {
      "echo 'test'"
    }
  }
}
```

Common validations include:

- Stage names must be defined in the `stages` list
- Job dependencies (`needs`) must reference existing jobs
- Artifact paths must be valid
- Retry counts must be within allowed range (0-2)

## Advanced Usage

### Using Variables

```pkl
variables {
  ["NODE_VERSION"] = "18"
  ["DEPLOY_ENV"] = "production"
}

jobs {
  ["build"] {
    script {
      "node --version"
      "echo $DEPLOY_ENV"
    }
  }
}
```

### Job Dependencies

```pkl
jobs {
  ["build"] {
    stage = "build"
    script { "make build" }
  }
  
  ["test"] {
    stage = "test"
    needs { "build" }  // Runs immediately after build completes
    script { "make test" }
  }
}
```

### Conditional Execution with Rules

```pkl
jobs {
  ["deploy"] {
    stage = "deploy"
    script { "deploy.sh" }
    rules {
      new {
        `if` = "$CI_COMMIT_BRANCH == 'main'"
        when = "on_success"
      }
    }
  }
}
```

## License

MIT

