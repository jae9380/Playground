# **[Algorithm] Manacher’s Algorithm - 회문 찾기**    


# Palindrome, 회문
> ### **회문 or Palindrome 이란? **  <br>
> 회문 또는 팰린드롬이라 부르는 것은 특정 단어를 거꾸로 읽어도 원래의 문자와 동일한 단어를 부른다.   <br>
> 간단한 예를 들면 오디오, 기러기, bob, level, abbcdcbba 와 같이 거꾸로 읽어도 동일한 것을 나타낸다.

회문은 앞에서 거꾸로 읽어도 원래의 단어와 동일한 단어라고 설명한다. 여기서 중요한 부분이 어느 지점을 중심으로 뒤집어야 원래의 단어와 동일할까? 이 부분이다.     

회문에서 중심이 되는 축은 회문의 중간 지점이 될 것이다.     
* 문자열 str의 길이가 짝수일 경우  ( str의 길이 = len )
  * len / 2 와 len / 2 + 1 사이 지점
* 문자열 str의 길이가 홀수일 경우
  * ( len + 1 ) / 2 지점

---

# Manacher’s Algorithm    

회문의 성질에 대해서 간단하게 이해하고 중심 내용인 회문을 어떻게 효율적으로 찾을 수 있는지 알아보자.    

Manacher’s 알고리즘은 회문을 찾는 방법 중 자주 사용되는 알고리즘이라고 한다. 해당 알고리즘은 어떤 방법으로 회문을 찾고, 회문을 찾을 때 왜 자주 사용되는지 Banana를 예를 들어서 알고리즘 동작 과정을 알아보자.    

**일단 Manacher’s 알고리즘의 동작 과정을 간단하게 정리를 하면 아래의 과정과 같다.**

1. 문자열 i번을 중심 축으로 가정한다.
2. i를 기준으로 회문의 최대 반지름을 기록한다.
   * 여기서 반지름이란 중심축으로 가장 멀리 있는 문자 까지의 길이
위 과정을 통해서 주어지는 문자열 내 길이가 긴 회문을 찾을 수 있게 된다.

그럼 직접 동작 과정을 확인하자.

| Index      | b         | a         | n         | a         | n         | a         |
|:----------:|:---------:|:---------:|:---------:|:---------:|:---------:|:---------:|
| str [ i ]  | str [ 0 ] | str [ 1 ] | str [ 2 ] | str [ 3 ] | str [ 4 ] | str [ 5 ] |
| Palindrome | b         | a         | ana       | anana     | ana       | a         |
| Radius     | 0         | 0         | 1         | 2         | 1         | 0         |

위 표는 문자열 str에서 i번의 위치를 중심 축으로 가정하여 어떤 회문이 성립이 되고, 반지름의 길이는 얼마정도 되는지 기록한 표이다.    

* i = 0 , str [ 0 ]은 “b”만이 회문이 된다. 그리고 반지름의 길이는 0 이다.
* i = 1 , “a”는 앞과 뒤의 문자가 다르기 때문에 혼자만 회문이며, 반지름이 0이다.
* i = 2 , 해당 위치에서는 회문 “ana”가 성립이 된다. 반지름은 1이다.
* …

해당 과정에서 반지름을 구하는 방법은 이와 같이 정의할 수 있다. 

* str [ i ] 를 기준으로 앞과 뒤의 문자가 동일할 경우 범위를 하나씩 넓혀서 계속 비교를 하고 반지름을 +1 해준다.
  * str [ 1 - ( r [ i ] + 1 )], str [ 1 + ( r [ i ] + 1 )] 을 비교 ( r [ i ] -> i인덱스를 기준으로 회문의 반지름 길이 )
* 앞과 뒤의 문자열이 동일하지 않거나, 문자열의 길이 범위를 초과하는 경우 반복동작을 멈춘다.

일단은 위 내용을 바탕으로 코드를 간단하게 작성을 했다.  
**해당 코드는 Manacher’s Algorithm을 구현한 코드가 아니다. 추가적으로 완벽한 코드가 아니다. <br>  설명을 위해 작성한 간단한 코드이다.**
```java
public static String findPalindrome(String s) {
    int n = s.length();
    if (n == 0) return "회문 없음";

    int[] radius = new int[n];
    String result = "";
    String[] str = s.split("");
    int max_radius_index = 0, max_radius = 0;
    for (int i = 0; i < n; i++) {
        radius[i] = 0;
        int l = i - (radius[i] + 1), r = i + (radius[i] + 1);
        while (l >= 0 && r < n && str[l].equals(str[r])) {
            radius[i] = radius[i] + 1;
            l = i - (radius[i] + 1);
            r = i + (radius[i] + 1);
        }
        if (max_radius < radius[i]) {
            max_radius = radius[i];
            max_radius_index = i;
        }
    }

    result = s.substring(max_radius_index - max_radius, max_radius_index + max_radius + 1);

    return result.isEmpty() ? "회문 없음" : result;
}

```

설명과 간단한 동작 과정을 바탕으로 코드를 작성했다.     

## 문제 발생 : 회문이 짝수일 경우 체크를 못 한다.

위 코드에서 “banana”를 입력값으로 주었을 때 가장 긴 회문인 “anana”를 잘 반환을 해주지만, “banaana”를 입력값으로 줄 때 반환값으로 “anaana”를 줘야 하지만 “ana”를 반환한다는 문제점이 발생한다.    

앞에서 회문에 대한 성질을 이야기 할 때 문자열의 길이가 짝수일 경우와 홀수일 경우를 나눠서 중심축이 문자가 되거나 문자 사이가 된다고 말을 했었다.    

하지만 작성된 코드에서는 이를 고려하지 않고 주어지는 문자열을 바로 배열로 만들어서 탐색을 하기 때문에 중심축이 문자와 문자 사이가 될 경우를 고려하지 않았기 때문에 발생하는 문제이다.    

### 문제 해결 방법

그럼 문자와 문자 사이가 회문의 중심축이 되기 위해서 문자와 문자 사이 공간에 동일한 특수문자를 추가하여 탐색을 해야한다.

```java
StringBuilder t = new StringBuilder("#");
for (char c : s.toCharArray()) {
    t.append(c).append('#');
}
char[] arr = t.toString().toCharArray();
```

주어지는 문자열을 배열로 바로 만드는 방법이 아닌 문자와 문자 사이 동일한 특수문자를 사용하여 문자와 문자 사이가 회문의 중심축이 되도록 만들어 준다.    

그러면 기존 코드에서는 “aa”, “anaana”와 같이 회문의 길이가 짝수일 경우를 정확하게 체크할 수 있다.   

## 문제 발생 : 무식한 방법으로 탐색을 하고 있다.

작성된 코드에서는 직접 좌우의 범위를 늘리면서 좌우 문자에 대하여 동일 여부를 확인하고 있다.    

```java
for (int i = 0; i < n; i++) {
    // 좌우 확장해서 회문인지 확인
    while (l >= 0 && r < n && s.charAt(l) == s.charAt(r)) {
        ...
    }
}
```

이러한 방법으로 탐색을 한다면 여러 문제점에 노출이 된다.  

1. 모든 위치를 기준으로 일일이 확장을 한다.  -> **O(n²)** 시간 소요
2. 이전에 탐색한 정보를 낭비한다. -> 이미 회문인 구간을 다시 검사
3. 회문 길이에 대한 분기점 필요

지금은 반복문을 통해서 각 인덱스 i를 중심으로 회문 가능 여부를 범위를 넓히면서 검증을 하고 있다. 이러한 방법은 주어지는 문자열의 길이가 길어지면 상당히 비효율적으로 탐색하게 된다.    

추가적으로 이전에 회문 여부를 검증한 정보를 사용하지 않고 무식하게 또 검증을 한다는 것이다.    

문자열 “abacaba”가 입력값이라 가정을 하면, i = 1일 때 검증한 회문은 “aba”일 것이다. 다음으로 i = 3일 경우의 회문은 “abacaba”가 된다.     

**하지만** i = 5일 경우 “aba”라는 회문이 된다는 점을 또 검증을 하고 있다는 것이다.

마지막으로 회문이 짝수일 경우와 홀수일 경우에 대한 분기점이 없다는 것이다.

### 문제 해결 방안 

```java
int l = i - (radius[i] + 1), r = i + (radius[i] + 1);
while (l >= 0 && r < n && str[l].equals(str[r])) {
    radius[i]++;
    l = i - (radius[i] + 1);
    r = i + (radius[i] + 1);
}
```

기존 방법에서는 변수 l과 r을 이용하여 중심축으로 확장을 하여 문자를 비교를 하며, 기존에 탐색되었던 정보를 이용하지 않고 모든 위치에 대해서 회문을 검증하고 있다는 점에서 불필요한 계산과 검증을 하고 있었다는 것이다. 

```java
int center = 0, right = 0;  // 중심과 우측 경계
int maxLen = 0, centerIndex = 0;  // 최대 길이 회문 및 인덱스

for (int i = 1; i < arr.length - 1; i++) {
    // [2-1] 대칭 위치 활용 (이미 찾은 회문 정보로 계산)
    int mirror = 2 * center - i;  // 대칭되는 i 위치

    if (i < right) {
        radius[i] = Math.min(right - i, radius[mirror]);  // 대칭되는 구간에서 가능한 반지름 갱신
    }

    // [2-2] 중심 확장 (현재 위치에서 회문 확장)
    while (arr[i + (radius[i] + 1)] == arr[i - (radius[i] + 1)]) {
        radius[i]++;  // 회문 확장
    }

    // [2-3] 경계 갱신 (새로운 회문이 오른쪽 경계를 넘어갈 경우)
    if (i + radius[i] > right) {
        center = i;  // 새로운 중심
        right = i + radius[i];  // 새로운 우측 경계
    }

    // [2-4] 최대 회문 길이 갱신
    if (radius[i] > maxLen) {
        maxLen = radius[i];
        centerIndex = i;
    }
}
```

회문 탐색에 있어 무작정 중심 확장을 통해 검증하는 방식이 아니라, 회문의 특성인 대칭을 활용하여 이미 검사한 회문 정보를 재활용함으로써 중복 연산을 줄이고 시간 복잡도를 **O(n)**으로 최적화할 수 있다. 이러한 방식은 특히 문자열의 길이가 긴 경우, 기존 방식에 비해 현저한 성능 차이를 보인다.

---
## Manacher 알고리즘이 추구하는 방향

**이전에 계산한 회문 정보를 재활용하면서, 최소한의 확장만 수행하는 최적화 중심 확장 알고리즘**

### 핵심 아이디어
	* 	문자열을 전처리해서 **모든 회문을 홀수 길이 회문처럼 보이게 만든다.** → 일관된 처리 가능
	* 	회문 중심 center, 오른쪽 끝 경계 right를 유지하면서,
	* 	**현재 인덱스 i가 right보다 작으면**, 그 대칭점 mirror의 값을 **재활용**해서 회문 길이를 미리 예측
	* 	예측한 회문 반지름이 부족할 때만 **실제로 확장**

### 회문 찾기에 있어 중심 확장법은?
물론 회문을 찾는 과정에서 Manacher’s Algorithm을 이용을 한다면 효율적으로 해답을 얻을 수 있겠지만, 구현에 있어 조금 복잡할 수 있다. 오히려 중심 확장법을 이용한 구현이 좀 더 쉽다고 느껴진다. 
(초반에 작성한 코드가 중심 확장법을 이용 했다.)

중심 확장법이 회문을 찾는 방법에 있어 좋지 않은 방법이라고는 확정할 수 없다. 중심 확장 방법은 구현이 간단하고 직관적이라는 장점이 있어, 입력 문자열의 길이가 짧거나 성능이 크게 중요하지 않은 경우에는 충분히 유용한 접근 방식이 될 수 있다.

---
## 최종 코드

```java
public static String findPalindrome(String s) {
    if (s == null || s.isEmpty()) return "회문 없음";

    StringBuilder t = new StringBuilder("#");
    for (char c : s.toCharArray()) {
        t.append(c).append('#');
    }

    char[] arr = t.toString().toCharArray();
    int[] radius = new int[arr.length];
    int center = 0, right = 0;  // 중심과 우측 경계
    int maxLen = 0, centerIndex = 0;  // 최대 길이 회문 및 인덱스

    for (int i = 1; i < arr.length - 1; i++) {
        // [2-1] 대칭 위치 활용 (이미 찾은 회문 정보로 계산)
        int mirror = 2 * center - i;  // 대칭되는 i 위치

        if (i < right) {
            radius[i] = Math.min(right - i, radius[mirror]);  // 대칭되는 구간에서 가능한 반지름 갱신
        }

        // [2-2] 중심 확장 (현재 위치에서 회문 확장)
        while (arr[i + (radius[i] + 1)] == arr[i - (radius[i] + 1)]) {
            radius[i]++;  // 회문 확장
        }

        // [2-3] 경계 갱신 (새로운 회문이 오른쪽 경계를 넘어갈 경우)
        if (i + radius[i] > right) {
            center = i;  // 새로운 중심
            right = i + radius[i];  // 새로운 우측 경계
        }

        // [2-4] 최대 회문 길이 갱신
        if (radius[i] > maxLen) {
            maxLen = radius[i];
            centerIndex = i;
        }
    }

    int start = (centerIndex - maxLen) / 2;
    return s.substring(start, start + maxLen);
}
```