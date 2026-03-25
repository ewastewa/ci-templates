# CI Pipeline Templates

Reusable GitHub Actions CI workflows for ROS 2 repositories. Covers linting, formatting, building, unit testing, integration testing, and simulation smoke testing.

## Available Workflows

| Workflow | File | Purpose |
|----------|------|---------|
| Lint & Format | `.github/workflows/lint-format.yml` | Runs clang-format, clang-tidy, ament_cpplint, ament_xmllint, and ruff |
| Build & Test | `.github/workflows/build-test.yml` | Builds the package in Docker and runs unit tests |
| Integration Test | `.github/workflows/integration-test.yml` | Runs launch_testing integration tests |
| Sim Smoke Test | `.github/workflows/sim-smoke-test.yml` | Launches nodes in simulation and verifies topic communication |

## Repository Structure

```
ci-templates/
  .github/workflows/   # Reusable workflow definitions
  docker/              # Shared Dockerfile templates
  config/              # Shared linter and formatter configs
```

## Shared Configs

The `config/` directory contains shared linter and formatter configurations.

- `.clang-format` - C++ formatting rules based on ROS 2 conventions
- `.clang-tidy` - C++ static analysis checks for ROS 2 code
- `.ruff.toml` - Python linting rules for launch files and nodes

To use these in your repo, copy the files to your project root or symlink them:

```bash
cp ci-templates/config/.clang-format .clang-format
cp ci-templates/config/.clang-tidy .clang-tidy
cp ci-templates/config/.ruff.toml ruff.toml
```

The lint-format workflow fetches these configs automatically from this repo at runtime.

## Usage

Add a caller workflow to your repository at `.github/workflows/ci.yml` that references these reusable workflows.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    uses: ewastewa/ci-templates/.github/workflows/lint-format.yml@main
    with:
      ros_distro: jazzy
      base_image: ghcr.io/launchpad-build/launchpad-ros2-jazzy:main

  build-test:
    needs: lint
    uses: ewastewa/ci-templates/.github/workflows/build-test.yml@main
    with:
      ros_distro: jazzy
      base_image: ghcr.io/launchpad-build/launchpad-ros2-jazzy:main

  integration-test:
    needs: build-test
    uses: ewastewa/ci-templates/.github/workflows/integration-test.yml@main
    with:
      ros_distro: jazzy
      base_image: ghcr.io/launchpad-build/launchpad-ros2-jazzy:main

  sim-smoke-test:
    needs: build-test
    uses: ewastewa/ci-templates/.github/workflows/sim-smoke-test.yml@main
    with:
      ros_distro: jazzy
      base_image: ghcr.io/launchpad-build/launchpad-ros2-jazzy:main
      launch_file: my_package/launch/sim.launch.py
      expected_topics: "/joint_states,/tf"
```

See `example-caller.yml` in this repo for a full example with concurrency, path filters, and cron scheduling.
