# Git 자주 쓰는 명령어

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
