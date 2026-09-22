## Reusable Workflow

A Reusable Workflow is defined as a complete YAML workflow file and triggered using `on: workflow_call`.

**1. The Reusable Workflow Definition (`.github/workflows/deploy.yml`)**
This file defines the inputs and secrets it requires to run the job.

```yaml
name: Reusable Deployment
on:
  workflow_call:
    inputs:
      environment_name:
        description: 'The environment to deploy to'
        required: true
        type: string
    secrets:
      DEPLOYMENT_TOKEN:
        description: 'Token required for deployment'
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Application
        run: echo "Deploying to ${{ inputs.environment_name }}..."
        env:
          TOKEN: ${{ secrets.DEPLOYMENT_TOKEN }}

```

**2. The Caller Workflow (`.github/workflows/main.yml`)**
This is your main pipeline that calls the reusable workflow as a separate job.

```yaml
name: Main Application Pipeline
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building the application..."

  # Calling the Reusable Workflow
  call-deployment-workflow:
    needs: build
    uses: my-org/my-repo/.github/workflows/deploy.yml@main
    with:
      environment_name: 'production'
    secrets:
      DEPLOYMENT_TOKEN: ${{ secrets.MY_PROD_TOKEN }}

```

---

## Composite Action

A Composite Action is defined in an `action.yml` file and bundles multiple steps together. Notice that every `run` step in a composite action **must** explicitly define the `shell`.

**1. The Composite Action Definition (`my-custom-action/action.yml`)**

```yaml
name: 'Setup and Test Node.js'
description: 'Installs dependencies and runs tests in one step'
inputs:
  node-version:
    description: 'The Node.js version to install'
    required: true
    default: '18'

runs:
  using: "composite"
  steps:
    - name: Setup Node
      uses: actions/setup-node@v3
      with:
        node-version: ${{ inputs.node-version }}
        
    - name: Install dependencies
      run: npm install
      shell: bash
      
    - name: Run tests
      run: npm test
      shell: bash

```

**2. The Caller Workflow (`.github/workflows/main.yml`)**
This main pipeline calls the composite action as a step within its own job.

```yaml
name: Main Application Pipeline
on: [push]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      # Calling the Composite Action
      - name: Run custom setup and test action
        uses: ./my-custom-action  # Path to the directory containing action.yml
        with:
          node-version: '20'

```
