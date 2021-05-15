---
title: 깃 커맨드 모음 (Git CLI)
tags: Git
---

## git Config

git config --list : 리포지토리의 깃 설정값들을 전부 보여준다.

git config <option> : option에 해당하는 깃 설정값을 설정한다.

git config --global <option> : option에 해당하는 깃 전역설정값을 설정한다.

git config --unset <option> : option에 해당하는 설정을 지운다.

git config --unset --global <option> : option에 해당하는 깃 전역설정을 지운다.

# git add

git add <file> : 어떤 파일을 커밋할지 등록한다.

git add . : gitignore에 정의된 파일을 제외한 모든 파일을 git에 추가한다.

# git commit

git commit -m 'comment' : 변경사항을 모두 커밋하고, comment라는 코멘트를 남긴다.

git commit

# git push

git push origin master : origin master 브랜치에 커밋한다.
