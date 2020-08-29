# 미니게임환경 만들기
본격적으로 미니게임 환경을 만들어봅시다.
> 강화학습 과정은 포함하고 있지 않습니다..! (only 환경)
일반적으로는 agent를 만든 이후에 터미널로 agent를 실행하는 방식을 편하게 사용하지만
강화학습 환경을 만들기 위해서 env를 코드 내부에서 실행하고 reward를 조정하게 하는게 
처음 접하기에 더 직관적이고 편할것 같다고 생각했다.

## step 0
https://github.com/deepmind/pysc2/blob/master/docs/mini_games.md
pysc2 미니게임에 대한 간단한 설명이다. (상당히 불친절하다)

## step 1
우선 import는 아래와 같습니다.
```
from pysc2.env import sc2_env, run_loop         # sc2환경을 코드내부에서 실행시키기 위한코드
from pysc2.agents import base_agent             # agent를 만들기 위한 부모클래스
from pysc2.lib import actions, features, units  # actions: 행동타입, features: 게임자체의 데이터 (게임끝나고 결과창 분석같은), units: 유닛식별용 열거형
from pysc2.tests import utils                   # utils: 싱글플레이 테스트 + 디버거

from absl.testing import absltest               # 검색해봤는데 자세히는 뭔지모르겠고 약간 종속성을 제거한 실행기?? 그런느낌인가??????

import numpy as np
import random
import time
```

아래는 agent 실행기
```
class TestEasy(utils.TestCase):
    steps = 2000
    step_mul = 16

    def test(self):
        with sc2_env.SC2Env(
                map_name="DefeatZerglingsAndBanelings",
                players=[sc2_env.Agent(sc2_env.Race.terran)],
                agent_interface_format=sc2_env.AgentInterfaceFormat(
                    action_space=sc2_env.ActionSpace.RAW,  # or: use_raw_actions=True,
                    use_raw_units=True),
                step_mul=self.step_mul,
                game_steps_per_episode=100 * self.steps * self.step_mul) as env:

            agent = MyAgent()
            run_loop.run_loop([agent], env, self.steps)

        self.assertLessEqual(agent.episodes, agent.reward)
        self.assertEqual(agent.steps, self.steps)


if __name__ == "__main__":
    absltest.main()
```
위 코드를 실행하면 test함수가 실행됩니다. (아직 agent를 만들지 않아서 바로실행하면 에러남, MyAgent()가 다음스텝에서 만들 agent클래스
- steps: 몇번돌릴지, step_mul: 몇 프레임마다 받아올지(였던 것 같습니다..)
- SC2Env: env환경설정후 맵을 실행한다
  - actionspace.raw & use_raw_unit: pysc2에는 기본실행환경과 raw실행환경이 있는데 raw실행환경에서는 캐릭터의 고유키와 캐릭터의 x,y좌표같은 화면이외의 데이터상으로 볼 수 있는 데이터들을 제공한다. (강화학습에 유리하다는 뜻)
맵은 저글링과 맹독충 vs 해병 으로 설정했습니다.

## step 2
Agent 생성의 기본 구조에 대하여 알아봅시다.
```
class MyAgent(base_agent.BaseAgent):
    
    def step(self, obs):
        super(MyAgent, self).step(obs)
        return actions.FUNCTIONS.no_op()
```
agent의 실행 최소 요건입니다. agent를 실행할때 step_mul에서 설정한 프레임마다 step이 호출되고 그 게임정보가 매개변수(obs)로 들어옵니다.  
step함수에서는 action을 리턴하며 한번에 1개의 action밖에 수행하지 못하는 것을 감안하고 강화학습코드를 짜야합니다.  
해결방법으로는 step_mul을 줄여서 빠르게 순서대로 action을 수행하게 만들면 어느정도 해결된다고 합니다.
<code>actions.FUNCTIONS.no_op()</code>는 코드그대로 아무것도 수행안하고넘기는것입니다.

## step 3
이제 MyAgent가 state를 이용해서 action을 수행하는 것을 만들어 봅시다.
```
    def step(self, obs):
        super(MyAgent, self).step(obs)
        marines = [unit.tag for unit in obs.observation.raw_units if unit.unit_type == units.Terran.Marine]
        enemys = [unit for unit in obs.observation.raw_units if unit.alliance == features.PlayerRelative.ENEMY]

        if marines and enemys:
            target = sorted(enemys, key=lambda r: r.y)[0].tag
            return actions.RAW_FUNCTIONS.Attack_unit("now", marines, target)

        return FUNCTIONS.no_op()
```
위 코드는 y좌표가 가장 작은 (가장 위쪽에 있는)적 먼저 공격하는 코드입니다.  
1. obs.observation.raw_unit에서 unit의 종류로 분류할 수도, 유닛이 아군인지 적인지 여부로 분리시킬 수 있습니다.  
2. 유닛각각이 가진 속성을 이용하여 action을 정할 수 있습니다.
3. tag는 차후 유닛을 식별하기위한 고유키입니다.

### 강화학습에 사용하기 유리한 state
공식코드 state 정의 부분 https://github.com/deepmind/pysc2/blob/master/pysc2/lib/features.py#L171#L218
> 필수적인 변수를 추려보았습니다.
```
health              # 채력
health_ratio        # 채력비율
shield              # 프로토스 유닛의 경우 쉴드도 필요함
x                   # x좌표
y                   # y좌표
tag                 # 식별자
weapon_cooldown     # 무기 쿨다운
```

### 강화학습에 사용하기 유리한 action
공식코드 action 정의 부분 https://github.com/deepmind/pysc2/blob/master/pysc2/lib/actions.py#L583#L1165 
공식코드 action-raw 정의 부분 https://github.com/deepmind/pysc2/blob/master/pysc2/lib/actions.py#L1186#L1751
> 미니게임에서 필수적인 변수를 추려보았습니다. (매개변수는 공식코드에서 찾아가면서 보기)
```
*raw action*
Attack_unit     # 유닛공격
Move_unit       # 유닛 움직임
*action*
Attack_screen   # 스크린상에서 공격
Move_screen     # 스크린상에서 유닛움직임
```

## step 4
