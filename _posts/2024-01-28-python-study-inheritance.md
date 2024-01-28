---
title: "[TIL] Python Study - Inheritance"
excerpt: "Inheritance, OOP, class, super(), range(), len()"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-28T23:54
---

## Inheritance

---

Category: `Python`, `OOP`

```python
class Puppy:

  def __init__(self, name, breed, age):
    self.name = name
    self.breed = breed
    self.age = age

  def woof_woof(self):
    print("Woof Woof!")

  def introduce(self):
    self.woof_woof()
    print(f"My name is {self.name} and I am a baby {self.breed}")
    self.woof_woof()

teemo = Puppy(
  name="Teemo",
  breed="Goldendoodle",
  age=1.6
)

teemo.introduce()
```

- class는 함수들을 묶어주는 역할을 한다.
  - `class ClassName:` 클래스 이름은 첫글자 대문자 표기
- `def __init__(self)` 은 객체(object)의 속성(attributes)을 초기화하고 초기 상태를 설정한다.
  - `def __init__(self, 속성1, 속성2, 속성3):
self.속성1 = 값 
self.속성2 = 값 
self.속성3 = 값`
- `objectname = Classname(
  속성1="값",
  속성2="값',
  속성3="값
)` 객체 이름은 소문자 표기
- 함수를 호출할 땐 `objectname.function()`

### 클래스에 중복이 많을 때는 어떻게 처리할까?

```python
class GuardDog:

  def __init__(self, name, breed):
    self.name = name
    self.breed = breed
    self.age = 5

  def rrrrr(self):
    print("stay away!")

class Puppy:

  def __init__(self, name, breed, age):
    self.name = name
    self.breed = breed
    self.age = age

  def woof_woof(self):
    print("Woof Woof!")
```

- GuardDog 클래스와 Puppy 클래스는 둘 다 Dog 라는 공통점이 있다.
- 이를 뽑아내 부모 클래스를 생성할 수 있다.

```python

class Dog:

  def __init__(self, name, breed, age):
    self.name = name
    self.breed = breed
    self.age = age

class GuardDog(Dog):

  def rrrrr(self):
    print("stay away!")

class Puppy(Dog):

  def woof_woof(self):
    print("woof woof!")


teemo = Puppy(
  name="Teemo",
  breed="Goldendoodle",
)

leo = GuardDog(
  name="Leo",
  breed="Shepard"
)

print(leo)

# 출력:
# TypeError: Dog.__init__() missing 1 required
# positional argument: 'age'
```

- Dog 클래스를 생성해 공통되는 속성을 뽑아 `__init__` 메서드로 초기화하고 GuardDog 클래스와 Puppy 클래스의 init 메서드를 삭제했다.
- 이 때 실행하면 콘솔 창에서 타입 에러를 보게 되는데, **`__init__`** 메서드가 호출될 때 **`age`**라는 매개변수에 대한 위치 인수(값)가 누락되었다는 의미이다.

```python
class Dog:

  def __init__(self, name, breed, age):
    self.name = name
    self.breed = breed
    self.age = age

  def sleep(self):
    print("zZzZzZzZzZ.....")

class GuardDog(Dog):

  def __init__(self, name, breed):
    super().__init__(
      name,
      breed,
      5,
    )

  def rrrrr(self):
    print("stay away!")

class Puppy(Dog):
  def __init__(self, name, breed):
    super().__init__(
      name,
      breed,
      0.1,
    )

  def woof_woof(self):
    print("woof woof!")


teemo = Puppy(
  name="Teemo",
  breed="Goldendoodle",
)

leo = GuardDog(
  name="Leo",
  breed="Shepard"
)

teemo.sleep()
leo.sleep()
```

- `super()` 파이썬 함수. 상속 관계에서 ChildClass 에서 ParentClass 의 메서드를 호출할 때 사용된다. ChildClass 에서 ParentClass 의 메서드를 오버라이드(재정의)하고, 그 오버라이드된 메서드에서 ParentClass 의 메서드를 호출할 때 사용된다.

### PEP 8 (Python Enhancement Proposal 8)

> [When to Use Trailing Commas](https://peps.python.org/pep-0008/#when-to-use-trailing-commas)

쉼표를 선택적으로 뒤에 붙이는 스타일은 PEP 8 (Python Enhancement Proposal 8) 스타일 가이드에 명시되어 있다. PEP 8은 파이썬 코드의 스타일과 구조에 관한 권장 사항을 제시하는 공식 문서이다. PEP 8에서는 여러 줄로 나열된 요소들의 마지막에 쉼표를 추가하는 것을 권장하고 있다.

여러 줄로 나열된 요소들에 대한 관련 내용은 PEP 8 문서의 "When to use trailing commas" 섹션에 기술되어 있다. 이 부분에서는 여러 줄로 나열된 요소들의 마지막에 쉼표를 추가하는 것이 가독성과 유지보수성을 향상시킬 수 있다고 설명하고 있다.

예를 들어, 다음은 PEP 8에 따라 여러 줄의 요소를 나열할 때 쉼표를 사용하는 예시이다:

```python
pythonCopy code
# PEP 8 스타일
colors = [
    'red',
    'green',
    'blue',
]

# 쉼표를 사용하지 않은 경우
colors = [
    'red',
    'green',
    'blue'
]

```

쉼표를 사용하면 요소를 추가하거나 제거할 때 코드 변경이 용이하며, 코드 리뷰나 버전 관리 시에도 변경 내용이 명확하게 표시된다. 이러한 이유로 PEP 8에서는 쉼표를 여러 줄로 나열된 요소들의 마지막에 추가하는 것을 권장하고 있다.

## Python Code Test Practice

> [PCCE 기출문제] 6번 / 가채점 (디버깅 문제)

```python
def solution(numbers, our_score, score_list):
    answer = []
    for i in range(len(numbers)):
        if our_score[i] == score_list[numbers[i]-1]:
            # debug: numbers[our_score[i]] == score_list[i]
            answer.append("Same")
        else:
            answer.append("Different")

    return answer
```

```python
'''
range()는 0부터 주어진 숫자 직전까지의 정수를 생성하는 함수:
리스트처럼 동작하지만 리스트가 아니다.
이런 객체를 '이터러블' 이라고 부른다. for 루프와 같이 쓰인다.
'''

for i in range(3):
    print(i)
# 출력:
# 1
# 2
# 3

'''스텝(step)을 지정하는 것도 가능하다.'''

for i in range(2, 10, 2):
    print(i)
# 출력:
# 2
# 4
# 6
# 8

'''len()은 시퀀스의 길이를 반환하는 함수:'''

my_list = [a, b, c]
length_of_list = len(my_list)
print(length_of_list)
# 출력:
# 3
```
