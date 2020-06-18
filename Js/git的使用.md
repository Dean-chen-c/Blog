git init
git add
git commit -m
git status
git diff
git log
git reflog
git reset --hard commit_id

git revert --no-commit 82536..HEAD
git commit -m "Reverting those bad changes I accidentally pushed and made public"

git checkout -- <file>
git reset HEAD <file>可以把暂存区的修改撤销掉

git rm <file>



git push --force 强制push