## 문제

1. 다음과 같이 플래그를 지정하고 프로그램을 실행시키시오.<br />
./process-run.py -l 5:100,5:100<br />
CPU 이용률은 얼마가 되어야 하는가<br />
(예, CPU가 사용 중인 시간의 퍼센트?)<br />
그러한 이용률을 예측한 이유는 무엇인가?<br />
-c 플래그를 지정하여 예측이 맞는지 확인하시오.<br />

풀이 : 입출력 작업 없이 10개의 작업을 연속해서 하므로 100퍼센트 계속 CPU를 사용.
![image](https://github.com/user-attachments/assets/55d3ecfa-1d0d-46e1-80cd-77e0feedc02b)

2. 이제 다음과 같이 플래그를 지정하고 실행시키시오.<br />
./process-run.py -l 4:100,1:0<br />
이 플래그는 4개의 명령어를 실행하고 모두 CPU만 사용하는 하나의 프로세스와<br />
오직 입출력을 요청하고 완료되기를 기다리는 하나의 프로세스를 명시한다.<br />
두 프로세스가 모두 종료되는 데 얼마의 시간이 걸리는가?<br />
-c 플래그를 사용하여 예측한 것이 맞는지 확인하시오.<br />

풀이 : 입출력작업과 명령어 처리를 동시에 못하므로 4개의 명령어 처리하는시간 + 입출력 작업 시간만큼 걸릴것이다.
![image](https://github.com/user-attachments/assets/5ec0d442-4994-484a-aaa6-62eb696aa9cb)

3. 옵션으로 지정된 프로세스의 순서를 바꾸시오.<br />
./process-run.py -l 1:0,4:100<br />
이제 어떤 결과가 나오는가?<br />
실행 순서를 교환하는 것은 중요한가?<br />
이유는 무엇인가?<br />
(언제나처럼 -c 플래그를 사용하여 예측이 맞는지 확인하시오.)<br />

풀이 : 첫번째 입출력 프로세스가 돌아가는동안 두번째 명령어 프로세스가 돌아가기 때문에 속도가 빨라진다. 순서는 매우 중요하다.
![image](https://github.com/user-attachments/assets/bae4b79c-49ca-4602-b7cd-e3c7ea3c4271)

4. 자, 다른 플래그에 대해서도 알아보자.<br />
중요한 플래그 중 하나는 -S로서 프로세스가 입출력을 요청했을 때 시스템이 어떻게 반응하는지를 결정한다.<br />
이 플래그가 SWITCH_ON_END로 지정되면 시스템은 요청 프로세스가 입출력을 하는 동안<br />
다른 프로세스로 전환하지 않고 대신 요청 프로세스가 종료될 때까지 기다린다.<br />
입출력만 수행하는 프로세스와 CPU 작업만 하는 프로세스 두 개를 실행시키면 어떤 결과가 발생하는가?<br />
(-l 1:0,4:200 -c -S SWITCH_ON_END)<br />

풀이 : 다시 순서를 안바꿨을때 만큼 속도가 걸릴것 같다.

![image](https://github.com/user-attachments/assets/4278a0a5-3863-49a7-adc7-3e7e85e68c83)

5. 이번에는 프로세스가 입출력을 기다릴 때마다<br />
다른 프로세스로 전환하도록 플래그를 지정하여 같은 프로세스를 실행시켜 보자<br />
(-l 1:0,4:100 -c -S SWITCH_ON_IO)<br />
이제 어떤 결과가 발생하는가?<br />
-c를 사용하여 예측이 맞는지 확인하시오.<br />

풀이 :순서를 안바꾸고도 입출력과 명령어 처리가 동시에 처리될것 같다.
![image](https://github.com/user-attachments/assets/9a6a2dca-833e-497a-b4de-b6ffffd746b7)

6. 또 다른 중요한 행동은 입출력이 완료되었을 때 무엇을 하느냐이다.<br />
-I IO_RUN_LATER 가 지정되면 입출력이 완료되었을 때<br />
입출력을 요청한 프로세스가 바로 실행될 필요가 없다.<br />
완료 시점에 실행 중이던 프로세스가 계속 실행된다.<br />
다음과 같은 조합의 프로세스를 실행시키면 무슨 결과가 나오는가?<br />
(./process-run.py -l 3:0, 5:100,5:100, 5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p)<br />
시스템 자원은 효과적으로 활용되는가?<br />

풀이 : 입출력 프로세스 실행만 맨 뒤로 미뤄지지 않을까 그럼 자원의 낭비 아닌가? 첫번째 입출력이 끝나고 나머지 입출력 동안은 놀게된다.
![image](https://github.com/user-attachments/assets/d19e092c-3df4-4c3f-886e-70990942a341)

7. 같은 프로세스 조합을 실행시킬 때<br />
-I IO_RUN_IMMEDIATE를 지정하고 실행시키시오.<br />
이 플래그는 입출력이 완료되었을 때 요청 프로세스가 곧바로 실행되는 동작을 의미한다.<br />
이 동작은 어떤 결과를 만들어 내는가?<br />
방금 입출력을 완료한 프로세스를 다시 실행시키는 것이 좋은 생각일 수 있는 이유는 무엇인가?<br />

풀이 : 입출력이 여러개일때는 첫번째 입출력이 끝나기 전까진 다음 입출력을 시작 못하기 때문에 조금이라도 빨리 끝내고 빨리 시작해야 시간이 단축된다.
![image](https://github.com/user-attachments/assets/2787a1f1-ea55-4f7b-ad35-5958026f6b1e)

8. 이제 다음과 같이 무작위로 생성된 프로세스를 실행시켜 보자.<br />
예를 들면,<br />
-s 1 -l 3:50,3:50, -s 2 -l 3:50,3:50, -s 3 -l 3:50,3:50<br />
어떤 양상을 보일지 예측할 수 있는지 생각해 보시오.<br />

-I IO_RUN_IMMEDIATE 를 지정했을 때와<br />

-I IO_RUN_LATER 를 지정했을 때 어떤 결과가 나오는가?<br />

-S SWITCH_ON_IO 대 -S SWITCH_ON_END의 경우에는 어떤 결과가 나오는가?<br />

풀이 : 
SWITCH_ON_END 일때는 요청 프로세스가 종료될때까지 기다리므로 입출력이나 명령어 처리동안 계속 놀게된다.<br />
SWITCH_ON_IO 일때는 동시에 처리하므로 입출력이 두개 동시에 돌아가는 경우도 있다.<br />
의외로 입출력 실행을 언제 하냐는 별 차이가 없다.<br />
단지SWITCH_ON_IO가 바로 실행해서 살짝 빠르다<br />
