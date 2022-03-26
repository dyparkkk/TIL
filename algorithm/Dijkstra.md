# Dijkstra

한 정점에서 다른 모든 정점까지의 최소 거리를 구하는 알고리즘 

- 한 정점에서 다른 한 정점으로 최소 거리를 구할 때도 사용됨
- 가중치에 음수가 있다면 쓸 수 없음
- 힙을 사용해서 구현함 → 이럴 경우에 O(E logE)의 시간복잡도를 가짐
- 구현이 까다로우니 코드를 가져다 쓰는 것을 추천함

백준 - **최소비용 구하기 2** 

```cpp
// https://www.acmicpc.net/problem/11779
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second

vector<pair<int, int>> adj[1005]; // 비용, 정점 번호
int d[1005]; //최단 거리 테이블
int pre[1005]; // 경로 복원 테이블
const int INF = 0x3f3f3f3f;

int main(){
  ios_base::sync_with_stdio(false); cin.tie(nullptr);
  int n, e; cin >> n >> e;
  for(int i=0; i<e; i++){
    int u, v, w; cin >> u >> v >> w;
    adj[u].push_back({w, v});
  }
  fill(d, d+n+1, INF);

  int st, en; cin >> st >> en;

  priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq; // 그리디 방식이므로 힙 사용
  d[st] = 0;
  pq.push({d[st], st});
  while(!pq.empty()){
    int cost = pq.top().first;
    int node = pq.top().second; pq.pop(); // 비용, 정점번호
    if(d[node] != cost) continue; // 사용 정상 값인지 확인
    for(auto next : adj[node]){
      int nCost = next.first;
      int nn = next.second;
      if(d[nn] <= d[node] + nCost) continue;
      //현재 기록된 최단거리보다 기존거리+새로운 edge가 짧다면 최단거리 배열 갱신 해줌 
      d[nn] = d[node] + nCost;
      pq.push({d[nn], nn});
      pre[nn] = node; // 경로복원용
    }
  }
  cout << d[en] << '\n';

  // 경로 추적 구현
  vector<int> path;
  int cur = en;
  while(cur != st){
    path.push_back(cur);
    cur = pre[cur];
  }
  path.push_back(cur);
  reverse(path.begin(), path.end());
  cout << path.size() << '\n';
  for(auto x : path)
    cout << x << ' ';
  return 0;
}
```

---

참고자료 : 

[https://blog.encrypted.gg/1037?category=773649](https://blog.encrypted.gg/1037?category=773649)