# Git 자주 쓰는 명령어
## 커밋 내역 보기
```bash
git log
```

## 작업/스테이징 디렉토리에서 파일 상태 확인하기

```bash
git status
```

## 가장 최근의 commit 메시지 변경하기

```bash
git commit --amend
```

vim에서 커밋 메시지 수정 후 `:wq` 입력해 저장

## add한 파일 취소

```bash
git reset HEAD 파일명
git reset HEAD # 전부취소
```

## 최근 커밋 취소

```bash
git reset --soft HEAD^ # staged 상태로 파일은 보존하고 커밋만 취소
```

## 별칭 사용하기

```bash
git config --global alias.ci commit
```

`git ci`를 `git commit` 대신 사용가능

## 최근 N개의 커밋을 하나로 합치기
```bash
git rebase -i HEAD~3 # 최근 3개 커밋 합치기
```

명령어를 수행하면 뜨는 vi 창에 남겨둘 커밋 메시지만 빼고 `pick` -> `s`로 변경 후 `:wq`로 저장

```bash
pick 1ab3c21 README 수정 3
s 49b2ce1 README 수정 2
s f23ck2l README 수정 1
# ...
```

저장하고 나면 뜨는 vi창에서 원하는 커밋 메시지로 변경 후 `:wq` 저장

```bash
# 1st commit message:
README 수정

# ....
``` 

이후 `git push` 수행