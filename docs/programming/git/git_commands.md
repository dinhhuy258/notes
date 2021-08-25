# Git commands

## 1. Configuration

### Check git configuration

```
git config -l
```

### Setup git username, email

#### Repository configuration

```
git config user.name "Huy Duong"
git config user.email "huy.duongdinh@gmail.com"
```

#### Global configuration

```
git config --global user.name "Huy Duong"
git config --global user.email "huy.duongdinh@gmail.com"
```

## 2. Branch

### List branches

#### Local

```
git branch
```

#### Remote

```
git branch -r
```

### Rename branch

#### Local

```
git branch -m <new_branch_name>
```

#### Remote

```
git push origin --delete <old_branch_name>
git push origin -u <new_branch_name>
```

### Delete branch

#### Local

```
git branch -d <branch_name>
```

#### Remote

```
git push origin --delete <branch_name>
```

### Create branch

```
git branch -b <branch_name>
```

## 3. Revert

### Revert specific commit

```
git revert [--no-commit] <commit_id>
```

### Revert file to specific commit

```
git checkout <commit_id> -- <file>
```

## 4. Rebase

### Rebase interactive mode

```
git rebase -i <branch_name>
```

## 5. Submodule

### Add submodule

```
git submodule add <remote_url>

git submodule add <remote_url> <folder>
```

### Update submodule

```
git submodule update --init --recursive
```
