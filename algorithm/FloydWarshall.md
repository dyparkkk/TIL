## 플로이드 와샬 알고리즘

**모든 정점에서 모든 정점으로 최단거리 구하는 알고리즘** 

- **거쳐가는 정점**을 기준으로 알고리즘 수행
- dp 사용

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

int n = 4;
int INF = 1000000000;
int arr[4][4] = {
    {0, 5, INF, 8},
    {7, 0, 9, INF},
    {2, INF, 0, 4},
    {INF, INF, 3, 0}
};

int main(){
    // Floyd-warshall : k-거쳐가는 노드
    for(int k=0; k<n; k++){
        //i-출발 노드
        for(int i=0; i<n; i++){
            //j-도착 노드
            for(int j=0; j<n; j++){
                arr[i][j] = min(arr[i][j], arr[i][k]+arr[k][j]);
            }
        }
    }

    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++)
            cout << arr[i][j] << "   ";
        cout <<"\n";
    }
    
    return 0;
}
```

- INF+INF 연산이 있으니 오버플로우를 조심하자 ! ( 초기화 1,000,000,000 하는게 좋을듯)

참고자료 : 
[안경잡이개발자 : 네이버 블로그](https://blog.naver.com/ndb796/221234427842)
