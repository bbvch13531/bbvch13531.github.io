---
title: Swift-memory-experiment
date: 2023-06-30 18:20:15
tags:
---
# Swift에서 Class의 멤버 변수인 Struct의 메모리 공간을 알아보는 실험

Swift에서 Value type인 Struct은 Stack space에 할당된다고 알려져 있고,
Reference type인 Class는 Heap space에 할당된다고 알려져 있습니다.

그렇다면 Class의 멤버변수로 정의된 Struct은 어느 메모리 공간에 할당될 것인가? 에 대한 의문이 생겨서 시작하게 되었습니다.

--------
실험환경은 MacOS 12.6, 맥북프로 M1 16인치, M1 Pro 칩셋, Xcode 14.1, Swift 5.7.1 입니다.

### 예제코드

ClassA의 멤버 변수인 InnerStruct 타입인 innerStruct가 어떤 메모리 공간에 할당되는지 확인하는 코드입니다.

```swift
class ClassA {
  var innerStruct: InnerStruct
  init(innerStruct: InnerStruct) {
    self.innerStruct = innerStruct
  }
}

struct InnerStruct {
  var name: String

  init(name: String) {
    self.name = name
  }
}

struct Memory {
    @inlinable static func dump<T>(variable: inout T) {
        withUnsafePointer(to: &variable) { print($0) }
    }

    @inlinable static func dump(object: AnyObject) {
        print(Unmanaged.passUnretained(object).toOpaque())
    }
}

var a = ClassA(innerStruct: InnerStruct(name: "Hello"))

print("Memory address of Class a")
Memory.dump(object: a)

print("Memory address of innerStruct")
Memory.dump(variable: &(a.innerStruct))
```

여기서 `Memory.dump(variable:)` 는 변수의 메모리 주소 값을 출력하는 함수이고,

`Memory.dump(object:)` 는 `AnyObject` 타입 객체의 주소 값을 출력하는 함수입니다.

--------
### 실행 결과

```
Memory address of Class a
0x0000600000201fe0
Memory address of innerStruct
0x0000600000201ff0
```

Class a의 메모리 주소와 innerStruct의 메모리 주소가 인접한 위치에 있는 것 같습니다.
좀 더 정확하게 살펴보기 위해 Xcode의 'Debug' - 'Debug Workflow' - 'View Memory'를 눌러 'Memory Document'를 보겠습니다.

Class a의 메모리 `0x0000600000201fe0`의 내용은 다음과 같습니다.
<img width="1433" alt="image" src="https://user-images.githubusercontent.com/9498744/210127253-e3d83eb8-b5c5-4be3-8267-920da1ebbec1.png">

innerStruct의 메모리 `0x0000600000201ff0`의 내용은 다음과 같습니다.
`name` 프로퍼티의 값 "Hello"가 있는 것을 확인할 수 있습니다.
<img width="1434" alt="image" src="https://user-images.githubusercontent.com/9498744/210127254-56073361-1cf3-4786-a7b6-2279ed50913b.png">

`0x0000600000201fe0`와 `0x0000600000201ff0`의 메모리 공간을 살펴보기 위해 `vmmap` 커맨드를 이용합니다.

```
$ vmmap { 해당 프로세스의 PID } | grep Malloc

실행결과
REGION TYPE                    START - END         [ VSIZE  RSDNT  DIRTY   SWAP] PRT/MAX SHRMOD PURGE    REGION DETAIL
MALLOC metadata             1002dc000-1002e0000    [   16K    16K    16K     0K] r--/rwx SM=COW          MallocHelperZone_0x1002dc000 zone structure
MALLOC metadata             10046c000-100470000    [   16K    16K    16K     0K] r--/rwx SM=COW          DefaultMallocZone_0x10046c000 zone structure
MALLOC_TINY                 100500000-100600000    [ 1024K    32K    32K     0K] rw-/rwx SM=PRV          MallocHelperZone_0x1002dc000
MALLOC_TINY                 100600000-100700000    [ 1024K    16K    16K    16K] rw-/rwx SM=PRV          MallocHelperZone_0x1002dc000
MALLOC_SMALL                100800000-101000000    [ 8192K    16K    16K    16K] rw-/rwx SM=PRV          MallocHelperZone_0x1002dc000
MALLOC_SMALL                101000000-101800000    [ 8192K    16K    16K    16K] rw-/rwx SM=PRV          MallocHelperZone_0x1002dc000
MALLOC_NANO              600000000000-600008000000 [128.0M   176K   176K   288K] rw-/rwx SM=PRV          DefaultMallocZone_0x10046c000
```

--------

### 결론

MALLOC_NANO 의 Start - End에 `Class a`와 `innerStruct`의 메모리 주소가 포함되는 것을 확인할 수 있습니다.

따라서 "Class의 멤버변수로 정의된 Struct은 어느 메모리 공간에 할당될 것인가?" 에 대한 질문은
"Class의 생성과 함께 Struct가 초기화된다면 **Heap 공간에 할당된다"** 라는 결론을 내릴 수 있을 것 같습니다.

Class의 생성 이후 Struct의 값이 바뀌거나, 새로운 Struct를 할당하고 대입하게 된다면 Swift의 Value type의 동작과 마찬가지로
Stack space에 새로 할당되고, 그 주소의 값이 할당되게 됩니다.

--------
### 추가

Heap과 Stack 공간의 주소가 다르게 나오는지 확인하기 위해서 Value type인 Int와 String에 대해서도 확인해보겠습니다.

```swift
var message = "Hello"
var number = 3

print("Memory address of message")
Memory.dump(variable: &message)

print("Memory address of number")
Memory.dump(variable: &number)
```

--------

### 실행결과
```
Memory address of message
0x000000016fdff250
Memory address of number
0x000000016fdff248
```

`message`의 메모리 공간

<img width="1434" alt="image" src="https://user-images.githubusercontent.com/9498744/210127658-ea27a83d-b1fe-440e-a4e1-aee0b9cbe85d.png">

`number`의 메모리 공간, `16FDFF248`의 첫번째 바이트에 '03' 의 값을 확인할 수 있음

<img width="1436" alt="image" src="https://user-images.githubusercontent.com/9498744/210127663-0a89bfc2-a517-49c1-b89a-3ef5db8d10c1.png">

```
$ vmmap { 해당 프로세스의 PID } | grep Stack

REGION TYPE                    START - END         [ VSIZE  RSDNT  DIRTY   SWAP] PRT/MAX SHRMOD PURGE    REGION DETAIL
Stack                       16f604000-16fe00000    [ 8176K    32K    32K     0K] rw-/rwx SM=PRV          thread 0
Stack                               8176K      32K      32K       0K       0K       0K       0K        1
```

해당 프로세스의 Stack 메모리 공간은 `16f604000-16fe00000` 이고, message와 number는 Stack 메모리 공간에 할당되는 것을 확인할 수 있음.




### 출처

Memory dump코드와 메모리 공간에 대한 내용은 [godrm](https://github.com/godrm)님의 글을 참고했습니다.
https://medium.com/@jungkim/%EC%8A%A4%EC%9C%84%ED%94%84%ED%8A%B8-%ED%83%80%EC%9E%85%EB%B3%84-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%B6%84%EC%84%9D-%EC%8B%A4%ED%97%98-4d89e1436fee

copy on write에 대한 내용은 해당 글을 참고했습니다.
https://medium.com/@iostpointblog/what-is-copy-on-write-18701742e60c


