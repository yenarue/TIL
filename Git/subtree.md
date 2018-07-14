subtree
=====

# 개념


# 액션

## add

```bash
$ git subtree add --prefix=dist_test https://git.heroku.com/dragonwebapp-circleci2.git master
```
커밋메세지 추가됨

## push
git subtree push --prefix=dist https://git.heroku.com/dragonwebapp-circleci2.git master

### 강제 푸쉬(force push)는 안되는걸까?
git push heroku `git subtree split --prefix dist master`:master --force

## split

