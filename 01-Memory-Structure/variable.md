## Variable types


- variable types는 compiler, architecture 마다 다를 수 있다.


이러한 문제를 두 가지의 방법으로 차단할 수 있는데 첫 번째 방법으로 `compilation assertion`을 이용하는 것이다.
이건 컴파일 과정에서 어떤 조건이 만족 하는지 안하는지를 알려준다.

```cpp
#include <iostream>

int main()
{
  static_assert(sizeof(int)==4, "int is 4bytes"); // int의 size가 4인지 알고싶다 만약 4바이트가 아니면 에러발생
  int a = 0;
  
  std::cout << sizeof(int) << std::endl;  //  4
  std::cout << sizeof(a) << std::endl;    //  4

  return 0;
}
```

두 번째 방법으로는 `Fixed width variable types`의 방법으로 사이즈를 정해서 사용하는 방법이다.
Memory layout을 정확히 생각하고 사용해야 할 때는 이 방법을 이용하는 것이 좋다.

```cpp
#include <cstdint>  //  이 방법을 위한 Fixed width integer types 헤더파일 선언
#include <array>    //  배열 생성을 위한 헤더파일
#include <iostream>

int main()
{
  uint64_t ui8;               //  64bit(8byte)
  float f4;                   //  32bit(4byte)
  std::array<uint8_,5> uia5;  //  40bit(5byte)
  
  uint64_t * ui64ptr = &ui8;  //  
  
  std::cout << sizeof(ui64ptr) << std::endl;    //  포인터의 사이즈는 OS에 따라 타름
  std::cout << (uint64_t)ui64ptr << std::endl;  //  주소값 반환

  return 0;
}
```

- padding

OS의 memory access pattern으로 인해 compiler는 알아서 memory에 padding을 넣어준다.


```cpp
struct Aligned
{
  char c1;
  int i4a;
  int i4b;
  double d8;
};

int main()
{
  std::cout << sizeof(Aligned) << std::endl;  //  24
  return 0;
}
```
Aligned의 size는 17byte로 생각하기 쉽지만 아니다.
각 variable types는 `자신의 byte배수로 memory access`를 하기 때문에 c1이 1byte 생성되면 3byte만큼 padding이 생기고, 4byte째 부터 i4a, i4b가 할당된다. 그리고 double은 8배수로 접근하므로 0, 8, 16 일때 접근하기 때문에 12~15까지 padding이 생기고 16byte째 부터 d8이 할당된다. 그래서 sizeof(Aligned)는 24가 된다.

```cpp
struct Aligned
{
  double d8;
  int i4b;
  int i4a;
  char c1;
};

struct S0
{
  char c1a;
  int i4;
  char c1b;
}; // 12bytes

struct S1
{
  int i4;
  char c1a;
  char c1b;
}; // 8bytes

int main()
{
  std::cout << sizeof(Aligned) << std::endl;  //  24
  return 0;
}
```

이러한 struct가 위에서 언급한 법칙대로하면 17바이트로 딱 떨어지게 memory가 할당될것 같지만 하나 더 고려해야 하는것은 struct 안에 `memory가 가장 큰 variable의 배수에 맞게 끊어줘야 하기 때문에` 0, 8, 16, 24등으로 memory가 할당되어야 하므로 18~24 까지 padding을 갖게 된다.
S0, S1가 각각의 byte를 차지하는 이유를 생각해보자.

```cpp
struct Aligned
{
  double d8;
  int i4b;
  int i4a;
  char c1;
};  //  17~24byte 까지 padding

struct alignas(32) Aligned_specifier
{
  double d8;
  int i4b;
  int i4a;
  char c1;
};  // 32byte로 만들어줘서 64byte인 cash line에 알맞게 설정한다. 17~32byte까지 padding

int main()
{
  std::cout << sizeof(Aligned) << std::endl;  //  24
  std::vector<Aligned> vec(10);               //  Aligned 자료형으로 10개의 vector 선언
  return 0;
}
```

`Cashe line이 보통 64byte 단위`로 된다. 즉, 하나의 코어당 64byte씩 가져가는데 위의 vec 같은 경우 24, 48, 72 byte로 memory가 aligned되기 때문에 struct가 가운데서 짤리게 된다. 그래서 프로그램을 어떻게 짜느냐에 따라 `race condition, false sharing 문제`가 나올 수 있다.
일단 병렬프로그램일 때 이 문제를 해결하기 위해 `c++11부터 alignas specifier를 사용`할 수 있다.
또한 `compiler 중에서도 padding 옵션을 끄고 compact한 struct를 만들어주는 옵션`도 있다.




