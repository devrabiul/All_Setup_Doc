# Merge Branch into Master

If you're looking for a quick answer, to merge a branch into the master branch - you checkout master and merge some_branch:


```bash
git checkout new_branch
```

...develop some code...

```bash
git add .
git commit –m "Some commit message"
git push
git checkout main
```
Switched to branch 'main'

```bash
git merge new_branch
````
