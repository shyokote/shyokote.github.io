# Delete the entire contents of a Git repository

## 目的
Gitリポジトリの中身を全部一気に消す

## 方法
```bash
git clean -fdx && test $(git ls-files | wc -l) -eq 0 || git rm -rf .
```
