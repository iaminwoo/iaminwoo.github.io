+++
date = '2026-04-04T10:23:51+09:00'
draft = true
title = 'Tree 구조들'
summary = '요약'
tags = ['자료구조', 'Java']
+++

## 1) 빠른 조회와 수정을 위한 이진 탐색 트리 (BST) 의 탄생

### 1. 선형적 자료구조의 어려움

일반적인 **배열과 리스트**는 '**선형적**' 이다.

저장된 데이터 중 내가 **원하는 데이터를 찾기 위해서**는 **전부를 다 찾아봐야** 하는 것이다.
많은 데이터가 쌓이면, 그 중에서 필요한 것을 찾는 것은 어려워진다.

```
0. b
1. c
2. a
3. e
```

### 2. 정렬된 배열의 이진 탐색

**더 효율적인 조회**를 위해서는 데이터를 정리하는 것이 필요하다.
그래서 배열을 특정 기준으로 **정렬**하면, **이진 탐색으로 빠른 조회**가 가능하다.

사람들을 **키 순으로** 자리에 앉게 하면, 키가 165cm 인 사람을 찾는 건 간단해진다.
중간에 서있는 사람의 키가 165cm 보다 **작으면 뒤의** 사람들에서,
**크면 앞의** 사람들에서 찾으면 된다.

```
0. 160
1. 165
2. 170
3. 175
4. 180
```

이렇게 정렬된 배열을 이진탐색하면, 기존의 **O(n) 에서 O(log n) 의 향상된 조회 성능**을 보여준다.

하지만 **새로운 사람을 추가**하려면?
해당 사람의 위치 **뒤의 모든 사람을 이동시키는 시프팅 비용이 발생**하게 된다.

### 3. 연결 리스트 방식

그러면 다른 방법을 사용해보자.
고정된 자리를 사용하지 않고 **사람들이 각자 손을 잡게 하는 연결 리스트의 방식**이다.
이 경우 **새로운 사람이 추가**되면 단순히 **기존 손을 놓고 새로운 사람과 손**을 잡으면 된다.

```
170 - 180 - 165 - 160 - 175
```

하지만 키가 165cm 인 사람을 찾으려면? **다시 줄의 앞부터 뒤까지를 모두 확인**해야 한다.

### 4. 이진 탐색 트리 (BST) 의 등장

> 이진 탐색의 효율성과 연결 리스트의 유연함을 더한 이진 탐색 트리

**이진 탐색 트리** 는 이 **두가지 방법의 타협안**으로 등장하였다.
이번에는 손을 잡는데, **나보다 키가 작은 사람은 왼쪽, 키가 큰 사람은 오른쪽에 잡는다.**

그러면 정렬된 배열의 **이진 탐색**처럼,
루트에 있는 사람의 키가 165cm 보다 작다면 오른쪽, 크다면 왼쪽에서 찾아보면 된다.

```
     175
    /  \
  165   180
  /  \
160   170
```

그리고 **새로운 사람을 추가**할 때는 루트부터 시작해서 빈 곳으로 이동해 손을 잡으면 되고,
**기존의 사람을 삭제**할 때는 빈자리를 찾을 적절한 사람을 찾아 손을 놓으면 된다.

이렇게 트리 구조, 특히 BST 는 **데이터의 위계**를 사용하여
**빠른 조회와 수정이 가능한 효율적인 자료구조**를 만들고자 하였다.

---

## 2) 이진 탐색 트리의 구현

이진 탐색 트리는 **삽입과 조회** 등을 **재귀 혹은 반복문**을 사용하여 구현한다.

**삭제 로직**만 주의하면 되는데,
삭제하려는 노드가 **자녀가 없거나 한쪽만 있는 경우**는
해당 노드를 삭제하고 부모 노드와 자녀 노드를 연결하면 되지만,

**자녀가 양쪽 다 있는 경우**는 양쪽 자녀를 유지하고
주로 오른쪽 서브트리의 최소 노드와 삭제하려는 노드를 바꾸는 작업을 진행한다.

```java
public class BST {
    private static class Node {
        int value;
        Node left;
        Node right;
        Node parent;

        private Node(int value) {
            this.value = value;
        }
    }

    private Node root;

    public void insert(int value) {
        Node newNode = new Node(value);

        if (root == null) {
            root = newNode;
            return;
        }

        insertNode(root, newNode);
    }
    private void insertNode(Node parent, Node newNode) {
        if (newNode.value >= parent.value) {
            // right
            if (parent.right == null) {
                parent.right = newNode;
                newNode.parent = parent;
            } else {
                insertNode(parent.right, newNode);
            }
        } else {
            // left
            if (parent.left == null) {
                parent.left = newNode;
                newNode.parent = parent;
            } else {
                insertNode(parent.left, newNode);
            }
        }
    }

    public void inorder() {
        if (root == null) return;
        StringBuilder sb = new StringBuilder();
        inorderNode(root, sb);
        sb.setLength(sb.length() - 1);
        System.out.println(sb);
    }
    private void inorderNode(Node node, StringBuilder sb) {
        if (node == null) return;
        inorderNode(node.left, sb);
        sb.append(node.value).append(" ");
        inorderNode(node.right, sb);
    }

    public boolean search(int searchKey) {
        Node found = searchNode(root, searchKey);
        return found != null;
    }
    private Node searchNode(Node node, int searchKey) {
        if (node == null) return null;

        if (node.value == searchKey) return node;

        if (searchKey >= node.value) {
            return searchNode(node.right, searchKey);
        } else {
            return searchNode(node.left, searchKey);
        }
    }

    public void delete(int searchKey) {
        Node toDelete = searchNode(root, searchKey);
        if (toDelete == null) return;

        Node left = toDelete.left;
        Node right = toDelete.right;

        if (left == null && right == null) {
            deleteFromParent(toDelete, null);
            return;
        }

        if (left != null && right == null) {
            // 왼쪽만
            deleteFromParent(toDelete, left);
            return;
        }

        if (left == null) {
            // 오른쪽만
            deleteFromParent(toDelete, right);
            return;
        }

        // 둘다
        Node rightMin = getRightMin(right);
        toDelete.value = rightMin.value;
        deleteFromParent(rightMin, rightMin.right);
    }

    private Node getRightMin(Node node) {
        while (node.left != null) node = node.left;
        return node;
    }

    private void deleteFromParent(Node node, Node child) {
        Node parent = node.parent;
        if (parent != null) {
            if (parent.left == node) {
                parent.left = child;
            } else {
                parent.right = child;
            }
        } else {
            root = child;
        }
        if (child != null) child.parent = parent;
    }
}
```

---

## 2) 이진 탐색 트리의 함정 : 편향

하지만 이진 탐색 트리에는 **치명적인 문제**가 있는데,
바로 **정렬된 순서로 요소가 추가**되는 경우이다.

위의 예시에서 새로 추가되는 사람들이 기존의 사람들보다 모두 키가 큰 사람들이라면?
루트에서 시작해서 **모든 사람들이 다 오른손만 잡고 있는 구조**로
사실상 연결 리스트와 다를 것이 없어지게 되어 **O(n) 으로 조회 성능이 떨어진다.**

```
160
  \
  165
    \
    170
      \
      175
        \
        180
```

이렇게 **조회 성능**이 고작 추가되는 **데이터의 성질에 따라 변동**된다면,
이런 자료 구조를 실제로 믿고 사용할 수 있을까?
O(log n) 과 O(n) 을 오가는 도박적인 성능 말고 **구조적으로 성능을 유지**할 수는 없을까?

---

## 3) 자가 균형 매커니즘 : 엄격한 AVL 트리

이런 이진 탐색 트리의 불안정성을 해결하는 것이 자가 균형 매커니즘이다.

그 중 AVL 트리는,

## 4) 자가 균형 매커니즘 : 실용적인 Red-Black 트리

AVl 트리는 너무 엄격

## 5) TreeMap 자료구조

이런 Red-Black 트리를 활용하는 TreeMap

---

1. 자가 균형 매커니즘: AVL 트리, 완벽주의자의 설계
   AVL 트리는 모든 노드에서 왼쪽과 오른쪽 서브트리의 높이 차이를 1 이하로 유지하도록 강제하는 '결벽증'에 가까운 엄격함을 보여줍니다.

물리적 공정으로서의 회전(Rotation): AVL 트리의 핵심은 단순히 값을 옮기는 것이 아니라, 치우친 무게 중심을 반대편으로 끌어올려 전체 트리의 높이를 물리적으로 깎아내는 과정에 있습니다. 이는 마치 나뭇가지가 한쪽으로 휘어지려 할 때 중심축을 비틀어 전체적인 수평을 맞추는 정교한 엔지니어링과 같습니다.

장점: 매우 엄격하게 균형을 강제하기 때문에, 데이터의 수정보다 조회(Search) 성능이 극한으로 중요할 때 압도적인 효율을 보여줍니다.

4. 자가 균형 매커니즘: Red-Black 트리, 실용주의의 선택
   실무, 특히 자바의 TreeMap이나 HashMap 내부에서 선택한 것은 AVL이 아닌 **Red-Black 트리(RBT)**입니다. 여기에는 철저히 계산된 실용주의가 숨어 있습니다.

느슨함의 미학: RBT는 높이 차이를 1 이하로 맞추려 애쓰지 않습니다. 대신 색상 규칙(Red, Black)을 통해 **"최장 경로가 최단 경로의 2배를 넘지 않는다"**는 비교적 느슨한 가이드라인을 지킵니다.

실용적 관점의 효율성: AVL은 완벽한 균형을 위해 삽입/삭제 시마다 너무 잦은 회전 연산을 수행해야 합니다. 반면 RBT는 균형을 조금 양보하는 대신, 삽입/삭제 시 발생하는 회전 횟수를 통계적으로 줄였습니다. **"조금 더 자주 찾을 것인가(AVL), 아니면 더 효율적으로 데이터를 넣고 뺄 것인가(RBT)"**라는 질문에 현대 컴퓨팅은 RBT의 손을 들어준 것입니다.

5. TreeMap과 HashMap: 방어적 설계의 정점
   자바 컬렉션 프레임워크는 이러한 RBT의 특성을 극한까지 활용합니다.

TreeMap: RBT를 직접 구현하여 데이터를 저장함과 동시에 항상 정렬 상태를 유지합니다. 덕분에 특정 범위의 데이터를 찾거나(Range Search), 최솟값/최댓값을 즉시 뽑아내는 데 탁월합니다.

HashMap의 진화: HashMap은 기본적으로 해시 충돌 시 연결 리스트를 사용하지만, 특정 버킷에 요소가 8개(Java 8 기준)를 넘어가면 해당 리스트를 **TreeNode(RBT)**로 변환(Treeify)합니다.

인사이트: Hash DOS 공격 방어
이는 단순히 성능 최적화를 넘어선 방어적 설계입니다. 악의적으로 해시 충돌을 유도하여 시스템 성능을 $O(n)$으로 떨어뜨리려는 공격(Hash DOS)이 들어와도, 트리가 성능 하한선을 $O(\log n)$으로 보장해주기 때문입니다. 결국 자료구조는 효율성을 넘어 시스템의 보안과 안정성을 책임지는 든든한 방패가 됩니다.
