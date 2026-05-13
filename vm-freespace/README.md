# 개요 (Overview)

이 프로그램인 `malloc.py`를 사용하면 간단한 메모리 할당기(memory allocator)가 어떻게 작동하는지 확인할 수 있습니다. 사용할 수 있는 옵션은 다음과 같습니다:

```sh
  -h, --help            이 도움말 메시지를 표시하고 종료합니다
  -s SEED, --seed=SEED  난수 시드 (random seed)
  -S HEAPSIZE, --size=HEAPSIZE
                        힙의 크기
  -b BASEADDR, --baseAddr=BASEADDR
                        힙의 기본 주소 (base address)
  -H HEADERSIZE, --headerSize=HEADERSIZE
                        헤더의 크기
  -a ALIGNMENT, --alignment=ALIGNMENT
                        할당 단위를 이 크기에 맞게 정렬합니다; -1->정렬 안 함
  -p POLICY, --policy=POLICY
                        빈 공간 리스트 검색 정책 (BEST, WORST, FIRST)
  -l ORDER, --listOrder=ORDER
                        리스트 정렬 순서 (ADDRSORT, SIZESORT+, SIZESORT-, INSERT-FRONT, INSERT-BACK)
  -C, --coalesce        빈 공간 리스트(free list)를 병합(coalesce)할 것인가?
  -n OPSNUM, --numOps=OPSNUM
                        생성할 임의의 연산 수
  -r OPSRANGE, --range=OPSRANGE
                        최대 할당 크기
  -P OPSPALLOC, --percentAlloc=OPSPALLOC
                        할당 연산의 비율 (%)
  -A OPSLIST, --allocList=OPSLIST
                        임의 생성 대신, 사용할 연산 리스트 지정 (+10,-0 등)
  -c, --compute         정답을 계산해 줍니다
```

이 프로그램을 사용하는 한 가지 방법은 프로그램이 임의의 할당(allocation) 및 해제(free) 연산을 생성하게 하고, 여러분이 각 연산의 성공이나 실패 여부와 더불어 빈 공간 리스트(free list)가 어떤 모습일지 알아맞혀 보는 것입니다.

간단한 예시는 다음과 같습니다:

```sh
prompt> ./malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 

ptr[0] = Alloc(3)  returned ?
List?

Free(ptr[0]) returned ?
List?

ptr[1] = Alloc(5)  returned ?
List?

Free(ptr[1]) returned ?
List?

ptr[2] = Alloc(8)  returned ?
List?
```

이 예시에서는 주소 1000(`-b 1000`)에서 시작하는 크기 100 바이트(`-S 100`)의 힙을 지정합니다. 할당된 각 블록마다 추가적으로 4 바이트의 헤더를 지정하고(`-H 4`), 할당된 각 공간의 크기를 4의 배수로 올림하여 정렬하도록 설정합니다(`-a 4`). 빈 공간 리스트는 주소순(오름차순)으로 유지되도록 지정합니다(`-l ADDRSORT`). 마지막으로, "최적 적합(best fit)" 빈 공간 검색 정책을 지정하고(`-p BEST`), 5개의 임의 연산을 생성하도록 요청합니다(`-n 5`). 이 실행 결과는 위에 나와 있습니다. 여러분의 임무는 각 할당/해제 연산이 무엇을 반환하는지, 그리고 각 연산 후에 빈 공간 리스트의 상태가 어떻게 되는지 파악하는 것입니다.

여기서 `-c` 옵션을 사용하여 결과를 확인해 보겠습니다.

```sh
prompt> ./malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 -c

ptr[0] = Alloc(3)  returned 1004 (searched 1 elements)
Free List [ Size 1 ]:  [ addr:1008 sz:92 ]

Free(ptr[0]) returned 0
Free List [ Size 2 ]:  [ addr:1000 sz:8 ] [ addr:1008 sz:92 ]

ptr[1] = Alloc(5)  returned 1012 (searched 2 elements)
Free List [ Size 2 ]:  [ addr:1000 sz:8 ] [ addr:1020 sz:80 ]

Free(ptr[1]) returned 0
Free List [ Size 3 ]:  [ addr:1000 sz:8 ] [ addr:1008 sz:12 ] [ addr:1020 sz:80 ]

ptr[2] = Alloc(8)  returned 1012 (searched 3 elements)
Free List [ Size 2 ]:  [ addr:1000 sz:8 ] [ addr:1020 sz:80 ]
```

보시다시피, 첫 번째 연산(할당)은 다음 정보를 반환합니다:

```sh
ptr[0] = Alloc(3)  returned 1004 (searched 1 elements)
Free List [ Size 1 ]:  [ addr:1008 sz:92 ]
```

빈 공간 리스트의 초기 상태는 단순히 하나의 큰 요소이므로, `Alloc(3)` 요청이 성공할 것임을 쉽게 추측할 수 있습니다. 나아가, 이는 단순히 메모리의 첫 번째 청크(chunk)를 반환하고 나머지를 빈 공간 리스트로 만들 것입니다. 반환되는 포인터는 헤더 바로 다음 주소(address: 1004)가 되며, 할당된 공간은 4 바이트로 올림되어, 빈 공간 리스트에는 1008에서 시작하는 92 바이트가 남게 됩니다.

다음 연산은 이전 할당 요청의 결과를 저장하고 있는 `ptr[0]`에 대한 해제(Free)입니다. 예상할 수 있듯이 이 해제는 성공할 것이며(따라서 `0`을 반환), 빈 공간 리스트는 이제 조금 더 복잡해 보일 것입니다:

```sh
Free(ptr[0]) returned 0
Free List [ Size 2 ]:  [ addr:1000 sz:8 ] [ addr:1008 sz:92 ]
```

실제로, 우리는 빈 공간 리스트를 병합(coalesce)하지 않기 때문에, 이제 리스트에는 두 개의 요소가 있습니다. 첫 번째는 방금 반환된 공간을 유지하는 8 바이트 크기의 요소이고, 두 번째는 92 바이트의 청크입니다.

`-C` 플래그를 통해 병합 기능을 켤 수 있으며, 그 결과는 다음과 같습니다:

```sh
prompt> ./malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 -c -C
ptr[0] = Alloc(3)  returned 1004 (searched 1 elements)
Free List [ Size 1 ]:  [ addr:1008 sz:92 ]

Free(ptr[0]) returned 0
Free List [ Size 1 ]:  [ addr:1000 sz:100 ]

ptr[1] = Alloc(5)  returned 1004 (searched 1 elements)
Free List [ Size 1 ]:  [ addr:1012 sz:88 ]

Free(ptr[1]) returned 0
Free List [ Size 1 ]:  [ addr:1000 sz:100 ]

ptr[2] = Alloc(8)  returned 1004 (searched 1 elements)
Free List [ Size 1 ]:  [ addr:1012 sz:88 ]
```

해제(Free) 연산이 수행될 때 예상대로 빈 공간 리스트가 병합되는 것을 볼 수 있습니다.

살펴볼 만한 몇 가지 다른 흥미로운 옵션들이 있습니다:

* `-p BEST` 또는 `-p WORST` 또는 `-p FIRST`: 이 옵션은 할당 요청 중에 사용할 메모리 청크를 찾기 위해 세 가지 다른 전략 중 하나를 사용할 수 있게 해줍니다.
* `-l ADDRSORT` 또는 `-l SIZESORT+` 또는 `-l SIZESORT-` 또는 `-l INSERT-FRONT` 또는 `-l INSERT-BACK`: 이 옵션을 사용하면 빈 공간 리스트를 특정 순서로 유지할 수 있습니다. 예를 들어, 빈 공간의 주소순, 빈 공간의 크기순(`+`로 오름차순 또는 `-`로 내림차순), 혹은 단순히 해제된 빈 공간을 리스트의 맨 앞(INSERT-FRONT)이나 맨 뒤(INSERT-BACK)에 반환하는 방식입니다.
* `-A list_of_ops`: 이 옵션은 임의로 생성되는 요청 대신 일련의 정확한 요청을 지정할 수 있게 해줍니다. 예를 들어, 플래그를 `-A +10,+10,+10,-0,-2`로 설정하고 실행하면 10 바이트 크기(플러스 헤더)의 청크 세 개를 할당한 다음, 첫 번째 청크를 해제(`-0`)하고 세 번째 청크를 해제(`-2`)합니다. 그때 빈 공간 리스트는 어떤 모습일까요?

이것이 기본적인 내용입니다. 교재 해당 장의 질문들을 활용하여 더 탐구해 보거나, 할당기가 어떻게 기능하는지 더 잘 이해하기 위해 직접 새롭고 흥미로운 질문들을 만들어 보세요.
