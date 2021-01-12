- git init : Initialize repository
- .git : git repository
- git status : working tree status
- git status -s : git status보다 짧게 요약해서 상태를 보여주는 명령. 변경된 파일 많을 때 유용
- git add : add to staging area
- git commit : create version
- git commit -a : modified 상태 파일의 add 명령 생략하고 바로 커밋. (untracked는 커밋 X)
- git commit -am : 커밋 추가
- git commit -m : 커밋 수정
- git commit —amend : 가장 마지막에 저장했던 커밋 메시지 수정 (vi 에디터)
- git log : show version
- git log —stat
- git diff : show changes
- git log -p
- git checkout : 특정 버전으로 워킹트리 변경시키는 방법. commit id를 복사해 시간여행
- git checkout -b <브랜치명> <커밋 체크섬>: 특정 커밋에서 브랜치를 새로 생성하고 동시에 체크아웃까지 함
- git reset —hard (or —soft or —mixed) : remove version
- git revert : 버전을 삭제하지 않으면서 되돌리는 방법

    ex) ver 1~4가 있을 때 3으로 돌아가고 싶다면 git revert <ver4's commit id> 입력

- git pull : 원격 저장소의 변경 사항을 워킹 트리에 반영 (= git fetch + git merge)
- git fetch [원격저장소별명] [브랜치명] : 원격저장소의 브랜치와 커밋들을 로컬저장소와 동기화.  옵션 생략하면 모든 원격 저장소에서 모든 브랜치를 가져옴
- git merge 브랜치명 : 지정한 브랜치의 커밋들을 현재 브랜치 및 워킹트리에 반영
- git merge —abort : merge를 취소하는 명령
- git reset [파일명] (soft, mixed, hard) : 스테이지 영역에 있는 파일들을 언스테이징함. 워킹 트리 내용은 변경되지 않으며, 옵션을 생략할 경우 스테이지의 모든 변경사항을 초기화함
