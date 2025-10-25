# typed-github-actions

Type-safe GitHub Actions workflow configuration using Pkl.

## Overview 

`pkl-github-actions` provides a type-safe way to define GitHub Actions workflows using the [Pkl configuration language](https://pkl-lang.org). Write your workflow YAML files with validation, and type checking.

## Installation

### Using Pkl CLI

Add `typed-github-actions` as a dependency to your `PklProject`:

```pkl
amends "pkl:Project"

dependencies {
  ["typed-github-actions"] {
    uri = "package://pkl-lang.org/typed-github-actions@0.1.0"
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
import "package://pkl-lang.org/typed-github-actions@0.1.0/GitHubActions.pkl"
```

## Quick Start

Create a file `ci-workflow.pkl`:

```pkl
amends "package://pkl-lang.org/typed-github-actions@0.1.0/GitHubActions.pkl"

output {
  renderer = new YamlRenderer {}
}

name = "CI"

on {
  push {
    branches { "main" }
  }
  pull_request {}
}

jobs {
  ["build"] {
    `runs-on` = "ubuntu-latest"
    
    steps {
      new {
        name = "Checkout code"
        uses = "actions/checkout@v4"
      }
      
      new {
        name = "Setup Node.js"
        uses = "actions/setup-node@v4"
        with {
          ["node-version"] = "20"
        }
      }
      
      new {
        name = "Install and build"
        run = """
          npm ci
          npm run build
          """
      }
    }
  }
}
```

Render to YAML:

```bash
pkl eval -f yaml ci-workflow.pkl > .github/workflows/ci.yml
```

## Examples

The `examples/` directory contains complete working examples:

- **basic-workflow.pkl** - Simple CI workflow with build steps
- **matrix-build.pkl** - Multi-platform matrix build strategy
- **deploy-workflow.pkl** - Deployment workflow with environments

To try an example:

```bash
cd examples
pkl eval -f yaml basic-workflow.pkl
```

## Validation

`typed-github-actions` validates your configuration at evaluation time:

```pkl
// This will fail validation - step must have either 'uses' or 'run'
steps {
  new {
    name = "Invalid step"
    // Error: Must specify either 'uses' or 'run'
  }
}
```

Common validations include:

- Steps must have either `uses` (action) or `run` (command), not both
- Job dependencies must reference existing jobs
- Runner labels must be valid
- Matrix configurations must be properly structured

## Advanced Usage

### Matrix Builds

```pkl
jobs {
  ["test"] {
    `runs-on` = "ubuntu-latest"
    
    strategy {
      matrix {
        node { "18", "20", "22" }
        os { "ubuntu-latest", "windows-latest", "macos-latest" }
      }
    }
    
    steps {
      new {
        name = "Setup Node.js ${{ matrix.node }}"
        uses = "actions/setup-node@v4"
        with {
          ["node-version"] = "${{ matrix.node }}"
        }
      }
    }
  }
}
```

### Job Dependencies

```pkl
jobs {
  ["build"] {
    `runs-on` = "ubuntu-latest"
    steps {
      new { run = "make build" }
    }
  }
  
  ["test"] {
    `runs-on` = "ubuntu-latest"
    needs { "build" }  // Runs after build completes
    steps {
      new { run = "make test" }
    }
  }
}
```

### Conditional Execution

```pkl
jobs {
  ["deploy"] {
    `runs-on` = "ubuntu-latest"
    `if` = "github.ref == 'refs/heads/main'"
    
    steps {
      new {
        name = "Deploy to production"
        run = "./deploy.sh"
      }
    }
  }
}
```

### Using Secrets and Environment Variables

```pkl
jobs {
  ["deploy"] {
    `runs-on` = "ubuntu-latest"
    
    env {
      ["NODE_ENV"] = "production"
    }
    
    steps {
      new {
        name = "Deploy"
        run = "deploy.sh"
        env {
          ["API_KEY"] = "${{ secrets.API_KEY }}"
        }
      }
    }
  }
}
```

### Deployment Environments

```pkl
jobs {
  ["deploy"] {
    `runs-on` = "ubuntu-latest"
    
    environment {
      name = "production"
      url = "https://example.com"
    }
    
    steps {
      new {
        name = "Deploy to production"
        run = "./deploy.sh"
      }
    }
  }
}
```

## License

MIT
