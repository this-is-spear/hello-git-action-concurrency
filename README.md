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

다음과 같은 결과가 실행된다.

|동시에 세 개 실행|세 개 중 두 개 실패|
|---|---|
|<img width="2032" alt="스크린샷 2025-04-02 오전 7 52 24" src="https://github.com/user-attachments/assets/24f83cbc-2b80-4cbe-be81-f8a6b1291331" />|<img width="2032" alt="스크린샷 2025-04-02 오전 7 53 42" src="https://github.com/user-attachments/assets/7a852b38-7127-4c85-8980-020735ef876b" />|

에러는 push 하기 전에 pull을 받아야한다는 메시지가 발생한다.

<img width="1470" alt="스크린샷 2025-04-02 오전 7 53 23" src="https://github.com/user-attachments/assets/579a625e-05fb-44e8-8945-e352ad577337" />

해결할 수 있는 방법은 두 가지다.

- 실패하면  push 될 때까지 pull 받는다.
- 동시성을 제어한다.

두 번째 방법을 진행해보겠다. git action 에서는 concurrency 키워드를 사용해 진행하면 된다.

```

jobs:
  build:
    ...
    concurrency:
      group: tis-hello-git-action-concurrency
      cancel-in-progress: false
```




\n테스트 진행합니다.
테스트 진행합니다.
테스트 진행합니다.
