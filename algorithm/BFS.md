# BFS

그래프를 전체 탐색하기 위해 너비를 우선으로 탐색하는 알고리즘

- 구현을 빠르게 할 수 있어야 함

### 기본 bfs 틀

```cpp
#include<bits/stdc++.h>
using namespace std;
#define fir first
#define sec second
int board[502][502];
int check[502][502];
int dx[]={1, 0, -1, 0};
int dy[]={0, 1, 0, -1};
int n = 7, m = 10; // 크기

// 기본 bfs
int bfs(){
  queue<pair<int,int>> q;
  check[0][0]=1; // 주의
  q.push({0,0});
  while(!q.empty()){
    pair<int,int> cur = q.front(); q.pop();
    for(int dir=0; dir<4; dir++){
      int ny = cur.fir + dy[dir];
      int nx = cur.sec + dx[dir];

      if(nx < 0 || nx >= m || ny < 0 || ny >= n) continue; // 먼저 체크
      if(check[ny][nx] || board[ny][nx] != 1) continue;
      check[ny][nx] = 1; // 넣을 때 체크해야함
      q.push({ny, nx});
    }
  }
}
```

### 거리 측정

- 거리 측정시 dist 초기화에 주의하자

백준 미로탐색(2178)

```cpp
// https://www.acmicpc.net/problem/2178
#include <bits/stdc++.h>
using namespace std;
int n, m;
string board[102];
int dist[102][102];
int dy[]={0, 1, 0, -1};
int dx[]={1, 0, -1, 0};

void bfs(){
  queue<pair<int,int>> q;
  dist[0][0] = 0;
  q.push({0,0});
  while(!q.empty()){
    auto cur = q.front(); q.pop();
    for(int dir=0; dir<4; dir++){
      int ny = cur.first+dy[dir];
      int nx = cur.second+dx[dir];
      if(ny < 0 || ny >= n || nx <0 || nx >= m) continue;
      if(board[ny][nx] != '1' || dist[ny][nx] != -1) continue;
      dist[ny][nx] = dist[cur.first][cur.second] +1;
      q.push({ny, nx});
    }
  }
}

int main(){
  cin >> n >> m;
  for(int i=0; i<n; i++)
    fill(dist[i], dist[i]+m, -1); // 거리 측정의 경우 -1로 check 겸 거리 측정해야함 

  for(int i=0; i<n; i++)
    cin >> board[i];

  bfs();
  cout << dist[n-1][m-1]+1;
  return 0;
}
```

### 시작점이 두 종류 일때

- 한쪽만 영향을 받는 경우
- 두개 짜고 순서대로 돌리면 됨

백준 불!-4179번

```cpp
// https://www.acmicpc.net/problem/4179
#include <bits/stdc++.h>
using namespace std;
int n, m;
string board[1002];
int dist[1002][1002];
int dist2[1002][1002];
int dx[]={1, 0, -1, 0};
int dy[]={0, 1, 0, -1};

// 
int bfsJ( queue<pair<int,int>> &q){
  while(!q.empty()){
    auto cur = q.front(); q.pop();
    for(int dir=0; dir<4; dir++){
      int ny = cur.first + dy[dir];
      int nx = cur.second + dx[dir];
      // 경계로 도착하면 탈출
      if(ny < 0 || ny >= n || nx < 0 || nx >= m) 
        return dist2[cur.first][cur.second] + 1;
      if(board[ny][nx] == '#' ||  dist2[ny][nx] > 0 ) continue; // 벽 & 이미 왔던곳
      if(dist[ny][nx] != -1 && dist[ny][nx] <= dist2[cur.first][cur.second]+1) continue; // 불 조건 추가
      dist2[ny][nx] = dist2[cur.first][cur.second] +1;
      q.push({ny, nx});
    }
  }
  return -1;
}

// 불이 먼저 퍼짐
void bfsF(queue<pair<int,int>> &q){
 
  while(!q.empty()){
    auto cur = q.front(); q.pop();
    for(int dir = 0; dir<4; dir++){
      int ny = cur.first+dy[dir];
      int nx = cur.second+dx[dir];

      if(ny < 0 || ny >= n || nx < 0 || nx >= m) continue;
      if(board[ny][nx] == '#' || dist[ny][nx] != -1) continue;
      dist[ny][nx] = dist[cur.first][cur.second]+1;
      q.push({ny, nx});
    }
  }
}

int main(){
  ios_base::sync_with_stdio(false); cin.tie(NULL);
  cin >> n >> m;
  for(int i=0; i<n; i++){
    fill(dist[i], dist[i]+m, -1);
    fill(dist2[i], dist2[i]+m, -1);
  }
  queue<pair<int,int>> q1, q2;
  for(int i=0; i<n; i++)
    cin >> board[i];
  
  for(int i=0; i<n; i++){
    for(int j=0; j<m; j++){
      if(board[i][j] == 'F'){
        dist[i][j] = 0;
        q1.push({i, j});
      }
      if(board[i][j] == 'J'){
        dist2[i][j] = 0;
        q2.push({i, j});
      }
    }
  }

  // bfs 불 
  bfsF(q1);
  // bfs j
  int ans = bfsJ(q2);
  if(ans == -1) cout << "IMPOSSIBLE";
  else cout << ans;
  return 0;
}
```

### 응용 - 일차원B BFS

- 한 칸 이동 뿐만 아니라 *2 등의 이동도 처리할 수 있음

백준 숨바꼭질(1697)

```cpp
// https://www.acmicpc.net/problem/1697
#include <bits/stdc++.h>
using namespace std;
int dist[200001];
int main(){
  int n, k; cin >> n >> k;
  fill(dist, dist+200001, -1);
  queue<int> q;
  dist[n] = 0;
  q.push(n);
  while(dist[k] == -1){
    int cur = q.front(); q.pop();
    for(int nx : {cur+1, cur-1, cur*2}){ // +1 or -1 or *2
      if(nx < 0 || nx >= 100001) continue; // *2 후에 -1보다 -1하고 *2 하는게 더 빠름
      if(dist[nx] != -1) continue;
      dist[nx] = dist[cur] +1;
      q.push(nx);
    }
  }
  cout << dist[k];
  return 0;
}
```

---
참고자료 :  
[https://blog.encrypted.gg/941?category=773649](https://blog.encrypted.gg/941?category=773649)