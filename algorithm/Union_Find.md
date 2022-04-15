# Union Find

- 여러 개의 노드에서 서로 다른 노드가 연결되어 있는지 확인하는 알고리즘
- parent( root )를 작은 노드로 정함
    - 1과 2 연결 시에 parent는 1
- parent를 확인하는 방법은 재귀로 구현

```cpp
#include<bits/stdc++.h>
using namespace std;

int parent[10];

// 부모를 찾음 (찾으면서 갱신도 함)
int getParent(int x){
  if(parent[x] == x) return x;
  return parent[x] = getParent(parent[x]);
}

// 부모 노드를 합침
void unionParent(int a, int b){
  a = getParent(a);
  b = getParent(b);
  // 작은 쪽을 부모로 맞춤
  if(a < b) parent[b] = a; 
  else parent[a] = a; 
}

// 같은 부모 노드를 가지는지 확인
bool isEqualParent(int a, int b){
  a = getParent(a);
  b = getParent(b);
  if( a == b ) return true;
  else return false;
}

int main(){
  for(int i=0; i<10; i++){
    parent[i] = i;
  }

  unionParent(1, 2);
  unionParent(2, 3);
  unionParent(3, 4);
  unionParent(5, 6);
  unionParent(6, 7);
  unionParent(7, 8);

  cout << "1과 5 연결 ?? " << isEqualParent(1, 5) << "\n";
  unionParent(1, 6);
  cout << "1과 5 연결 ?? " << isEqualParent(1, 5) << "\n";
  return 0;
}

// 1과 5 연결 ?? 0
// 1과 5 연결 ?? 1
```

---

참고 자료 :   

[https://blog.naver.com/ndb796/221230967614](https://blog.naver.com/ndb796/221230967614)