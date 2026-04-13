#Java

# Execution Engine

**Execution Engine**은 **JVM**에서 **바이트코드를 실제로 실행**하는 핵심 구성 요소로, `Interpreter`와 `JIT Compiler`를 통해 실행 성능을 최적화하는 역할을 담당한다. 또한 실행 과정에서 `메모리 관리(GC)`와 `네이티브 코드 연동`까지 수행한다.
**Execution Engine**의 내부 구조와 각 구성 요소의 동작 방식, 그리고 Java가 높은 실행 성능을 확보하는 원리를 살펴보자

---

## Execution Engine의 구조

[![1-dr-Ua6ici-Iselj-LXd7OZDA.webp](https://i.postimg.cc/8CY1rpzX/1-dr-Ua6ici-Iselj-LXd7OZDA.webp)](https://postimg.cc/5HwchMv8)

**Execution Engine**에는 위와 같은 구조를 갖고있다.

- **Interpreter**
  바이트코드를 한 줄씩 읽어 즉시 실행하는 컴포넌트
- **JIT Compiler**
  반복적으로 호출되는 코드를 감지하여 Native 코드로 변환한다.
- **Garbage Collector**
  Heap 영역에서 더 이상 사용되지 않는 객체를 자동으로 회수하여 메모리를 효율적으로 관리하는 컴포넌트
- **Native Method Interface** (JIN)
  Java 코드에 네이티브 코드를 호출할 수 있도록 외부 환경을 연결하는 인터페이스

---

## Interpreter

**Interpreter**은 JVM Execution Engine에서 바이트 코드를 한 줄씩 읽어 즉시 실행하는 역활을 한다. 여기서 별도의 컴파일을 거치지 않고 즉시 실행되기 때문에 프로그램을 빠르게 실행할 수 있다는 장점이 존재한다.

### 동작 방식

```
바이트코드 로드
   ↓
한 줄씩 읽기 (Fetch)
   ↓
명령어 해석 (Decode)
   ↓
즉시 실행 (Execute)
   ↓
다음 명령어 반복
```

동작 방식은 CPU처럼 바이트 코드를 순차적인 단계를 반복한다.

### 장점과 단점

- 장점
  별도의 컴파일 없이 바이트 코드를 읽고 즉시 실행하기 때문에 실행이 빠르다는

- 단점
  동일한 코드 또는 반복작업에 있어 계속 다시 읽고 해석하시 떄문에 비효율적인 부분이 존재

### Interpreter의 한계점

동작 방식과 단점을 보았을 떄 **Interpreter**의 한계는 명확하게 드러난다.

```
반복 실행 ->  비용 누적 -> CPU 실행 대비 비효율적 -> 최적화 불가능
```

이러한 한계를 보완하기 위해서 **JIT Compiler**가 존재한다.

---

## JIT Compiler

**JIT(Just-In-Time) Compiler**는 **Interpreter** 방식의 성능 한계를 보완하기 위해 설계되었다.
반복적으로 실행되는 바이트코드를 네이티브 코드로 컴파일하여 실행 속도를 향상시키는 컴포넌트이다.
프로그램 실행 중(Runtime)에 동적으로 최적화를 수행한다는 점에서 정적 컴파일과 구별된다.

### JIT Compiler의 역활과 동작

앞에서 **Interpreter**의 한계를 보완하기 위해 자주 사용되는 바이트 코드를 Native Code로 변환하여 메모리에에 저장하고, 이를 재사용 한다.

```
바이트코드 실행 (Interpreter)
        ↓
실행 횟수 증가 (HotSpot 감지)
        ↓
JIT Compiler 동작
        ↓
Native Code 생성
        ↓
Code Cache 저장
        ↓
이후 Native Code로 직접 실행
```

### **HotSpot Detection**

**JIT Compiler**는 모든 코드를 컴파일 하지 않고 자주 사용되는 코드만을 컴파일한다.

- 판단 기준
  - 메서드 호출 횟수
  - 루프 반복 횟수
    일정 횟수 기준에 해당하면 **Hot Method**로 판단

### Code Cahce

**Code Cahce**같은 경우에는 **JIT Compiler**가 생성한 Native Code를 **Code Cache**영역에 저장이 된다.
이를 통하여 재컴파일을 방지한다.

### **Tiered Compilation**

현대의 JVM은 단일의 JIT 구성이 아닌 **단계별 컴파일 전략**을 사용한다.

```
Interpreter
   ↓
C1 Compiler (빠른 컴파일, 낮은 최적화)
   ↓
C2 Compiler (느린 컴파일, 높은 최적화)
```

- C1 Compiler
  - 빠르게 컴파일
  - 기본적인 최적화 수행
  - 초기 성능 개선 목적

- C2 Compiler
  - 더 많은 분석 수행
  - 고급 최적화 적용
  - 장기 실행 성능 극대화

> _왜 Tiered Compilation의 구조를 갖게 되었을까?_

### 초기 Compilation의 구조

```
Interpreter 실행
   ↓ (HotSpot 감지)
JIT Compiler (C1 또는 C2 중 하나)
   ↓
Native Code
```

초기 구조에서는 단계별 구조를 갖고 있지 않고 HotSpot이라고 판단이 될 경우 메모리에 저장하여 Native Code으로 실행하는 단일의 구조를 갖고 있다.

_(여기서 여전히 C1, C2가 존재한다. 그래서 뭔 차이가 있는지 명화히 몰랐다.)_

초기 Compilation에서 JVM 실행 시 옵션에 따라 하나만을 사용하는 동작이다.

- -client → C1 Compiler
- -server → C2 Compiler

여기서 초기 구조에 대한 문제점이 대두되었다.

- C1만 사용할 경우
  빠르게 컴파일이 된다는 장점이 존재 한다. 하지만 최적화 수준이 낮았다.
  -> 장기적인 실행에 있어 성능이 떨어진다.
- C2만 사용할 경우
  최적화가 뛰어난다. 하지만 컴파일 속도 부분에서 느리다.
  -> 초기 실행에 있어 느리다.

**이와 같은 문제로 Java 8에서 Tiered Compilation이 적용되었다.**

|  **구분**   | **초기 JIT** | **Tiered JIT** |
| :---------: | :----------: | :------------: |
| 컴파일 방식 |  단일 단계   |     다단계     |
|  컴파일러   | 하나만 사용  |    C1 + C2     |
| 최적화 전략 |     없음     |     점진적     |
|  성능 균형  |    불가능    |      가능      |

### **JIT의 주요 최적화 기법**

- Inline Expansion (인라이닝)
  - 메서드 호출 제거
  - 호출 비용 감소
- Loop Optimization
  - 불필요한 반복 제거
  - 루프 전개 (Loop Unrolling)
- Dead Code Elimination
  - 실행되지 않는 코드 제거
- Escape Analysis
  - 객체가 스레드 밖으로 나가지 않으면
    → Heap 대신 Stack에 할당

---

## Garbage Collector

**Garbage Collector(GC)**는 JVM에서 Heap 영역의 객체 생명주기를 관리하며, 더 이상 참조되지 않는 객체를 자동으로 회수하여 메모리를 효율적으로 사용하는 역할을 수행한다
개발자가 직접 메모리를 해제하지 않아도 된다는 점에서 Java의 중요한 특징이다.

### GC의 필요성

Java에서는 객체를 생성할 수는 있지만, 명시적으로 메모리를 해제할 수는 없다.

```
Object obj = new Object();
obj = null;
```

객체를 더 이상 사용하지 않지만, 메모리는 해제되지 않는다. 그리고 추가적으로 앞으로 메모리 해제가 계속 되지 않는다라고 가정을 하면 다음과 같은 문제들이 발생하게 된다.

- 메모리 누수 (Memory Leak)
- Out Of Memory Error
- 시스템 성능 저하

따라서 **JVM**은 더 이상 사용되지 않는 객체를 메모리에서 해제하는 시스템이 필요한 것이다.

### GC의 동작

**GC**의 핵심 동작 원리는 **“아직도 참조하고 있는가?”**이다.

#### **Reachability**

객체는 다음과 같은 기준으로 구분된다.

```
GC Root
  ↓
참조된 객체 → 살아있음
참조되지 않음 → GC 대상
```

해당 라인처럼 GC는 존재하고 있는 객체에 대해서 참조 여부를 확인하여 GC 대상에 포함시키고 있다.

조금 더 자세한 동작을 확인하면 아래와 같은 동작 단계를 기준으로 포함한다.

1. **Mark (마킹)**
   살아있는 객체 표시
2. **Sweep (제거)**
   사용되지 않는 객체 제거
3. **Compact (압축)**
   메모리 단편화 제거

```
[객체 존재]
 O O X O X
 ↓
[Mark]
 O O   O
 ↓
[Sweep]
 O O   O
 ↓
[Compact]
 O O O
```

### Stop The World

**Stop The World**는 **GC**가 실행될 때 발생하는 핵심 개념이다.

**GC**가 수행되고 있는 시점에서는 모든 어플리케이션 스레드가 중지되고, 오직 **GC**만 실행한다. 그렇기 때문에 발생되는 문제로 **응답 지연 발생**과 **성능 저하**가 발생된다.

그래서 우리의 목표는 **Stop The World**시간을 최소화 하는것이 중요한 덕목이 된다.

### Generational GC

**JVM**은 객체 생존 특성을 기반으로 Heap 영역을 **Young Generation**과 **Old Generation**으로 구분된다.

- Young Generation
  - 새로 생성된 객체
  - 대부분 금방 사라짐

  -> GC 자주 발생 (Minor GC)

- Old Generation
  - 오래 살아남은 객체
  - 안정적

  -> GC 적게 발생 (Major GC)

### GC의 종류 (간단 언급)

대표 GC 알고리즘:
_ Serial GC
_ Parallel GC \* G1 GC (Java 9+ 기본)

_(**GC**에 대하여 상세한 내용은 따로 작성)_

---

## Native Methode Interface

**JNI(Java Native Interface)**는 Java 코드에서 C/C++과 같은 네이티브 언어로 작성된 코드를 호출할 수 있도록, **JVM**과 **외부 환경을 연결하는 인터페이스**이다.
이를 통해 Java는 운영체제 수준의 기능이나 기존 네이티브 라이브러리를 활용할 수 있다.

### JNI 필요성

**JVM**에 대해서 알아보면 Java는 OS 바로 위에서 동작하지 않고 OS 사이에 JVM이 존재하는 구조를 갖고 있다.
그렇기 떄문에 Java는 **플렛폼의 독립성**을 갖는 이점이 있지만, 시스템 자원 접근에 제약이 존재하게 된다.

예를 들어 OS 시스템 호출과 하드웨어 직접 제어, 고성능 네이티브 라이브러리 활용 등의 작업에 있어 Java만으로는 해결할 수 없다.
이러한 문제를 해결하기 위해 **JNI**이 사용된다.

### JNI 기본 동작 구조

```
Java Code
   ↓
JNI 호출
   ↓
Native Method (C/C++)
   ↓
OS / Hardware
```

### 내부 동작 흐름

```
1. Java에서 native 메서드 호출
2. JVM이 JNI를 통해 Native 함수 탐색
3. Native 라이브러리 로드
4. C/C++ 코드 실행
5. 결과를 Java로 반환
```

### JNI의 구조적 관점

**JNI**는 **Execution Engine**의 내부 컴포넌트라기 보다는 **JVM**과 **외부를 연결하는 계층**이다.

```
[JVM]
 └── Execution Engine
        ↓
      JNI
        ↓
[Native Library (C/C++)]
```

### GC와 연관된 JNI

**JNI**는 **GC**와도 연결되어 있다. **JNI**은 Native Code에서 Java 객체 참조가 가능하다. 그렇기 때문에 **GC Toor**에 포함이 된다.
그렇기 떄문에 잘못 사용하게 된다면 **메모리 누수**문제가 발생할 가능성이 충분히 존재한다.

---

## 마무리 정리

|         **구성 요소**         |               **주요 역할**               |             **핵심 특징**              |
| :---------------------------: | :---------------------------------------: | :------------------------------------: |
|          Interpreter          |  바이트코드를 한 줄씩 해석하여 즉시 실행  | 빠른 시작 속도, 반복 실행 시 성능 저하 |
|         JIT Compiler          |  반복 실행되는 코드를 Native 코드로 변환  |    HotSpot 기반, 런타임 최적화 수행    |
|    Garbage Collector (GC)     | 사용되지 않는 객체를 회수하여 메모리 관리 |  Reachability 기반, 자동 메모리 관리   |
| Native Method Interface (JNI) |     Java와 네이티브 코드(C/C++) 연결      |    OS 및 외부 라이브러리 활용 가능     |

**Execution Engine**은 **JVM**에서 바이트코드를 실제로 실행하는 핵심 계층으로, 단순한 코드 실행을 넘어 성능 최적화와 메모리 관리까지 포함하는 중요한 역할을 수행한다.

**Interpreter**를 통해 즉시 실행을 제공하고, **JIT Compiler**를 통해 반복 코드에 대한 성능을 극대화하며, **Garbage Collector**를 통해 안정적인 메모리 관리를 보장한다. 또한 **JNI**를 통해 **JVM** 외부 환경과의 연계를 가능하게 함으로써, Java의 한계를 보완하고 확장성을 제공한다.

결과적으로 **Execution Engine**은 단순한 실행기가 아니라, **“효율적인 실행을 위한 최적화된 실행 시스템”**이라고 볼 수 있다. 이러한 구조 덕분에 Java는 인터프리터 기반의 언어임에도 불구하고 높은 실행 성능과 안정성을 동시에 확보할 수 있다.
