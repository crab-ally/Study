# Git & GitHub

## 작업 흐름

```
작업 폴더 → (git add) → 스테이징 → (git commit) → 로컬 저장소 → (git push) → 원격 저장소
```

---

## 최초 설정

```bash
git config --global user.name "이름"
git config --global user.email "이메일"
```

> ``--global`` : 이 PC 전체에 적용

---

## 로컬에 처음 폴더를 만들었을 때

```bash
git init                    # 현재 폴더를 Git 관리 시작
git add .                   # 현재 폴더의 모든 파일을 스테이징 영역에 올림
git commit -m "커밋메시지"   # 로컬 저장소에 저장
git remote add origin 원격저장소주소 # 원격 저장소 연결
git branch -M main          # 현재 브랜치 이름을 main 으로 변경
git push -u origin main     # 원격 저장소에 업로드
```

---

## GitHub에서 가져오기

```bash
git clone 원격저장소주소    # 빈 폴더에 원격 저장소 복제
git pull origin main    # 기존 폴더에 원격 가져오기
```

> **참고** `git clone`은 폴더가 완전히 비어 있어야 합니다 (`.git` 포함).

---

## 브랜치

> 브랜치는 파일 복사가 아니라 커밋을 가리키는 포인터입니다.  
> 브랜치를 전환하면 작업 폴더가 해당 브랜치 상태로 바뀝니다.

### 생성 · 이동

```bash
git switch -c 브랜치명      # 생성 후 이동 (권장)
git checkout -b 브랜치명    # 생성 후 이동 (구버전)
```

### 확인

```bash
git branch      # 로컬 브랜치 목록
git branch -r   # 원격 브랜치 목록
git branch -a   # 로컬 + 원격 전체
```

### 병합 (main 기준)

```bash
git switch main             # main으로 이동
git pull origin main        # main 최신화
git merge 브랜치명          # 브랜치 내용 병합
git push origin main        # GitHub에 반영
```

### 원격 브랜치 생성 및 push

```bash
git push -u origin 브랜치명 # 원격 브랜치 생성
```

### 삭제

```bash
git branch -d 브랜치명              # 로컬 삭제 (병합된 경우)
git branch -D 브랜치명              # 로컬 강제 삭제
git push origin --delete 브랜치명   # 원격 삭제
```

---

## 상태 확인

```bash
git status                  # 수정·스테이징·새 파일, 현재 브랜치 확인
git log --oneline --graph   # 커밋 기록을 한 줄 + 그래프로 보기 (종료 q)
git diff                    # add 전 변경사항 확인
git diff --staged           # add 후 변경사항 확인
```

---

## 되돌리기

### add 취소

```bash
git restore --staged 파일명 # 스테이징만 취소 (파일 내용 유지)
git reset HEAD 파일명       # 위와 동일 (구버전)
```

### commit 취소

```bash
git reset --soft HEAD~1     # 커밋 취소, 스테이징 유지
git reset --mixed HEAD~1    # 커밋 + 스테이징 취소
git reset --hard HEAD~1     # 커밋 + 스테이징 + 파일 전부 이전 커밋 상태로 되돌리기
```

### 파일 내용 되돌리기

```bash
git restore 파일명          # 마지막 커밋 이후 변경 내용 버리기
```

---

## 원격 저장소 관리

```bash
git remote -v                           # 연결된 원격 저장소 목록
git remote show origin                  # 원격 저장소 상세 정보
git remote set-url origin 새URL        # 원격 저장소 주소 변경
git remote remove origin                # 원격 저장소 연결 해제
```