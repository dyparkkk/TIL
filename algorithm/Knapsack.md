# Knapsack


## DP의 가정

"어떤 문제의 입력사례의 최적해가 그 입력사례를 분할한 부분사례에 대한 **최적해를 항상 포함**하고 있으면, 그 **문제에 대하여 최적의 원리가 성립**한다."

- 집합 A가 n번째 보석을 포함하고 있지 않다면, A는 n번째 보석을 뺀 나머지 n-1개의 보석들 중에서 최적으로 고른 부분집합과 같다.
- 집합 A가 n번째 보석을 포함하고 있다면, A에 속한 보석들의 총 가격은 n-1개의 보석들 중에서 최적으로 고른 가격의 합에다가 보석 n의 가격을 더한 것과 같다. (단, n번째 보석을 넣었을 때 배낭이 터지지 않아야 한다)

## 점화식

![knapsack](https://raw.githubusercontent.com/dyparkkk/TIL/main/algorithm/img/knapsack.png)

## DP[i][j]정의

처음부터 i 까지 물건을 살펴보고 , 배낭 용량이 j 였을 때 배낭에 들어간 가치합의 최대값

→ i번째 물건을 넣을 때와 넣지 않을 때 가치가 큰 것을 고르면 됨!

## DP 2차원 for문 풀이

```cpp
int w[101]; // 무게
int v[101]; // 가치
int dp[101][100001];

int n, k; // 물품의 수 N, 배낭 무게 K

void knapsack(){
    for(int i = 0; i<n; i++){
        for(int j=1; j<=k; j++){
            if(j >= w[i]) // i번째 물건을 넣을 수 있다면 
                dp[i][j] = max(dp[i-1][j], dp[i-1][j-w[i]]+v[i]);
                // 안넣거나 넣거나
						else dp[i][j] = dp[i-1][j];
        }
    }

    cout << dp[n-1][k];
}
```

## DP 1차원 배열로 풀기

```cpp
int w[101]; // 무게
int v[101]; // 가치
int dp[101];

int n, k; // 물품의 수 N, 배낭 무게 K

void knapsack(){
    for (int i = 0; i < n; i++) {
		for (int j = k; j >= 0; j--) {
			if (w[i] <= j) { // 넣을 수 있다면?
				dp[j] = max(dp[j], dp[j - w[i]] + v[i]);
                // 그 물건을 넣었을 때와 넣지 않았을 때 중 더 큰 값으로 초기화
			}
		}
	}
}
```

## 재귀로 풀이

```cpp
int w[101]; // 무게
int v[101]; // 가치
int dp[101][100001];

int n, k; // 물품의 수 N, 배낭 무게 K

int knapsack(int i, int c){
    //i = 물건번호, c = 넣을 수 있는 중량
    if(i == n) return 0;

    int& ret = dp[i][c];
    if(ret != -1) return ret;

    ret = 0;
    if(c >= w[i])
        ret = max(knapsack(i+1, c), knapsack(i+1, c-w[i])+v[i]);
    else ret = knapsack(i+1, c);

    return ret;
}
... 
```

for문으로 구현한 게 더 빠름

---

참고자료:

[https://gsmesie692.tistory.com/113](https://gsmesie692.tistory.com/113)

[https://chanhuiseok.github.io/posts/improve-6/](https://chanhuiseok.github.io/posts/improve-6/)

[https://godls036.tistory.com/5](https://godls036.tistory.com/5)
