# C++ namespace - 네임스페이스

- 네임스페이스는 코드에서 이름이 서로 충돌하는 문제를 해결하기 위해 나온 개념이다.
- 예를 들어 foo라는 이름의 함수를 만들어서 사용하다가 외부 라이브러리를 참조했을 때, 거기에도 foo라는 함수가 있다면 충돌이 발생한다.
- 이럴 경우, 컴파일러는 foo가 어떤 함수를 가리키는지 알수 없다.
- 아래 간단 예제 코드를 보자

[namespace 예제 코드]

```cpp
#include <iostream>

namespace nameA{
	void foo(){
		std::cout<<"It is in nameA"<<std::endl;
	}
};

namespace nameB{
	void foo(){
		std::cout<<"It is in nameB"<<std::endl;
	}
};

int main(void){
	nameA::foo();
	nameB::foo();

	return 0;
}
```

[실행 결과]

```cpp
It is in nameA
It is in nameB
```

- :: 기호를 이용해서 (정확한 명칭은 scope resolution operator) 사용하면 된다.

- nested namespace라고 해서 namespace를 중첩해서 사용하는 것도 가능하다.
- namespace alias로 이름을 다르게 표현할 수 있는데 아래 예제를 통해 두가지를 한번에 확인해보자

```cpp
#include <iostream>

namespace A{
	namespace B{
		namespace C{
			void foo(){
				std::cout<<"Nested namespace example"<<std::endl;
			}
		}
	}
}

namespace a::b::c{
	void foo(){	
		std::cout<<"Supported  C++17"<<std::endl;
	}
}

int main(void){
	namespace ABC = A::B::C;
	namespace abc = a::b::c;

	A::B::C::foo();
	a::b::c::foo();
	ABC::foo();
	abc::foo();

	return 0;
}
```

```bash
Nested namespace example
Supported  C++17
Nested namespace example
Supported  C++17
```

- 만약 사용하는 빈도수가 높은 경우 using 지시자를 통해 컴파일러에 처리하도록 하면 된다
- 보통 c++ 코드에서 쓰이는 using namespace std; 도 std라는 namespace를 :: 기호 없이 사용하겠다는 뜻이다.
- using 예제 코드

```cpp
#include <iostream>

using std::cout;

namespace nameA{
	void foo(){
		cout<<"It is in nameA"<<std::endl;
	}
};

namespace nameB{
	void foo(){
		cout<<"It is in nameB"<<std::endl;
	}
};

using namespace nameA;

int main(void){
	foo();
	nameB::foo();

	cout<<"using std::cout (not endl)"<< std::endl;

	return 0;
}
```

```bash
It is in nameA
It is in nameB
using std::cout (not endl)
```

- using namespace를 통해 nameA:: 를 생략하여 사용했다. (using을 했어도 nameA를 붙여도 되긴한다.)
- 또한, using namespace가 아니라 using 후 특정 항목만 지정해서 사용가능하다.
- using std::cout 을 통해 컴파일러에게 지시를 하여 cout은 std::없이 사용했지만 endl의 경우 std::endl로 사용하였다.

웬만하면 using 사용은 지양하자 (using namespace std 포함) - 예상치 못한 변수나 함수명이 겹치는 경우가 발생할 수 있다.

ref) [https://en.cppreference.com/w/cpp/language/namespace](https://en.cppreference.com/w/cpp/language/namespace)