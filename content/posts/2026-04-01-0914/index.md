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

---

## 3. PriorityQueue : 우선순위 큐 / 힙

이진 트리 구조를 배열로 관리하는 법

enqueue(E element) (heapify-up)
dequeue() (heapify-down)
peek()

Complete Binary Tree의 배열 표현: 인덱스 계산 방식(parent = (i-1)/2, left = 2i+1 등)을 직접 도출해 보세요.

최소 힙 vs 최대 힙: 비교 연산자(Comparator)를 통해 힙의 성질을 자유롭게 바꿀 수 있도록 추상화해 보세요.

- 원소가 추가/삭제될 때 부모-자식 노드 간의 위치를 바꾸는 Sift-up/Sift-down 로직을 재귀나 반복문으로 구현했는가?
