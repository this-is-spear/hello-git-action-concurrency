# hello-git-action-concurrency


git action 실행시 동시성 제어가 필요할 수 있다.
대표적인 사례로 git action 내부에서 chcek out 이후 파일을 수정하고 push 이벤트가 발생할 때 원자적 연산이 일어나지 않는 경우를 의미한다.

동시성 제어 테스트를 위해 워크플로우 파일을 생성했다.

```
name: Test

on:
  push:
    branches: [ "main" ]
  workflow_call:


jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: update readme
        run: |
          echo "README.md 파일을 업데이트합니다..."
          sleep 5
          
          # README.md 파일이 존재하는지 확인하고, 없으면 생성
          if [ ! -f "README.md" ]; then
            echo "README.md 파일이 존재하지 않아 새로 생성합니다."
          fi
          
          # 새 README 내용으로 기존 파일 덮어쓰기
          echo "\n테스트 진행합니다." >> README.md

      - name: commit
        run: |
          # 변경 사항이 있는지 확인
          git add README.md
          if git diff --staged --quiet; then
            echo "README에 변경 사항이 없습니다."
            exit 0
          fi
          
          # 변경 사항이 있으면 커밋 후 푸시
          git commit -m "docs: README 자동 업데이트 [skip ci]"
          git push
```
\n테스트 진행합니다.
\n테스트 진행합니다.
