# Vectorization / 벡터화

- 대부분의 프로그래밍 언어가 그렇듯 스칼라 계산을 여러번 돌리는 것보다 **벡터화(Vectorization)을 통해 연산을 한번에 하는 것이 월등히 빠르다.** 이유를 대충 설명하자면 1000번의 연산을 한다했을때, 스칼라 계산으로 돌릴 경우 한 사람이 1000번의 일을 하게 되지만 벡터화로 짜게 되면 내부적으로 뭐 예를들어 8명의 사람이 125번의 일을 하도록 돌아간다고 보면 된다.

- 그와중에 **매트랩은 유독 for, while과 같은 loop문을 제대로 못돌린다.** C/C++나 심지어 느리기로 유명한 Python도 귀찮을땐 그냥 loop문으로 짜서 돌려도 뭐 돌아가기는 하는데 매트랩은 실행 시간이 유독 심하게 늘어난다. (Iteration이 1,000번을 넘어가는 loop문 사용을 지양해야한다)

- 아래에서 매트랩에서의 여러 Vectorization에 대해서 살펴보자

**1. 배열 관련**

코드 1
```matlab
nIteration = 1e8;
x = linspace(0,2*pi,nIteration);
tic()
for i = 1 : nIteration
    y(i) = sin(x(i));
end
toc()
```
실행결과
```matlab
경과 시간은 11.778212초입니다.
```

코드 2
```matlab
nIteration = 1e8;
x = linspace(0,2*pi,nIteration);
tic()
y = sin(x);
toc()
```
실행결과
```matlab
경과 시간은 0.451144초입니다.
```

- 매트랩 문법을 조금 이해한다면 알겠지만 (사실 몰라도 쉽게 이해되지만) x의 sin값인 y를 생성하는 코드이다. (tic, toc은 시간 측정 코드)
- 똑같은 연산인데도 **500배 넘는 성능차이**를 보인다. (사실 Pre-allocation의 이유가 더 크지만 이건 나중에 다루도록 하자 - Vectorization을 안할 시 많이 놓치게 되는 부분이기도하다.)
- sum / mean / diff / cumsum 등 내부함수와 . 기호를 활용한 요소 단위 연산을 무조건적으로 활용해서 loop문을 지우는게 매트랩 코드의 퍼포먼스를 상승하는데 도움이 된다.

**2. Array-Indexing 활용**

- 사실 위에 배열 같은 경우는 몇번 짜다보면 누가 가르쳐주지 않아도 알아서 잘 쓰고 있을텐데 배열 인덱싱 (Array-Indexing) 같은 경우는 초보자 대부분이 놓치는 부분이다.
- 배열 인덱싱은 말 그대로 인덱스에 배열을 넣는 기법인데 매트랩에서 지향하는 방법이기도 하다.

- 배열 인덱싱을 하기 전에 논리값 배열과 find에 대해서 먼저 보고 넘어가자

```matlab
x = [10,15,7,19];
y = x > 13;        % y = [false,true,false,true];
z = find(x > 13);  % z = [2,4];
```

- Line 2의 y가 논리값 배열이고 z는 find 명령어를 수행한 부분이다.
- 논리값 배열은 주어진 조건을 만족하는 경우에 대해서 true를, 만족시키지 못할 경우 false를 지닌 배열을 반환한다.
- find 명령어는 주어진 조건을 만족시키는 Index를 반환한다.
- 약간 비슷하긴 하지만 보통 배열 인덱싱에는 y와 같이 논리값 배열을 많이 쓰는 편이다.
- 해당 문법을 이용해 Vectorization 아래와 같은 기법을 구현해보자

```matlab
x = rand(1e8, 1);
result = 0;
tic()
idx = 1;
for i = 1 : length(x)
    if x(i) > 0.5
       result(idx) = x(i);
       idx = idx + 1;
    end
end
toc()
```

```matlab
x = rand(1e8, 1);
tic()
idx = x > 0.5;
result = x(idx); % equal to x = x(x>0.5);
toc()
```

- [0,1] 구간 에서 랜덤하게 생성된 x에 대해 0.5보다 큰 값에 대해서 result에 저장하는 코드로, 왼쪽은 loop문 오른쪽은
- **왼쪽 코드는 5.45초 오른쪽 코드는 0.79초로 7~8배 성능차이를 보인다.** (역시나 pre-allocation 문제가 크다.) 코드 간결성 또한 덤이다.
- idx = x > 0.5 라는 구문을 통해 논리값 배열을 생성하고 x 배열의 인덱스로 활용해 쉽게 casting 할 수 있다.
- 1920 by 1080 by 3의 크기를 갖는 FHD RGB 이미지에서 특정 구간이나 색 범위 등을 추출하는데 쓴다면 이미지 처리시에 보다 빠르게 처리가 가능하다.

**3. 기타 연산들**

- 행렬 복사

```matlab
A = [1,2,3];
B = [];
C = [];
for i = 1 : 3
    B = [B, A];
    C = [C; A];
end

```

```matlab
A = [1,2,3];
B = repmat(A,1,3);
C = repmat(A,3,1);
```

- fread

```matlab
% Pseudo code
cnt = 10
for i = 1 : 10
    A(i) = fread(fid, 1, 'char');
end
```

```matlab
% Pseudo code
cnt = 10
A = fread(fid, cnt, 'char');
```

- 생각나는대로 추가 업로드 예정

Ref) [https://kr.mathworks.com/help/matlab/matlab_prog/vectorization.html](https://kr.mathworks.com/help/matlab/matlab_prog/vectorization.html)
