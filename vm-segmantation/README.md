
# 개요

이 프로그램은 세그먼테이션이 있는 시스템에서 주소 변환(address translations)이 어떻게 수행되는지 볼 수 있게 해줍니다. 이 시스템이 사용하는 세그먼테이션은 매우 간단합니다: 주소 공간은 단지 *두* 개의 세그먼트만 있습니다; 더 나아가, 프로세스에 의해 생성된 가상 주소의 최상위 비트가 주소가 어느 세그먼트에 있는지 결정합니다: 0은 세그먼트 0 (코드와 힙이 위치할 곳)이고 1은 세그먼트 1 (스택이 위치하는 곳)입니다. 세그먼트 0은 양의 방향으로 성장합니다 (더 높은 주소로), 반면 세그먼트 1은 음의 방향으로 성장합니다.

시각적으로, 주소 공간은 다음과 같이 보입니다:

```
 --------------- 가상 주소 0
 |    seg0     |
 |             |
 |             |
 |-------------|
 |             |
 |             |
 |             |
 |             |
 |(할당되지 않음)|
 |             |
 |             |
 |             |
 |-------------|
 |             |
 |    seg1     |
 |-------------| 가상 주소 최대 (주소 공간의 크기)
```

세그먼테이션에서, 각 세그먼트마다 base/limit 쌍의 레지스터가 있습니다. 따라서 이 문제에서는 두 개의 base/limit 쌍이 있습니다. 세그먼트-0 base는 세그먼트 0의 *상단*이 물리 메모리의 어느 주소에 배치되었는지 알려주고, limit은 세그먼트의 크기를 알려줍니다; 세그먼트-1 base는 세그먼트 1의 *하단*이 물리 메모리의 어느 주소에 배치되었는지 알려주고, 해당 limit도 세그먼트의 크기(또는 음의 방향으로 얼마나 성장하는지)를 알려줍니다.

이전과 마찬가지로, 프로그램을 실행하여 세그먼테이션에 대한 이해를 테스트하는 두 단계가 있습니다. 먼저, "-c" 플래그 없이 실행하여 변환 세트를 생성하고, 주소 변환을 올바르게 수행할 수 있는지 확인하세요. 그런 다음, 완료되면 "-c" 플래그와 함께 실행하여 답을 확인하세요.

예를 들어, 기본 플래그로 실행하려면 다음을 입력하세요:

```sh
prompt> ./segmentation.py 
```
또는

```sh 
prompt> python ./segmentation.py 
```

다음과 같이 표시됩니다:
```sh
  ARG seed 0
  ARG address space size 1k
  ARG phys mem size 16k
  
  Segment register information:

    Segment 0 base  (grows positive) : 0x00001aea (decimal 6890)
    Segment 0 limit                  : 472

    Segment 1 base  (grows negative) : 0x00001254 (decimal 4692)
    Segment 1 limit                  : 450

  Virtual Address Trace
    VA  0: 0x0000020b (decimal:  523) --> PA or segmentation violation?
    VA  1: 0x0000019e (decimal:  414) --> PA or segmentation violation?
    VA  2: 0x00000322 (decimal:  802) --> PA or segmentation violation?
    VA  3: 0x00000136 (decimal:  310) --> PA or segmentation violation?
    VA  4: 0x000001e8 (decimal:  488) --> PA or segmentation violation?

  For each virtual address, either write down the physical address it translates
  to OR write down that it is an out-of-bounds address (a segmentation
  violation). For this problem, you should assume a simple address space with
  two segments: the top bit of the virtual address can thus be used to check
  whether the virtual address is in segment 0 (topbit=0) or segment 1
  (topbit=1). Note that the base/limit pairs given to you grow in different
  directions, depending on the segment, i.e., segment 0 grows in the positive
  direction, whereas segment 1 in the negative.  
```

그런 다음, 가상 주소 추적에서 변환을 계산한 후, "-c" 플래그와 함께 프로그램을 다시 실행하세요. 다음과 같이 표시됩니다 (중복 정보 제외):

```sh
  Virtual Address Trace
    VA  0: 0x0000020b (decimal:  523) --> SEGMENTATION VIOLATION (SEG1)
    VA  1: 0x0000019e (decimal:  414) --> VALID in SEG0: 0x00001c88 (decimal: 7304)
    VA  2: 0x00000322 (decimal:  802) --> VALID in SEG1: 0x00001176 (decimal: 4470)
    VA  3: 0x00000136 (decimal:  310) --> VALID in SEG0: 0x00001c20 (decimal: 7200)
    VA  4: 0x000001e8 (decimal:  488) --> SEGMENTATION VIOLATION (SEG0)
```

각 가상 주소에 대해, 그것이 변환되는 물리 주소를 적거나, 범위를 벗어난 주소(세그먼테이션 위반)임을 적으세요. 이 문제에서는 두 개의 세그먼트로 구성된 간단한 주소 공간을 가정해야 합니다: 가상 주소의 최상위 비트는 가상 주소가 세그먼트 0 (topbit=0)인지 세그먼트 1 (topbit=1)인지 확인하는 데 사용할 수 있습니다. 주어진 base/limit 쌍이 세그먼트에 따라 다른 방향으로 성장한다는 점에 유의하세요, 즉 세그먼트 0은 양의 방향으로 성장하고 세그먼트 1은 음의 방향으로 성장합니다.

보시다시피, `-c` 옵션을 사용하면 프로그램이 주소들을 대신 변환해주므로
세그먼테이션을 사용하는 시스템이 주소를 어떻게 변환하는지 스스로 확인할
수 있습니다.

물론 다양한 문제를 만들기 위해 사용할 수 있는 매개변수들이 있습니다.
특히 중요한 매개변수는 `-s` 또는 `-seed`로, 다른 랜덤 시드를 전달하여
다른 문제를 생성할 수 있게 해줍니다. 문제를 생성할 때와 풀이할 때는 같은
랜덤 시드를 사용해야 결과를 재현할 수 있습니다.

또한 서로 다른 크기의 주소 공간과 물리 메모리를 실험해볼 수 있는
매개변수들이 있습니다. 예를 들어 작은 시스템에서 세그먼테이션을
실험해보려면 다음과 같이 실행할 수 있습니다:

```sh
prompt> ./segmentation.py -s 100 -a 16 -p 32
ARG seed 0
ARG address space size 16
ARG phys mem size 32
 
Segment register information:

  Segment 0 base  (grows positive) : 0x00000018 (decimal 24)
  Segment 0 limit                  : 4

  Segment 1 base  (grows negative) : 0x00000012 (decimal 18)
  Segment 1 limit                  : 5

Virtual Address Trace
  VA  0: 0x0000000c (decimal:   12) --> PA or segmentation violation?
  VA  1: 0x00000008 (decimal:    8) --> PA or segmentation violation?
  VA  2: 0x00000001 (decimal:    1) --> PA or segmentation violation?
  VA  3: 0x00000007 (decimal:    7) --> PA or segmentation violation?
  VA  4: 0x00000000 (decimal:    0) --> PA or segmentation violation?
```

이는 16바이트 크기의 주소 공간을 32바이트 물리 메모리의 어딘가에
배치하도록 가상 주소들을 생성하라는 의미입니다. 보시다시피 생성된 가상
주소들은 매우 작습니다(12, 8, 1, 7, 0). 또한 프로그램이 상황에 맞게 작은
base와 limit 값을 선택합니다. 정답을 보려면 `-c` 옵션으로 다시 실행하세요.

이 예시는 각 base/limit 쌍이 무엇을 의미하는지도 분명히 보여줍니다. 예를
들어 세그먼트 0의 base가 물리 주소 24(decimal)로 설정되고 크기가 4바이트라면,
가상 주소 0, 1, 2, 3은 세그먼트 0에 속하며 각각 물리 주소 24, 25, 26, 27에
매핑됩니다.

음의 방향으로 성장하는 세그먼트 1은 조금 더 까다롭습니다. 위의 작은 예에서
세그먼트 1의 base 레지스터가 물리 주소 18로 설정되고 크기가 5바이트이면,
가상 주소 공간의 마지막 다섯 바이트(이 예에서는 11, 12, 13, 14, 15)가 유효한
가상 주소가 되며, 이들은 각각 물리 주소 13, 14, 15, 16, 17로 매핑됩니다.

이 부분이 이해되지 않으면 다시 읽어보세요 — 이 동작을 이해해야 문제를 풀 수
있습니다.

`-a` 또는 `-p` 플래그에 전달하는 값 뒤에 `k`, `m`, `g`를 붙여 킬로바이트,
메가바이트 또는 기가바이트 단위로 더 큰 값을 지정할 수 있습니다. 예를 들어
주소 공간을 1MB로, 물리 메모리를 32MB로 설정하려면 다음과 같이 실행합니다:

```sh
prompt> ./segmentation.py -a 1m -p 32m
```

더 구체적으로 제어하고 싶다면 `--b0`, `--l0`, `--b1`, `--l1` 옵션으로 각 세그먼트의
base와 limit 값을 직접 설정할 수도 있습니다. 직접 값들을 바꿔가며 실험해보세요.

마지막으로, 사용 가능한 모든 플래그와 옵션 목록을 보려면 다음을 실행하세요:

```sh
prompt> ./segmentation.py -h
```

즐겨보세요!



