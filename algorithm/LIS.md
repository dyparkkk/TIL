## Longest Increasing Subsequence 최장 증가 부분 수열

배열에서 각 원소가 이전 원소보다 크고 그 길이가 최대인 수열

어떤 원소 k를 기준으로 앞에 있는 원소가 자신보다 작은 증가 부분 수열을 이룰경우 +1로

dp를 이용해서 품

가장 간단한 방법 O(n^2)

```cpp
for(int i=0; i<n; i++){
	dp[i]=1;
	for(int j=0; j<i; j++){
		if(a[i]>a[j])
			dp[i] = max(dp[i], dp[j]+1);
	}
	ans = max(ans, dp[i]);
}
```

- dp문제는 문제를 해결하는 순서가 중요함 → 효율성이 달라짐
- j 부분을 이분탐색으로 logN으로 줄일 수 있음
- ...
