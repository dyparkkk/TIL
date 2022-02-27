# Topology Sort

생성일: 2022년 2월 15일 오후 4:09

위상정렬: ‘**순서가 정해져 있는 작업**’을 차례로 실행할 때 사용

- 위상 정렬은 여러 개의 답이 존재할 수 있음
- DAG(Directed Acyclic Graph) : (개별 요소들이 특정한 방향을 향하고 있으며 순환 구조가 없는 그래프) 에서만 사용가능  →  위상정렬의 시작점을 찾을 수 없다
- 

```cpp
#define MAX 10
int n, inDegree[MAX]; // 진입차수 - 조건이라 생각하면 편함 -> 0이 될 경우 진입가능
vector<int> v[MAX]; // v[i][j] - i에서 j로 향함( j는 i를 수행해야 수행 될 수 있음)

void topologySort(){
    int result[MAX]; // 결과값 저장
    queue<int> q;

    //진입차수가 0인 노드를 큐에 삽입
    for(int i=1; i<=n; i++){
        if(inDegree[i]==0) q.push(i);
    }
    // n개를 방문하기 전에 q가 비면 사이클이 발생...
    for(int k=1; k<=n; k++){
        if(q.empty()){
            cout << "싸이클 발생";
            return;
        }

        int x = q.front(); q.pop();
        result[k] = x;
        for(int i=0; i<v[x].size(); i++){
            inDegree[v[x][i]]--;
            if(inDegree==0) q.push(v[x][i]);
        }
    }
    // result
    for(int i=1; i<=n; i++)
        cout << result[i] << " ";
}
```

관련문제 : 

[https://www.acmicpc.net/problem/14567](https://www.acmicpc.net/problem/14567) (선수과목)

참고 자료:

[https://blog.naver.com/ndb796/221236874984](https://blog.naver.com/ndb796/221236874984)