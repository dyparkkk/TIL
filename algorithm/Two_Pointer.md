# Two Pointer

배열에서 원래 이중 for문으로 O(N^2)에 처리되는 작업을 2개의 포인터 알고리즘으로 O(N)에 처리하는 알고리즘 

- 이중 포문의 i와 j를 O(N)으로 움직임
- 보통 같은 문제를 이분탐색으로 풀 수 있는 경우가 많다. 다만 이분탐색은 O(NlogN)

백준 2230 - 수 구하기

```cpp
int ans = 2e9+1;
int mn = 0x7fffffff;

int main(){
  ios_base::sync_with_stdio(false); cin.tie(NULL);
  int n, m; cin >> n >> m;
  vector<int> v(n);
  for(auto &i:v)
    cin >> i;
  sort(v.begin(), v.end());

// 투포인터 사용 부분
  int en = 0;
  for(int st=0; st<n; st++){
    while(en < n && v[en] - v[st] < m) en++;
    if(en == n) break;
    ans = min(ans, v[en] - v[st]);
  }
  cout << ans<< "\n";

  return 0;
}
```

백준 1806 : 부분합

```cpp
int ans = 0x7fffffff;
int a[100001];

int main(){
  ios_base::sync_with_stdio(false); cin.tie(NULL);
  int n, m; cin >> n >> m;
  for(int i=0; i<n; i++)
    cin >> a[i];
  
  int sum=a[0], en = 0;
  for(int st=0; st<n; st++){
    while(en < n && sum < m){
      en++;
      sum += a[en];
    }
    if(en == n) break;
    ans = min(ans, en-st+1);
    sum -= a[st];
  }
  
  if(ans == 0x7fffffff) ans = 0;
  cout << ans;
  return 0;

}
```

응용  
start → | ← end 로 양끝에서 모이는 투포인터  
백준 : 두 용액  
```java
// https://www.acmicpc.net/problem/2470
#include <bits/stdc++.h>
using namespace std;

int n;
int gap = 2e9 + 1;
int main(){
  cin >> n;
  vector<int> v(n);
  for(int i=0; i<n; i++){
    cin >> v[i];
  }
    
  sort(v.begin(), v.end());
  // -6, -5, -2, 1, 2, 3

  int st = 0, en = v.size()-1;
  // cout << gap << '\n';
  vector<int> ans(2, 0);
  while(st < en){
    int tmp = v[st] + v[en];
    if(gap > abs(tmp)){
      ans[0] = v[st];
      ans[1] = v[en];
      gap = abs(tmp);
    }

    if(tmp < 0) st++;
    else en--;
  }
  cout << ans[0] << " " << ans[1] << '\n';
  return 0;
}
```

---

참고자료 : 

[https://blog.encrypted.gg/1004?category=773649](https://blog.encrypted.gg/1004?category=773649)