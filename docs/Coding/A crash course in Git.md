# A Crash Course in Git

## What is Git?

Git is a piece of software on your machine used to track changes in files. Git let's us: 

- See what's changed, when and by whom
- Compare different versions of your work
- Collaborate with other people without overwriting each other's changes
- Maintain a complete audit trail of a project
- Revert mistakes


### Key Terms

| Term | Definition |
|--------|------------|
| Repository (Repo) | A project tracked by Git |
| Commit | A snapshot of changes at a point in time |
| Branch | An isolated line of development |
| Merge | Combining changes from different branches |
| Clone | Creating a local copy of a repository |
| Push | Uploading changes to a remote repository |
| Pull | Downloading changes from a remote repository |

---

## Why Should I Use Git?

Git is good for tracking changes with:

- Scripts (Python,R,SQL)
- Notebooks (Rmd, Jupyter Notebooks)
- FME workspaces
- Configuration files
- Documentation

### 1. Change History

Every modification is recorded with:

- What changed
- Who changed it
- When it changed
- Why it changed (through commit messages - free text from the author)


### 2. Rollbacks

Git allows you to quickly return to a previous working version without manually restoring files.

### 3. Safe Experimentation

Create a branch and experiment freely without altering the main/production branch. 

### 4. Better Collaboration

Git solves the classic problem of multiple people editing the same files. Git helps coordinate changes.

---

## Git and GitHub

???+ question 
    "What's the difference between Git and GitHub?"

**Git** is the actual version control software installed on your machine.

It manages version history, branches, commits and merges. 

Git works perfectly well without an internet connection.

**GitHub** is a cloud platform built around Git.

It is a platform where repositories can be hosted, developers can collaborate, Pull Requests (PRs) and Issues can be rasied and tracked and other Continuous Integration /Continuous Development (CI/CD) apabilities. 

An analogy:

| Git | GitHub |
|------|---------|
| Engine | Car |
| Version control technology | Collaboration platform |
| Works locally | Lives online |
| Tracks changes | Helps teams work together |

Other Git hosting platforms exist too:

- GitHub
- Azure DevOps
- GitLab
- Bitbucket

### Standard Workflow

```text
Clone Repository
       ↓
Create Branch
       ↓
Make Changes
       ↓
Commit Changes
       ↓
Push Branch
       ↓
Open Pull Request
       ↓
Code Review
       ↓
Merge into Main
```

---

## Git Flow

A branching strategy that defines how code moves from development into production.

### Core Branches

#### main

Contains production-ready code.

Anything in `main` should be QA'ed, stable and deployable.

#### develop

Contains the latest development work.

Features are merged here before eventually being released.

### Supporting Branches

#### Feature Branches

Used for developing new functionality.

Example:

```text
feature/add-tracking-module
feature/add-health-classification
```

#### Release Branches

Created when preparing a release.

Used for:

- Final testing
- Documentation updates
- Minor fixes

Example:

```text
release/v2.0
```

#### Hotfix Branches

Created from production when urgent issues occur.

Example:

```text
hotfix/fix-critical-bug
```

### Git Flow Diagram

```text
main
 │
 ├───────────────┐
 │               │
 │           hotfix/*
 │               │
 │               ▼
 └───────────────●

develop
 │
 ├── feature/*
 ├── feature/*
 └── feature/*

       │
       ▼

    release/*
       │
       ▼

      main
```

???+ question 
    When Should You Use Git Flow?

Git Flow works well when:

- Multiple developers contribute
- Formal releases are required
- Production stability is important

For smaller projects or on projects where you are the sole developer, simpler workflows such as are often faster and easier to manage.
