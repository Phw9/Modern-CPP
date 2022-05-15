## Variable types


- variable types는 compiler, architecture 마다 다를 수 있다.


이러한 문제를 두 가지의 방법으로 차단할 수 있는데 첫 번째 방법으로 compilation assertion을 이용하는 것이다.
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

두 번째 방법으로는 Fixed width variable types의 방법으로 사이즈를 정해서 사용하는 방법이다.
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


