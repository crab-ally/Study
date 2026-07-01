# Git & GitHub 명령어 정리

## 작업 흐름

```
작업 폴더 → (git add) → 스테이징 영역 → (git commit) → 로컬 저장소 → (git push) → 원격 저장소
```

---

## 1. 최초 설정

```bash
git config --global user.name "이름"
git config --global user.email "이메일"
```

> `--global`: 이 PC에서 Git을 사용할 때 항상 적용되는 전역 설정

---

## 2. 저장소 시작하기

### 로컬에서 새로 시작

```bash
git init                              # 해당 폴더 Git 관리 시작
git add 파일명                        # 특정 파일 스테이징
git commit -m "커밋메시지"             # 로컬 저장소에 저장
git remote add origin <원격저장소주소>  # 원격 저장소 연결
git branch -M main                    # 브랜치명을 main으로 변경
git push -u origin main               # 원격 저장소에 업로드
```

> 로컬에 새 폴더 생성 시 원격에 새 저장소 생성 후 진행

### GitHub에서 가져오기

```bash
git clone <원격저장소주소>     # 새 폴더 생성 + 복제
git clone <원격저장소주소> .   # 현재(빈) 폴더에 복제
git pull origin main         # 기존 폴더에 최신 내용 반영 (fetch + merge)
```

---

## 3. 브랜치

커밋을 가리키는 포인터. 전환 시 작업 폴더가 해당 상태로 바뀜.


**생성 + 이동**

```bash
git switch -c <브랜치명>    # 권장
git checkout -b <브랜치명>  # 구버전
```

**목록 확인**

```bash
git branch        # 로컬 브랜치 목록
git branch -r     # 원격 브랜치 목록
git branch -a     # 전체(로컬+원격) 목록
```

**삭제**

```bash
git branch -d <브랜치명>             # 병합된 로컬 브랜치 삭제
git branch -D <브랜치명>             # 로컬 강제 삭제
git push origin --delete <브랜치명>  # 원격 브랜치 삭제
```

**원격 브랜치 생성**

```bash
git push -u origin <브랜치명>  # 원격에 새 브랜치 생성 + push
```

**병합 (main 기준)**

```bash
git switch main        # main 브랜치로 이동
git pull origin main   # main 브랜치 업데이트
git merge <브랜치명>    # 병합
git push origin main   # 원격 저장소에 업로드
```

**로컬에 없는 원격 브랜치 가져오기**

```bash
git fetch             # 원격 최신 내용 가져옴 (로컬 반영 X)
git switch <브랜치명>
```

**로컬에서 삭제된 원격 브랜치 정보 정리**

```bash
git fetch --prune
```

---

## 4. 임시 저장 (stash)

현재 작업 중인 수정사항을 임시로 숨겨두는 기능

```bash
git stash                       # 저장
git stash list                  # 목록 확인
git stash pop                   # 최근 복원 (적용 + 목록 삭제)
git stash apply stash@{index}   # 특정 복원 (목록 유지)
git stash drop                  # 최근 삭제
git stash drop stash@{index}    # 특정 삭제
git stash clear                 # 전체 삭제
```

## 5. 상태 확인

```bash
git status                  # 수정/스테이징/새 파일, 현재 브랜치
git log --oneline --graph   # 커밋 기록 (종료: q)
git diff                    # add 전 변경사항
git diff --staged           # add 후 변경사항
```

## 6. 되돌리기

### 파일 단위

```bash
git restore <파일명>                                    # 마지막 로컬 커밋 상태로 되돌리기
git fetch origin && git reset --hard origin/<브랜치명>  # 전체 파일을 원격 상태로
```

### add 취소

```bash
git restore --staged <파일명>   # 스테이징만 취소 (내용 유지)
git reset HEAD <파일명>         # 구버전
```

### commit 취소

```bash
git reset --soft HEAD~1    # 커밋 취소
git reset --mixed HEAD~1   # 커밋 + 스테이징 취소
git reset --hard HEAD~1    # 커밋 + 스테이징 + 작업 내용 취소
```

### 안전하게 취소 (새 커밋 생성)

기존 commit을 삭제하는 것이 아니라, 변경사항을 취소하는 새로운 commit을 생성한다.

```bash
git revert HEAD            # 최근 커밋 취소
git revert HEAD~1          # 최근 이전 커밋 취소
git revert <commit_hash>   # 특정 커밋 취소
```
> commit hash는 `git log --oneline --graph`로 확인

## 7. 원격 저장소 관리

```bash
git remote -v                      # 연결된 원격 저장소 목록
git remote show origin             # 상세 정보
git remote set-url origin <새URL>  # 주소 변경
git remote remove origin           # 연결 해제
```

---

## 8. 원격과 로컬

### rebase

```bash
# 기존
원격: A - B - C
로컬: A - B - D

git pull origin main
결과: A - B - C와 D가 합쳐진 Merge 커밋

# rebase 사용
원격: A - B - C
로컬: A - B - D

git pull --rebase origin main
결과: A - B - C - D
```

---

## 9. gitignore

해당 폴더에 파일(.gitignore) 생성 → 무시할 파일/폴더 경로 지정

**주요 카테고리**

1. 보안 및 환경 설정 파일
2. 의존성 패키지 폴더
3. 빌드 결과물 및 컴파일된 코드
4. 운영체제(OS) 및 개발 도구(IDE) 설정 파일
5. 로그 및 임시 파일