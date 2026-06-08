# Git Workflow

This document describes the Git workflow used in this project.

## 1. Branching Structure

The project follows a branching structure that includes the following branches:
### 1.1 `main`
- **Purpose**: The `main` branch is the stable branch that contains the production-ready code. All releases are made from this branch.
- **Protection**: The `main` branch is protected, meaning that direct commits are not allowed. All changes must go through a pull request process to ensure code quality and stability.
- **Rules**: Never commit directly to `main`.
### 1.2 `develop`
- **Purpose**: The `develop` branch is the integration branch where all feature branches are merged before being released to `main`.
- **Rules**: All feature branches must be merged into `develop` before being merged into `main`.
### 1.3 `feature/*`
- **Purpose**: Feature branches are used for developing new features or making significant changes.
- **Origin**: Feature branches are created from the `develop` branch.
- **Destination**: Once a feature is complete and tested, it is merged back into the `develop` branch.
---
## 2. Branch Nomenclature
All branches must follow a specific naming convention to maintain clarity and organization:
- **Feature Branches**: `feature/<short-description>`
### Valid examples:
```
feature/add-user-authentication
feature/improve-ui-design
feature/fix-login-bug
feature/update-api-endpoints
```
---
## 3. Workflow Steps
1. **Create a Feature Branch**: When starting work on a new feature, create a new branch from `develop` using the naming convention described above.
2. **Develop the Feature**: Make your changes in the feature branch. Ensure that you follow the commit message convention for all commits made in this branch.
3. **Merge to Develop**: Once the feature is complete and tested, create a pull request to merge the feature branch into `develop`.
4. **Merge to Main**: After all features for a release are merged into `develop`, create a pull request to merge `develop` into `main`. This should only be done when the code in `develop` is stable and ready for production.
5. **Release**: After merging `develop` into `main`, create a new release from the `main` branch.
---
## 4. Merge Criteria
Each pull request can only be merged in develop or main if it meets the following criteria:
- **Tests**: All tests must pass successfully.
- **Linter**: The code must pass all linting checks.
- **CI/CD**: GitHub Actions must complete successfully, ensuring that the code is ready for production.
---
## 5. Visual Representation
Here is a visual representation of the Git workflow:
```
┌─────────────────────────────────────────────────────────┐
│                    main (protected)                     │
│              Stable code, production ready              │
└─────────────────────────────────────────────────────────┘
                      ↑  CI/CD checks
                      │  + tests
                      │
┌─────────────────────────────────────────────────────────┐
│                  develop (protected)                    │
│                  Feature integration                    │
└─────────────────────────────────────────────────────────┘
   ↑                    ↑                    ↑
   │ pull request       │ pull request       │ pull request 
   │                    │                    │
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│feature/order-│    │ feature/fix- │    │feature/admin-│
│ notification │    │  login-bug   │    │    panel     │
└──────────────┘    └──────────────┘    └──────────────┘
```
---
## 6. Solo Development Note
Since this is a solo project, pull requests are self-reviewed.
Each PR must still pass all CI/CD checks before merging.
