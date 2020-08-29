# 1. 환경세팅

## step 1
스타크래프트2를 다운받아줍니다. (용량이 많이 커서 시간이 꽤 걸립니다)  
https://starcraft2.com/ko-kr/

## step 2
의존성 패키지 설치를 해줍니다
```
pip install pysc2
```

## step 3
스타크래프트 강화학습 전용으로 설정된 맵들을 다운로드받고 적용합니다  
- https://github.com/Blizzard/s2client-proto#downloads 
  - 비밀번호는 'iagreetotheeula'으로 풀 수 있습니다.
  - 위 링크에서는 <code>래더 2019 시즌 3</code> 맵을 다운로드 받아줍니다
- https://github.com/deepmind/pysc2/releases/download/v1.2/mini_games.zip
  - 위 링크에서는 미니게임들을 다운로드 받아줍니다.  
- 위 두 맵 폴더를 다운로드 한 뒤에 <code>StarCraft II/Maps</code>폴더를 만들고 여기 안에 다운로드된 맵 폴더들을 넣어줍니다.

맵이 잘 로드되었는지 확인할 수 있다.
```
python -m pysc2.bin.map_list
```

## step 4
에이전트 테스트 GUI 분석또한 제공되는 것을 테스트 해볼 수 있다.  
만약 안돌아간다면 pygame을 설치해보자
- 에이전트 테스트
```
python -m pysc2.bin.agent --map Simple64
```
- 에이전트랑 대결하기 테스트
```
python -m pysc2.bin.play --map Simple64
```

## 추가 
발표를 위해 갑작스럽게 준비했는데 준비끝나고보니 난투맵이라고 머신러닝에 쓰기좋은게 있네요?? 오우 쉣;;;;;;;;
