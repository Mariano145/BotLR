# Commit Message Convention

This document defines the strict commit message convention for my project. Following this convention will help maintain a clear and consistent commit history, i must adhere to this convention for every commit integrated into the branches.

## 1. General Rule
Every commit message must follow this structure, based on Conventional Commits:
```
<type>[optional scope]: <description>
[optional body]
[optional footer(s)]
```

### 1.1 Structural Elements:
- **<type>**: A mandatory field that describes the type of change being made. Common types include:
  - `feat`: A new feature
  - `fix`: A bug fix
  - `docs`: Documentation changes
  - `style`: Code style changes (formatting, missing semi-colons, etc.)
  - `refactor`: Code changes that neither fixes a bug nor adds a feature
  - `test`: Adding or updating tests
  - `chore`: Changes to the build process or auxiliary tools and libraries
- **[optional scope]**: Provides additional context about the change, such as the affected module or component.
- **<description>**: A brief summary of the change, written in the imperative mood
- **[optional body]**: A more detailed explanation of the change, if necessary.
- **[optional footer(s)]**: Can include references to issues or breaking changes.

## 2. Valid examples:
### Example 1 (feat):
```
feat(auth): add login functionality
This commit introduces a new login feature that allows users to authenticate using their email and password. 
The login form has been added to the frontend, and the backend API has been updated to handle authentication requests.
```
### Example 2 (fix):
```
fix(api): resolve user authentication bug
This commit fixes a bug in the API that was preventing users from logging in successfully. 
The issue was caused by an incorrect validation check in the authentication endpoint.
```
### Example 3 (docs):
```
docs: update README with installation instructions
This commit updates the README file with clear and concise installation instructions for new users.
```
