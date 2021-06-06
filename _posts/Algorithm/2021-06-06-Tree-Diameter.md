---
layout: post
title:  "[PS]트리의 지름"
date:   2021-06-06 20:19:35 +0900
categories: [Algorithm]
---
[트리의 지름](https://www.acmicpc.net/problem/1967) 문제입니다. 삽질을 좀 해서 기록으로 남깁니다.

## 삽질 포인트
- 트리가 이진트리라는 말이 없었음에도 이진트리로 가정하고 풀다가 두 번이나 틀린 뒤 잘못된 걸 알았습니다.
- 자식 노드들 중 가장 깊은 두 개를 갱신하는 부분을 잘못 짜서 또 한번 틀렸습니다. 

## 소스코드

```cpp
#include <iostream>
#include <vector>
using namespace std;

typedef struct _node {
  vector<struct _node *> children;
  vector<int> weights;
  int maxDepth = 0;
  int maxWidth = 0;
} Node;

int getMaxDepth(Node *node) {
  if (node == nullptr) return 0;
  if (node->maxDepth != 0) return node->maxDepth;
  for (int i = 0; i < node->children.size(); i++) {
    node->maxDepth =
        max(node->maxDepth, node->weights[i] + getMaxDepth(node->children[i]));
  }
  return node->maxDepth;
}

int getMaxWidth(Node *node) {
    /* children 중 가장 depth 큰 것 두개, 혹은 children 중 가장 width 큰 것. */
  if (node == nullptr) return 0;
  int widthByDepth = 0;
  int deepest1 = 0, deepest2 = 0;
  for (int i = 0; i < node->children.size(); i++) {
    int tempw = getMaxDepth(node->children[i]) + node->weights[i];
    if (deepest1 < tempw) {
      deepest2 = deepest1;
      deepest1 = tempw;
    } else if (deepest2 < tempw) {
      deepest2 = tempw;
    }
  }
  widthByDepth = deepest1 + deepest2;

  for (int i = 0; i < node->children.size(); i++) {
    node->maxWidth = max(node->maxWidth, getMaxWidth(node->children[i]));
  }
  node->maxWidth = max(node->maxWidth, widthByDepth);
  return node->maxWidth;
}

int main(void) {
  ios::sync_with_stdio(false);
  cin.tie(nullptr);
  int n, p, c, w;

  cin >> n;
  vector<Node> treeVec(n + 1);
  for (int i = 0; i < n - 1; i++) {
    cin >> p >> c >> w;
    treeVec[p].children.push_back(&treeVec[c]);
    treeVec[p].weights.push_back(w);
  }

  cout << getMaxWidth(&(treeVec[1])) << endl;

  return 0;
}
```