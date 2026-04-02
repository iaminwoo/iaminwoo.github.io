+++
date = '2026-04-01T09:14:33+09:00'
draft = true
title = '자바 핵심 자료구조 draft'
summary = '자바의 대표적인 자료구조 3종의 내부 동작 원리를 분석, 최악 및 평균 시간 복잡도와 효율적인 구현 전략을 정리'
tags = ['자료구조', 'Java']
+++

자료구조와 메서드마다 시간복잡도를 가지고 있다.
이 시간복잡도는 어떻게 나온 것인지 실제 구현을 통해 알아보자.

---

## 2. HashMap : 해싱과 충돌 관리

HashTable 역시 배열을 활용하는 자료구조이다.
**Key 와 Value 를 저장**하는데, **Key 를 인덱스로** 바꾸고 **해당 위치에 Value 를 저장**하는 구조이다.

> Key 를 인덱스로 바꾸는 Hashing!

먼저 자바의 모든 객체는 **HashCode** 를 가진다.

```java
String hw = "Hello World!";
System.out.println(hw.hashCode());
/* 결과: -969099747 */
```

이 해시코드는 **객체의 인식번호**로 당연히 **중복될 수도 있다.**
반환값은 Int 로 나오는데 이것을 배열의 인덱스로 바꾸게 된다.

> HashCode -> 배열의 인덱스

핵심 아이디어는 반환된 **HashCode 값을 배열의 길이로 나눈 나머지**를 사용하는 것이다.
그러면 자연스럽게 **배열의 길이에 맞는 인덱스들**이 나오게 된다.

> HashCode % 배열의 길이 = 인덱스

하지만,
이것에는 두가지 문제가 있는데
첫번째는 **HashCode 의 값은 음수**가 나올 수 있다는 점이고,
hashing 작업은 자주 수행되는데 **나머지 계산은 무거운 작업**이라는 것이다.

> AND 연산 사용!

이 두가지를 **동시에 해결하는 방법**은 **비트 연산**을 사용하는 것이다.
AND 연산을 사용하는데, 계산을 쉽게 하기 위해서
**일반적으로 배열의 길이는 2의 제곱수**로 정하며
**배열의 길이 -1 로 계산**할 것이다.

```
// AND 계산 수행
11000110 00111100 10110110 00011101 (-969099747)
00000000 00000000 00000000 00001111 (배열의 길이 16 - 1)
----------------------------------------
00000000 00000000 00000000 00001101 (13)
```

**결과값이 index 의 값**이 되고,
HashCode 값의 부호와 상관없이 **하위 비트 4개만 사용**되고 있는 것을 볼 수 있다.

> **Hashing 으로 나오는 index 의 값**은 HashCode 이진수의 **하위 비트 자리들!**

두번째로 실제로 **나머지 연산과 비트 연산의 속도 차이**를 비교해보았다.

```java
// 1. 테스트용 해시코드 10만 개를 미리 준비 (객체 생성 비용 제거)
int[] hashCodes = new int[100000];
for (int i = 0; i < hashCodes.length; i++) {
  hashCodes[i] = new Object().hashCode();
}

int arrSize = 16;
int mask = arrSize - 1;
long sum = 0; // 결과값을 저장해서 JVM 최적화 방지

// --- 실험 1: 비트 연산 (&) ---
long start = System.nanoTime();
for (int h : hashCodes) {
  sum += (h & mask); // 순수 비트 연산만 수행
}
long end = System.nanoTime();
System.out.println("비트 연산 (&) 순수 시간: " + (end - start) / 1_000_000.0 + " ms");
// ---------

sum = 0; // 초기화

// --- 실험 2: 나머지 연산 (%) ---
start = System.nanoTime();
for (int h : hashCodes) {
  sum += (Math.abs(h) % arrSize); // 순수 나머지 연산만 수행
}
end = System.nanoTime();
System.out.println("나머지 연산 (%) 순수 시간: " + (end - start) / 1_000_000.0 + " ms");
// ---------

// --- 출력 결과 ---
/* 비트 연산 (&) 순수 시간: 0.215875 ms */
/* 나머지 연산 (%) 순수 시간: 2.001166 ms */
```

**비트 연산이 나머지 연산보다 10배 빠른 것**을 확인할 수 있다.

그런데,
여기서 새로운 문제가 발생한다.
**만약 HashCode 의 값이 이상적이지 않고 하위 비트가 겹치는 값들이라면?**
(메모리 위치 등의 특정 값들이거나 의도적으로 충돌을 유발하려는 경우)

그래서 **상위 비트들**이 하위비트들에 **영향을 미칠 수 있도록**
= 모든 비트들이 영향을 주도록 32비트 HashCode 값을 반으로 접어
**상위, 하위 16비트를 XOR 연산**을 하는데 이를 **보조 해시 함수**라고 한다.

참고로 AND, OR 연산은 0, 1 둘중에 하나가 결과에 큰 영향을 미치는 반면
**XOR 연산**은 **양쪽의 값들이 모두 동일한 영향**을 준다.

```java
HashCode ^ (HashCode <<< 16)
```

위의 값을 예시로 들면

```
// XOR 계산 수행
11000110 00111100 10110110 00011101 (-969099747)
00000000 00000000 11000110 00111100 (상위 비트 이동)
----------------------------------------
11000110 00111100 01110000 00100001
```

이 경우 하위 비트 4자리를 사용하기 때문에 새로운 인덱스는 1이 된다.

HashCode 가 다양하게 나오는 이상적인 경우에는 결과에 큰 영향은 없지만
**XOR 연산 자체의 비용은 거의 없기 때문에 안전을 위해 사용한다.**

> Hashing 로직 끝!

그러면 본격적으로 HashMap 을 구현해보자.

(는 내일!)

put 에서 충돌 처리하는 것 -> 해당 인덱스 요소 뒤에 LinkedList 로
get 은 인덱스 찾아서 LinkedList 돌면서 Key 값 같은 요소 찾기
resize 는 배열의 크기 2배로

```java
public class PHashMap<K, V> {
    private static class Node<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;

        private Node(K k, V v, int h) {
            key = k;
            value = v;
            hash = h;
        }
    }

    private Node<K, V>[] table = (Node<K, V>[]) new Node[16];
    private int size = 0;
    private final float loadFactor = 0.75f;

    public void put(K key, V value) {
        Node<K, V> newNode = new Node<>(key, value, key.hashCode());
        int index = getIndex(key);

        if (size >= table.length * loadFactor) resize();

        if (table[index] == null) {
            table[index] = newNode;
        } else {
            Node<K, V> cur = table[index];
            while (true) {
                if (cur.key.equals(key)) {
                    cur.value = value;
                    return;
                }

                if (cur.next == null) {
                    cur.next = newNode;
                    break;
                }
                cur = cur.next;
            }
        }
        size++;
    }

    private int getIndex(Object key) {
        if (key == null) return 0;
        int hashcode = key.hashCode();
        hashcode = hashcode ^ (hashcode >>> 16);
        return hashcode & (table.length - 1);
    }

    public V get(K key) {
        Node<K, V> cur = table[getIndex(key)];
        while (!cur.key.equals(key)) {
            cur = cur.next;
        }
        return cur.value;
    }

    private void resize() {
        int newCap = table.length << 1;
        Node<K, V>[] newTable = (Node<K, V>[]) new Node[newCap];

        for (Node<K, V> node : table) {
            Node<K, V> cur = node;

            while (cur != null) {
                Node<K, V> next = cur.next;
                int newIdx = cur.hash & (newCap - 1);

                cur.next = newTable[newIdx];
                newTable[newIdx] = cur;

                cur = next;
            }
        }

        table = newTable;
    }
}
```

---

## 3. PriorityQueue : 우선순위 큐 / 힙

이진 트리 구조를 배열로 관리하는 법

enqueue(E element) (heapify-up)
dequeue() (heapify-down)
peek()

Complete Binary Tree의 배열 표현: 인덱스 계산 방식(parent = (i-1)/2, left = 2i+1 등)을 직접 도출해 보세요.

최소 힙 vs 최대 힙: 비교 연산자(Comparator)를 통해 힙의 성질을 자유롭게 바꿀 수 있도록 추상화해 보세요.

- 원소가 추가/삭제될 때 부모-자식 노드 간의 위치를 바꾸는 Sift-up/Sift-down 로직을 재귀나 반복문으로 구현했는가?
