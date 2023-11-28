# Java 학습자를 위한 C 언어의 포인터 이해

- C언어를 가르치는 대부분의 서적이나 강의에서는, `포인터`(pointer)라는 개념을 설명합니다.
- Java의 자료형에는 `기본 자료형`(primitive type)과 `참조 자료형`(reference type)이 있습니다.
- 이 중, `참조 자료형`의 작동방식은 `포인터`의 작동방식과 유사한 면이 있는데, 이를 제대로 이해하지 못한 채, `참조 자료형`을 다루면서 혼란스러워하는 학생들이 많은 것으로 보입니다.

이 강의는 [`포인터가 필요한 이유`](#포인터가-필요한-이유)와 [`Java의 참조`](#java의-참조)라는 2가지의 대주제를 가지고 있습니다.

---

# 포인터가 필요한 이유?

1. 배열의 사용
2. 리스트, 트리 등 연결된 데이터 구현
3. 지역변수와 메모리
4. 매개변수와 메모리

## 배열의 사용

배열은 기본적으로 포인터와 밀접한 관련이 있습니다.

```c
#include <stdio.h>

// 구조체 선언
struct Test
{
    char a;
    int b;
    float c;
};

int main(void)
{
    // 구조체 배열 선언
    struct Test testArr[3];

    // 배열 변수 출력
    printf("%p\n", testArr);
    // 배열 변수의 주소 출력
    printf("%p\n", &testArr);
    // 배열 변수의 첫번째 원소의 주소 출력
    printf("%p\n", &testArr[0]);

    return 0;
}
```

- 위의 예제는 3개의 동일한 주소값을 출력합니다.

> 배열 변수는 기본적으로 자신의 주소를 가르키고 있으므로, `참조 연산자`(&)의 유무와 무관하게 동일한 값을 출력합니다.

또한 배열의 요소는 `산술 연산`를 통해서 접근하는 것이 가능합니다.

```c
#include <stdio.h>

// 구조체 배열 선언
struct Test
{
    char a;
    int b;
    float c;
};

int main(void)
{
    // 구조체 배열 선언
    struct Test testArr[3];

    // 구조체 포인터 선언 및 배열의 주소를 할당
    struct Test *testPtr = testArr;
    while (&testArr[(sizeof(testArr) / sizeof(testArr[0]) - 1)] - testPtr >= 0)
    {
        //포인터의 값을 출력함과 동시에 연산자 적용
        printf("%p\n", testPtr++);
    }

    return 0;
}
```

- 결국 배열의 `각 요소`는 `배열 변수의 주소`로 부터 일정한(원소의 data type의 크기) 간격을 두고 나열된 `포인터`들의 집합임을 알 수 있습니다.

## 리스트, 트리 등 연결된 데이터 구현

```c
#include <stdio.h>
#include <stdlib.h>

// 리스트의 요소가 될 구조체 선언
struct Test
{
    char a;
    // 연결할 요소의 주소를 가르키는 포인터 변수
    struct Test *next;
};

// 요소를 추가하는 함수 정의
void pushList(struct Test **currentPtr)
{
    // 새로운 요소에 동적으로 메모리 할당
    struct Test *newPtr = (struct Test *)malloc(sizeof(struct Test));
    // 새로운 요소의 포인터 변수에 기존의 요소를 할당
    newPtr->next = *currentPtr;
    // 넘겨받은 인자를 새로운 인자로 대체
    *currentPtr = newPtr;
}

// 요소가 가진 값을 출력하고 제거하는 함수 정의
void popList(struct Test **currentPtr)
{
    // 현재 요소의 포인터 변수가 가진 주소를 복사
    struct Test *nextPtr = (*currentPtr)->next;
    printf("%c\n", (*currentPtr)->a);
    // 할당된 메모리 반환
    free((*currentPtr));
    // 현재 요소에 복사해두었던 주소로 할당
    (*currentPtr) = nextPtr;
}

int main(void)
{
    // 리스트의 길이가 될 변수 선언 및 초기화
    int length = 5;
    // 리스트 변수를 선언 및 NULL로 초기화
    struct Test *testList = NULL;

    int i = 0;
    while (i < length)
    {
        pushList(&testList);
        testList->a = (char)(i + 65);
        i++;
    }

    while (testList != NULL)
    {
        popList(&testList);
    }

    return 0;
}
```

위의 예제는 `단일 연결 리스트`라고 불리는 자료 구조의 일종입니다.

데이터들을 연결하여 관리함에 있어서 포인터의 개념이 중요하게 사용된다는 점만을 주목합니다.

> 자료 구조와 관련된 세부적인 내용은 수업의 주제를 벗어나므로 이 정도로만 설명하겠습니다.

## 지역변수와 메모리

지역변수와 관련된 이야기를 하기에 앞서, stack memory의 작동방식에 대한 이해가 필요합니다.

- C 코드 예제

```c
#include <stdio.h>

int main(void)
{
    int a = 0;
    if (1 == 1)
    {
        int b = 1;
        if (1 == 1)
        {
            int c = 2;
            printf("%d\n", c);
        }
        printf("%d\n", b);
    }
    printf("%d\n", a);
}
```

- Java 코드 예제

```java
public class Main {
    public static void main(String[] args) {
        int a = 0;
        if (1 == 1) {
            int b = 1;
            if (1 == 1) {
                int c = 2;
                System.out.println(c);
            }
            System.out.println(b);
        }
        System.out.println(a);
    }
}
```

- `stack`에 상태

|`stack`||||||
|:---:|:---:|:---:|:---:|:---:|:---:|
|`0xe0`|||c|
|`0xfc`||b|b|b|
|`0xff`|a|a|a|a|a|

> `FILO` (First In, Last Out) - 선입후출
>
> Stack memory는 선입후출되는 방식의 메모리로써, `Scope`(스코프) 또는 `block`(블럭) 내에서 선언된 변수를 stack의 높은 주소에서부터 추가하고(`push`), 해당 스크프를 빠져나올 때, 제거하는(`pop`) 방식으로 작동합니다.

이제 `동적 메모리 할당`을 사용하는 것이, 왜 memory 관리 효율을 높이는가에 대한 이해가 필요합니다.

- 사이즈가 큰 data를 stack memory에 추가하는 예시

```c
#include <stdio.h>

struct Test
{
    char a;
    int b;
    float c;
};

int main(void)
{
    // Test 구조체를 원소로 갖는 배열 선언.
    struct Test testArr[3];

    for (int i = 0; i < (sizeof(testArr) / sizeof(testArr[0])); i++)
    {
        testArr[i].a = (char)(i + 65);
        testArr[i].b = i;
        testArr[i].c = (float)i;
    }

    for (int i = 0; i < (sizeof(testArr) / sizeof(testArr[0])); i++)
    {
        printf("a=%c\n", testArr[i].a);
        printf("b=%d\n", testArr[i].b);
        printf("c=%f\n", testArr[i].c);
    }

    return 0;
}
```

> Java의 경우, 기본 자료형(primitive type)을 제외한 data type은 내부적으로 포인터와 같이 작동하므로, 위와 같은 방식으로 코드를 작성할 수 없습니다.

- 사이즈가 큰 data를 heap memory에 추가하는 예시

```c
#include <stdio.h>
#include <stdlib.h>

struct Test
{
    char a;
    int b;
    float c;
};

int main(void)
{
    // 포인터를 원소로 담을 포인터 배열을 선언.
    struct Test *testArr[3];

    for (int i = 0; i < (sizeof(testArr) / sizeof(testArr[0])); i++)
    {
        // 할당받은 heep memory 영역의 주소를
        // 포인터변수(포인터 배열의 원소)에 대입.
        testArr[i] = (struct Test *)malloc(sizeof(struct Test));
        testArr[i]->a = (char)(i + 65);
        testArr[i]->b = i;
        testArr[i]->c = (float)i;
    }

    for (int i = 0; i < (sizeof(testArr) / sizeof(testArr[0])); i++)
    {
        printf("a=%c\n", testArr[i]->a);
        printf("b=%d\n", testArr[i]->b);
        printf("c=%f\n", testArr[i]->c);
    }

    // 동적으로 할당한 메모리를 해제.
    for (int i = 0; i < (sizeof(testArr) / sizeof(testArr[0])); i++)
    {
        free(testArr[i]);
    }

    return 0;
}
```

> 위의 두 예제는 결과적으로 같은 결과물을 제공합니다
>
> 하지만 memory를 사용하는 방식은 전혀 다릅니다.

- `testArr` 변수를 stack memory에 쌓는 경우

|`stack`|stack memory에 추가된 `size` (bytes)|`variable`|
|:---:|:---:|:---:|
|`0xe7`|`sizeof(char) + sizeof(int) + sizeof(float)`<br>1 + 4 + 4 = 9|testArr[3]|
|`0xf3`|`sizeof(char) + sizeof(int) + sizeof(float)`<br>1 + 4 + 4 = 9|testArr[2]|
|`0xff`|`sizeof(char) + sizeof(int) + sizeof(float)`<br>1 + 4 + 4 = 9|testArr[1]|

> 실제로 `sizeof(testArr)`를 출력하면 `12`를 출력하는 것을 확인할 수 있을 것입니다.
>
> 이는 컴파일러가 프로세스의 효율을 위해서, `Data structure alignment`(데이터 구조 정렬)이라고 하는 작업을 수행하였기 때문입니다.
>
> 지금 수업의 내용과는 다소 거리가 있으므로, 이와 관련된 설명은 생략하겠습니다.

- `testArr` 변수를 heap memory에 쌓는 경우

|`stack`|stack memory에 추가된 `size` (bytes)|`variable`|heap memory에 추가된 `size` (bytes)|
|:---:|:---:|:---:|:---:|
|`0xe7`|4 or 8|testArr[3]|`sizeof(char) + sizeof(int) + sizeof(float)`<br>1 + 4 + 4 = 9|
|`0xfb`|4 or 8|testArr[2]|`sizeof(char) + sizeof(int) + sizeof(float)`<br>1 + 4 + 4 = 9|
|`0xff`|4 or 8|testArr[1]|`sizeof(char) + sizeof(int) + sizeof(float)`<br>1 + 4 + 4 = 9|

> `stack`에 추가되는 포인터의 크기는 `compiler`(컴파일러)가 32-bit라면 4bytes, 64-bit라면 8byte의 크기를 갖는다.
> 
> Windows 환경에서 `MinGW`를 통해 설치할 경우, 기본적으로 32비트의 컴파일러가 설치된다.


- 앞서 언급한 것과 같이 stack memory는 선입후출 방식으로 작동합니다.

- 때문에 특정 변수를 위해 할당받은 영역을 임의로 반환하는 것이 불가능합니다.

## 매개변수와 메모리

구조체나 클래스와 같이 사이즈가 큰 data type을 원소로 갖는 배열이 있고,

해당 배열을 각 원소의 주소값을 출력해 보려고 한다고 가정해봅니다.

```c
#include <stdio.h>

// 구조체 선언
struct Test
{
    char a;
    int b;
    float c;
};

// 구조체를 매개변수로 받아 주소를 출력하는(??) 함수 선언
void myFunction(struct Test test)
{
    printf("%p\n", &test);
}

int main(void)
{
    struct Test testArr[3];

    // 반복문을 통해 배열의 각 요소에 접근하면서 함수를 실행
    for (int i = 0; i < (sizeof(testArr) / sizeof(testArr[0])); i++)
    {
        myFunction(testArr[i]);
    }

    return 0;
}
```

위의 예제는 `기능`적인 차원에서의 문제와 `효율`적인 차원에서의 문제를 각각 하나씩 가지고 있습니다.

*기능적인 문제*

- 기능적인 문제는, 배열의 각 원소의 주소를 출력하고자 하였으나, *동일한 주소값*만이 반복되어서 출력된다는 문제입니다.
- 이는 매개변수의 작동원리와 관련이 있습니다. 매개변수는 해당 스코프 내에서 일종의 지역변수와 같이 사용되며, 인자로 넘겨받은 변수로부터 `복사`한 값을 `할당`받게 됩니다.
- 위의 예제에서 `myfunction` 함수를 호출되는 과정은, 마치 다음과 같은 스코프가 실행되는 것과 유사합니다.

```c
{
    struct Test test = testArr[i];
    printf("%p\n", &test);
}
```

- 이 코드가 같은 값만을 출력했던 이유는, `myFunction`함수의 `test` 매개변수가 반복문에 의해서 stack memory에 *추가*되었다가 *제거*되기를 반복했기 때문입니다.

|`scope`||||||
|:---:|:---:|:---:|:---:|:---:|:---:|
|`for문`||`test`<br>=<br>`testArr[0]`|`test`<br>=<br>`testArr[1]`|`test`<br>=<br>`testArr[2]`|
|`main함수`|`testArr`|`testArr`|`testArr`|`testArr`|`testArr`|

*효율적인 문제*

- 효율적인 문제는, 매개변수의 데이터 사이즈가 크다면, 지역변수의 경우와 동일하게, stack memory의 영역을 많이 차지하게 된다는 것입니다.

포인터를 사용해 다음과 같이 수정할 수 있습니다.

```c
#include <stdio.h>

// 구조체 선언
struct Test
{
    char a;
    int b;
    float c;
};

// 구조체의 주소를 매개변수로 받아 출력하는 함수 선언
void myFunction(struct Test *test)
{
    printf("%p\n", test);
}

int main(void)
{
    struct Test testArr[3];

    // 반복문을 통해 배열의 각 요소에 접근하면서 함수를 실행
    for (int i = 0; i < (sizeof(testArr) / sizeof(testArr[0])); i++)
    {
        // 인자로써 주소값을 지정
        myFunction(&testArr[i]);
    }

    return 0;
}
```

---

# Java의 참조

1. Java가 `참조 자료형` 변수를 선언하는 방식
2. Java의 `참조 자료형` 변수에 다른 변수를 할당(assign)하는 경우

## Java가 참조 자료형 변수를 선언하는 방식

```java
public class Main {
    public static void main(String[] args) {
        // 지역변수로써 클래스 객체 선언 및 초기화
        Test test = new Test();
        test.a = 'A';
        test.b = 1;
        test.c = 3.141592f;
        System.out.println(test.a); // 'A'
        System.out.println(test.b); // 1
        System.out.println(test.c); // 3.141592
    }
}

// 클래스 선언
class Test {
    public char a;
    public int b;
    public float c;
}
```

- 이 예제의 경우에도 지역변수 `test`는 9 Bytes(정확히는 `8-byte alignment`에 의한 16 Bytes)의 stack memory 공간을 차지하고 있는 것이 아닙니다.
- `test`변수는 일종의 포인터로써 작동하며, 실제 객체는 heap memory 공간에 할당되어 있습니다.

동일한 코드를 C로 작성하면 다음과 같습니다.

```c
#include <stdio.h>
#include <stdlib.h>

// 구조체 선언
struct Test
{
    char a;
    int b;
    float c;
};

int main(void)
{
    // 포인터인 지역변수를 선언 및 동적 메모리 할당
    struct Test *test = (struct Test *)malloc(sizeof(struct Test));
    test->a = 'A';
    test->b = 1;
    test->c = 3.141592;

    printf("%c\n", test->a); // 'A'
    printf("%d\n", test->b); // 1
    printf("%f\n", test->c); // 3.141592

    // 할당된 메모리 반납(또는 회수)
    free(test);

    return 0;
}
```

## Java의 참조 자료형 변수에 다른 변수를 할당(assign)하는 경우

> 앞서 설명한 모든 내용들은, 이 주제를 이해하기 위해서 필요한 사전 지식이었습니다.
>
> 만약에, 여기까지의 내용을 완벽하게 이해하셨다면, 이 주제는 따로 다루지 않아도 무방할지 모릅니다.
>
> 하지만, 논리를 이해하고 있더라도 익숙하지 않은 개념이라면 혼동이 오기 쉽습니다.
>
> 이 주제가 강의의 핵심인 만큼 명확하게 설명하고자 합니다.

Java에서는 `참조 자료형` 변수에 다른 변수를 할당할 때, 변수 자체가 아니라 해당 객체의 참조(reference)가 할당됩니다. 이는 C의 포인터와 유사한 개념으로써, 변수가 가지고 있는 `주소값`을 넘겨주게 됩니다.

```java
public class Main {
    public static void main(String[] args) {
        // 첫 번째 객체 생성
        Test test1 = new Test();
        test1.a = 'A';
        test1.b = 1;
        test1.c = 3.141592f;

        // 두 번째 객체 생성
        Test test2 = new Test();
        test2.a = 'B';
        test2.b = 2;
        test2.c = 6.283185f;

        // test1의 참조를 test2에 할당
        test2 = test1;

        // test2의 값을 변경하면 실제로 test1의 객체가 변경됨
        test2.a = 'C';

        // test1 출력
        System.out.println(test1.a); // 'C'
        System.out.println(test1.b); // 1
        System.out.println(test1.c); // 3.141592

        // test2 출력
        System.out.println(test2.a); // 'C'
        System.out.println(test2.b); // 1
        System.out.println(test2.c); // 3.141592
    }
}

// 클래스 선언
class Test {
    public char a;
    public int b;
    public float c;
}
```

이 예제에서 `test2 = test1`은 test2가 test1이 참조하고 있는 객체를 참조하도록 만듭니다. 따라서 이후에 test2의 값을 변경하면, 실제로는 test1의 객체가 변경됩니다.

Java에서는 객체의 복사가 아닌 참조(reference)를 다루므로 이러한 동작이 발생합니다. 이는 초심자에게 다소 혼란스러운 개념일 수는 있으나, 객체의 크기가 큰 경우 메모리 관리 및 성능 측면에서 효율적일 수 있습니다.

이것으로 강의를 마무리하겠습니다.
