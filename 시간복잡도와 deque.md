# 시간복잡도
### 시간제한이 1초라면 (N은 입력) 대략
| **O(1), O(log N)** | **N    <=    ∞** |
|:-:|:-:|
| **O(N)** | **N    <=    100,000,000(1억)** |
| **O(N log N)** | **N    <=    13,993,959(1천 3백만)** |
| **O(N²)** | **N    <=    10,000(1만)** |
| **O(N³)** | **N    <=    464** |
| **O(2ᴺ)** | **N    <=    26** |
| **O(N!)** | **N    <=    12** |
### 이정도로 어림잡을 수 있다.

내가 이번에 푼 백준 문제는 시간제한이 0.3초였고, 최대 50만개의 인풋이 주어질 수 있다.(연산이 50만개 수행되어야할수있음)
따라서 최대 nlogn의 시간 복잡도로 수행되어야함.

```python
def editor(myString, edit, cursor):
    type = edit[0]
    if type == "P":
        _, d = map(str,edit.split())
        myString = myString[:cursor]+d+myString[cursor:]
        cursor = cursor+1
    elif type == "L":
        if cursor > 0:
            cursor = cursor - 1
    elif type == "D":
        if cursor < len(myString):
            cursor = cursor + 1
    elif type == "B":
        if cursor >0:
            myString = myString[:cursor-1]+myString[cursor:]
            cursor = cursor-1
    else:
        print("Something gone wrong")
    return myString,cursor

myString = input()
cursor = len(myString)
for _ in range(int(input())):
    #처리하는 함수 호출
    myString,cursor = editor(myString, input(), cursor)
print(myString)

```
내생각엔 시간복잡도가 O(N)인거같은데,, (for에서만 N, 그리고 나머지 연산은 상수) 어느부분에서 추가로 품이 들어간건지 봐야할 것 같다.

아하.. gpt에 물어보니 문자열 슬라이싱 및 새로운 문자열을 만드는 연산이 O(M)이라 O(NM)이라고 한다.

replace와,,, del을 사용해서 해보려 했으나 실패

deque를 사용해보기로 결정

## deque
deque는 **앞과 뒤에서 데이터를 처리할 수 있는 양방향 자료형**으로, 스택처럼 써도 되고 큐처럼써도 된다. `collections.deque`

사용법을 잠시 알아봤다.

```python
>>> from collections import deque
>>> d = deque([1,2,3,4,5])
>>> d.append(6)
>>> d
deque([1, 2, 3, 4, 5, 6])
>>> d.appendleft(0)
>>> d
deque([0, 1, 2, 3, 4, 5, 6])
>>> d.pop()
6
>>> d
deque([0, 1, 2, 3, 4, 5])
>>> d.popleft()
0
>>> d
deque([1, 2, 3, 4, 5])
>>>
```

위를 사용해서 내가 짠 코드는 아래와 같다

```python

def editor(myDeque, edit, cursor):
    type = edit[0]
    if type == "P":
        _, d = map(str,edit.split())
        if cursor == "0":
            myDeque.appendleft(d)
        else:
            myDeque.append(d)
        cursor = cursor+1
        
    elif type == "L":
        if cursor > 0:
            myDeque.rotate(1)
            cursor = cursor-1
    elif type == "D":
        if cursor < len(myString):
            myDeque.rotate(-1)
            cursor = cursor + 1
    elif type == "B":
        if cursor > 0:
            myDeque.pop()
            cursor = cursor-1
    else:
        print("Something gone wrong")
    return myDeque,cursor

myString = input()
cursor = len(myString)
myDeque = deque(list(myString))
print(myDeque)
for _ in range(int(input())):
    #처리하는 함수 호출
    myDeque,cursor = editor(myDeque, input(), cursor)
    print(myDeque)

myDeque.rotate(cursor)
print(*myDeque)


```
이 코드의 문제는 deque의 특성상 커서의 위치를 완벽하게 구현해내지 못한다는 점에 있다.

계속 짜다가 결국 gpt한테 물어봤다,

```python

import sys
from collections import deque

input = sys.stdin.read
data = input().split("\n")

# 초기 문자열
myString = data[0].strip()
M = int(data[1])  # 명령어 개수

# 커서를 기준으로 왼쪽과 오른쪽을 관리할 두 개의 deque 사용
left_stack = deque(myString)  
right_stack = deque()  

# 명령어 수행
for i in range(2, M + 2):
    if not data[i]: 
        continue
    command = data[i].split()

    if command[0] == "L":
        if left_stack:
            right_stack.appendleft(left_stack.pop())
    
    elif command[0] == "D":
        if right_stack:
            left_stack.append(right_stack.popleft())

    elif command[0] == "B":
        if left_stack:
            left_stack.pop()
    
    elif command[0] == "P":
        left_stack.append(command[1])

# 최종 결과 출력
print("".join(left_stack) + "".join(right_stack))

```

GPT는 양쪽 스택을 두개의 deque로 구현하여 커서의 위치를 구현했다.
deque는 입출력이 상수시간으로 해결될수 있기 떄문에 결국 O(M)으로 구현해 낼수있다. 
