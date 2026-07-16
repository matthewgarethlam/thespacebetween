
## Useful Git Commands

### Setup

Initialise a new repository:

```bash
git init
```

Clone an existing repository:

```bash
git clone <repository-url>
```

Check your Git version:

```bash
git --version
```

---

### Commonly Used

Check repository status:

```bash
git status
```

See recent commits:

```bash
git log
```

Add all changed files:

```bash
git add .
```

Add a specific file:

```bash
git add myfile.py
```

Create a commit:

```bash
git commit -m "Add feature X"
```

---

### Branching

View branches:

```bash
git branch
```

Create a new branch:

```bash
git checkout -b feature/new-feature
```

Switch branches:

```bash
git checkout develop
```

Merge a branch:

```bash
git merge feature/new-feature
```

Delete a branch:

```bash
git branch -d feature/new-feature
```

---

### Pushing and Pulling

Download latest changes:

```bash
git pull
```

Upload commits:

```bash
git push
```

Push a new branch:

```bash
git push -u origin feature/new-feature
```

---

### Comparing Changes

Compare branches:

```bash
git diff main develop
```

---

### Undoing Mistakes

Undo changes to a file:

```bash
git checkout -- myfile.py
```

Unstage a file:

```bash
git reset myfile.py
```

Revert a commit safely:

```bash
git revert <commit-id>
```

---

### The "Save My Work Quickly" Commands

Temporarily save changes:

```bash
git stash
```

Restore stashed changes:

```bash
git stash pop
```

---

## "Do the dance"

This is my favourite. Used when I need to update branch without losing local changes: 

```bash
git stash #this stashes the changes in my current branch
git checkout main #checks out the main branch
git pull 
git checkout myoriginalbranch
git stash pop 
git merge main #brings in all the changes from main into current branch
```