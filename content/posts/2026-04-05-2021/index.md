+++
date = '2026-04-05T20:21:49+09:00'
draft = false
title = '이진 탐색 트리 직접 구현 과정'
summary = '이진 탐색 트리의 기본 원리 이해, 재귀와 반복문'
tags = ['과정', '자료구조', 'Java']
+++

궁극적으로는 인덱스에 사용되는 B tree 계열이나
TreeMap 등에서 사용되는 Red-Black Tree 구현을 준비하며
**기본적인 이진 탐색 트리**를 먼저 구현해보았다.

## 1) BST 구현 1

처음에 트리에 있는 **노드라는 것을 특정하는 것은 값**이고
이 값은 **불변해야 자료구조가 안전**해진다고 생각해서
value 필드를 **final 로 구현**했다.

그런데 **삭제 부분**을 구현하면서,
노드를 **물리적으로 끊고 옮기는 부분**이 **구현하기 까다로웠다.**

그래서 찾아보니,
일반적으로는 구현의 편의를 위해 값의 수정을 허용한다고 한다.

각자의 장단점이 있지만,
지금은 **자료구조 자체의 원리를 학습**하는 단계이므로
value 필드에는 **final 을 붙이지 않았다.**

```java
// 트리 노드 객체
private static class Node {
    int value;    // final 제거!
    Node left;    // 왼쪽 자녀 포인터
    Node right;   // 오른쪽 자녀 포인터
    Node parent;  // 노드의 부모 포인터

    // 생성자 생략...
}
```

### 1-1) 요소 추가

```java
public void insert(int value) {
    Node newNode = new Node(value);

    // 루트 노드 삽입 처리
    if (root == null) {
      root = newNode;
      return;
    }

    // 재귀로 처리
    insertNode(root, newNode);
}

private void insertNode(Node parent, Node newNode) {
  if (newNode.value >= parent.value) {
    // 부모의 값보다 크거나 같으면 오른쪽에 삽입
    if (parent.right == null) {
      // 빈 경우 바로 삽입
      parent.right = newNode;
      newNode.parent = parent;
    } else {
      // 이미 존재하면 재귀로 처리
      insertNode(parent.right, newNode);
    }
  } else {
    // 부모의 값보다 작으면 왼쪽에 삽입
    if (parent.left == null) {
      // 빈 경우 바로 삽입
      parent.left = newNode;
      newNode.parent = parent;
    } else {
      // 이미 존재하면 재귀로 처리
      insertNode(parent.left, newNode);
    }
  }
}
```

### 1-2) 요소 조회

```java
// 해당 값을 가진 노드가 존재하는지
public boolean search(int searchKey) {
  Node found = searchNode(root, searchKey);
  return found != null;
}

private Node searchNode(Node node, int searchKey) {
  if (node == null) return null;

  if (node.value == searchKey) return node;

  // 값 비교를 통해 왼쪽 혹은 오른쪽 서브트리 조회
  if (searchKey >= node.value) {
    return searchNode(node.right, searchKey);
  } else {
    return searchNode(node.left, searchKey);
  }
}
```

### 1-3) 요소 삭제

```java
public void delete(int searchKey) {
  // 삭제해야 할 노드 찾기
  Node toDelete = searchNode(root, searchKey);
  if (toDelete == null) return;

  Node left = toDelete.left;
  Node right = toDelete.right;

  // 양쪽 자녀가 없는 리프노드의 경우 그냥 제거
  if (left == null && right == null) {
    deleteFromParent(toDelete, null);
    return;
  }

  // 왼쪽 자녀만 있는 경우 왼쪽 자녀를 기존 노드 위치로
  if (left != null && right == null) {
    deleteFromParent(toDelete, left);
    return;
  }

  // 오른쪽 자녀만 있는 경우 오른쪽 자녀를 기존 노드 위치로
  if (left == null) {
    deleteFromParent(toDelete, right);
    return;
  }

  // 자녀가 둘다 있는 경우 오른쪽 서브트리의 최소 노드를 기존 노드 위치로
  Node rightMin = getRightMin(right);
  toDelete.value = rightMin.value;
  deleteFromParent(rightMin, rightMin.right);
}

private Node getRightMin(Node node) {
  while (node.left != null) node = node.left;  // 왼쪽 노드만 따라가며 확인
  return node;
}

private void deleteFromParent(Node node, Node child) {
  Node parent = node.parent;
  // 부모 노드에서 연결 변경
  if (parent != null) {
    if (parent.left == node) {
      parent.left = child;
    } else {
      parent.right = child;
    }
  } else {
    root = child;
  }

  // 자녀 노드에서도 연결 변경
  if (child != null) child.parent = parent;
}
```

---

## 2) BST 구현 2

이렇게 완성하고 보니 아쉬운 점이 몇개 있었다.

간단하게 먼저는, **해당 값을 가진 노드가 있는지** 찾는 메서드의 이름은
search 보다 **contains 가 더 일반적**인 것 같고,

자녀가 둘다 있는 노드의 **삭제 시 사용하는 대체 노드**의 이름이
rightMin 보다는 **successor 가 더 일반적이고 명확**한 것 같다는 생각이 들었다.

두번째로는 **부모 포인터의 필요성**이었다.

지금은 **노드에서 상위 노드를 바로 찾거나 이동**하는 경우가 없고,
주요 연산들은 **재귀를 더 잘 사용**하면 **굳이 부모 포인터가 없어도** 구현할 수 있었다.

트리의 가장 큰 특성 중 하나는
어느 노드를 루트로 하는 **서브트리를 봐도 독립적인 트리**를 이루는 것인데
이게 **더 잘 드러나게 하는 것이 재귀**를 사용하는 것 같아서
**부모 포인터 없이** 다시 구현하였다.

```java
private static class Node {
    int value;
    Node left;
    Node right;
    // 부모 포인터 제거!

    // 생성자 생략...
}
```

### 2-1) 요소 추가

```java
// 재귀의 반환값을 사용해 더 간단하게 구현
public void insert(int value) {
  // 삽입되는 새로운 노드부터 루트까지의 과정을 반환값으로 연결
  root = insertNode(root, value);
}

private Node insertNode(Node node, int value) {
  // 빈 자리를 발견하면 새 노드를 연결하고 반환
  if (node == null) return new Node(value);

  // 노드의 값에 따라 반환된 노드들이 자연스럽게 부모 노드에 연결됨
  if (node.value < value) {
    node.right = insertNode(node.right, value);
  } else if (node.value > value) {
    node.left = insertNode(node.left, value);
  }
  return node;
}
```

### 2-2) 요소 조회

```java
// 메서드 이름 contains 로 변경
public boolean contains(int searchKey) {
  Node node = root;

  // 단순히 트리를 따라 이동하기 때문에 메서드 오버헤드 없이 반복문으로 구현
  while (node != null) {
    if (node.value == searchKey) return true;
    node = (node.value < searchKey) ? node.right : node.left;
  }

  return false;
}
```

### 2-3) 요소 삭제

```java
public void delete(int searchKey) {
  // 삽입과 마찬가지로 재귀의 반환값들이 삭제되는 노드부터 루트까지 자연스럽게 연결
  root = deleteNode(root, searchKey);
}

private Node deleteNode(Node node, int value) {
  // 값에 해당하는 노드가 없다면 null 반환
  if (node == null) return null;

  // 노드의 값과 비교하며 노드 찾기
  if (node.value < value) {
    node.right = deleteNode(node.right, value);
  } else if (node.value > value) {
    node.left = deleteNode(node.left, value);
  } else {
    // 값에 해당하는 노드를 찾음

    // 자녀 노드가 없거나 하나만 있는 경우
    // 둘다 없으면 자연스럽게 null 이 반환
    if (node.right == null) return node.left;
    if (node.left == null) return node.right;


    // 자녀 노드가 둘다 있는 경우
    Node successor = getRightMin(node.right);  // 오른쪽 서브트리 최소 노드가 대체 노드가 됨
    node.value = successor.value;
    node.right = deleteNode(node.right, successor.value);  // 대체 노드의 값으로 해당 노드 제거
  }
  return node;
}

private Node getRightMin(Node node) {
  while (node.left != null) node = node.left;
  return node;
}
```

확실히 **부모 포인터가 없어지니까**
삽입과 삭제 과정에서 **포인터 작업이 줄어 더 단순**해졌다.

그리고 **반복문과 재귀의 차이**를 고민하게 되었는데
일반적으로 같은 상황에서 **둘다 사용**이 가능하다.

하지만 **단순히 이동을 위한 목적**이라면
**반복문을 사용**해서 메서드 호출 오버헤드를 줄이고,

다음 단계로 **갔다가 다시 돌아와야** 하거나
그 **반환값이 필요한 경우**라면 **재귀를 사용**하는 것이 더 적합하다.

하지만 **이진 탐색 트리**는 요소가 **정렬되어 삽입될 때 위험**한데,
이때는 삽입되는 요소가 바로 **트리의 높이**가 되어
트리의 높이가 **깊어짐에 따라**
**StackOverflowError 가 발생**할 수 있으니 주의해야 한다.

---

이제 기본적인 이진 탐색 트리를 구현했으니
정렬 삽입의 비효율을 해결하는
AVL과 Red-Black Tree!
그리고 인덱스에 사용하는 B Tree 까지 갑니다!

---

## 3) 전체 코드

{{< tabs >}}

    {{< tab label="BST 구현 1" >}}
    ```java
    public class BST {
      // 트리 노드 객체
      private static class Node {
          int value;    // final 제거
          Node left;    // 왼쪽 자녀 포인터
          Node right;   // 오른쪽 자녀 포인터
          Node parent;  // 노드의 부모 포인터

          private Node(int value) {
            this.value = value;
          }
      }

      private Node root;

      public void insert(int value) {
          Node newNode = new Node(value);

          // 루트 노드 삽입 처리
          if (root == null) {
            root = newNode;
            return;
          }

          // 재귀로 처리
          insertNode(root, newNode);
      }

      private void insertNode(Node parent, Node newNode) {
        if (newNode.value >= parent.value) {
          // 부모의 값보다 크거나 같으면 오른쪽에 삽입
          if (parent.right == null) {
            // 빈 경우 바로 삽입
            parent.right = newNode;
            newNode.parent = parent;
          } else {
            // 이미 존재하면 재귀로 처리
            insertNode(parent.right, newNode);
          }
        } else {
          // 부모의 값보다 작으면 왼쪽에 삽입
          if (parent.left == null) {
            // 빈 경우 바로 삽입
            parent.left = newNode;
            newNode.parent = parent;
          } else {
            // 이미 존재하면 재귀로 처리
            insertNode(parent.left, newNode);
          }
        }
      }


      public void inorder() {
        if (root == null) return;
        StringBuilder sb = new StringBuilder();
        // 중위 순회 출력
        inorderNode(root, sb);

        sb.setLength(sb.length() - 1);
        System.out.println(sb);
      }

      private void inorderNode(Node node, StringBuilder sb) {
        if (node == null) return;
        inorderNode(node.left, sb);         // 왼쪽 서브트리 추가
        sb.append(node.value).append(" ");  // 본인 추가
        inorderNode(node.right, sb);        // 오른쪽 서브트리 추가
      }


      // 해당 값을 가진 노드가 존재하는지
      public boolean search(int searchKey) {
        Node found = searchNode(root, searchKey);
        return found != null;
      }

      private Node searchNode(Node node, int searchKey) {
        if (node == null) return null;

        if (node.value == searchKey) return node;

        // 값 비교를 통해 왼쪽 혹은 오른쪽 서브트리 조회
        if (searchKey >= node.value) {
          return searchNode(node.right, searchKey);
        } else {
          return searchNode(node.left, searchKey);
        }
      }


      public void delete(int searchKey) {
        // 삭제해야 할 노드 찾기
        Node toDelete = searchNode(root, searchKey);
        if (toDelete == null) return;

        Node left = toDelete.left;
        Node right = toDelete.right;

        // 양쪽 자녀가 없는 리프노드의 경우 그냥 제거
        if (left == null && right == null) {
          deleteFromParent(toDelete, null);
          return;
        }

        // 왼쪽 자녀만 있는 경우 왼쪽 자녀를 기존 노드 위치로
        if (left != null && right == null) {
          deleteFromParent(toDelete, left);
          return;
        }

        // 오른쪽 자녀만 있는 경우 오른쪽 자녀를 기존 노드 위치로
        if (left == null) {
          deleteFromParent(toDelete, right);
          return;
        }

        // 자녀가 둘다 있는 경우 오른쪽 서브트리의 최소 노드를 기존 노드 위치로
        Node rightMin = getRightMin(right);
        toDelete.value = rightMin.value;
        deleteFromParent(rightMin, rightMin.right);
      }

      private Node getRightMin(Node node) {
        while (node.left != null) node = node.left;  // 왼쪽 노드만 따라가며 확인
        return node;
      }

      private void deleteFromParent(Node node, Node child) {
        Node parent = node.parent;
        // 부모 노드에서 연결 변경
        if (parent != null) {
          if (parent.left == node) {
            parent.left = child;
          } else {
            parent.right = child;
          }
        } else {
          root = child;
        }

        // 자녀 노드에서도 연결 변경
        if (child != null) child.parent = parent;
      }
    }
    ```
    {{< /tab >}}

    {{< tab label="BST 구현 2" >}}
    ```java
    public class BST2 {
      private static class Node {
        int value;
        Node left;
        Node right;
        // 부모 포인터 제거!

        private Node(int value) {
          this.value = value;
        }
      }

      private Node root;

      // 재귀의 반환값을 사용해 더 간단하게 구현
      public void insert(int value) {
        // 삽입되는 새로운 노드부터 루트까지의 과정을 반환값으로 연결
        root = insertNode(root, value);
      }

      private Node insertNode(Node node, int value) {
        // 빈 자리를 발견하면 새 노드를 연결하고 반환
        if (node == null) return new Node(value);

        // 노드의 값에 따라 반환된 노드들이 자연스럽게 부모 노드에 연결됨
        if (node.value < value) {
          node.right = insertNode(node.right, value);
        } else if (node.value > value) {
          node.left = insertNode(node.left, value);
        }
        return node;
      }

      // 중위 순회 로직 생략...

      // 메서드 이름 contains 로 변경
      public boolean contains(int searchKey) {
        Node node = root;

        // 단순히 트리를 따라 이동하기 때문에 메서드 오버헤드 없이 반복문으로 구현
        while (node != null) {
          if (node.value == searchKey) return true;
          node = (node.value < searchKey) ? node.right : node.left;
        }

        return false;
      }


      public void delete(int searchKey) {
        // 삽입과 마찬가지로 재귀의 반환값들이 삭제되는 노드부터 루트까지 자연스럽게 연결
        root = deleteNode(root, searchKey);
      }

      private Node deleteNode(Node node, int value) {
        // 값에 해당하는 노드가 없다면 null 반환
        if (node == null) return null;

        // 노드의 값과 비교하며 노드 찾기
        if (node.value < value) {
          node.right = deleteNode(node.right, value);
        } else if (node.value > value) {
          node.left = deleteNode(node.left, value);
        } else {
          // 값에 해당하는 노드를 찾음

          // 자녀 노드가 없거나 하나만 있는 경우
          // 둘다 없으면 자연스럽게 null 이 반환
          if (node.right == null) return node.left;
          if (node.left == null) return node.right;


          // 자녀 노드가 둘다 있는 경우
          Node successor = getRightMin(node.right);  // 오른쪽 서브트리 최소 노드가 대체 노드가 됨
          node.value = successor.value;
          node.right = deleteNode(node.right, successor.value);  // 대체 노드의 값으로 해당 노드 제거
        }
        return node;
      }

      private Node getRightMin(Node node) {
        while (node.left != null) node = node.left;
        return node;
      }
    }
    ```
    {{< /tab >}}

{{< /tabs >}}
