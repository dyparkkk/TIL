# LCS

Longest Common Subsequence : 최장 공통 부분수열

1. 문자열 A와 문자열 B를 한 글자씩 비교함
2. 두 문자가 다르다면 LCS[i-1][j] 와 LCS[i][j-1] 중에 큰값
3. 같다면 LCS[i][j] = LCS[i-1][j-1] +1 

- 부분수열은 연속된 값이 아님 → 이전의 최대 값이 유지됨(2번)
- 문자가 같으면 지금까지 최대 수열 +1

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/algorithm/img/LCS.png)

```cpp
int LCS(){
    string s1, s2;
    int LCS[100][100];

    // 초기화 부분 
    for(int i=0; i<=s1.size(); i++)
        LCS[i][0] = 0;
    for(int i=0; i<=s2.size(); i++)
        LCS[0][i] = 0;
    
    // LCS 길이 구하는 과정
    for(int i=1; i<= s1.size(); i++){
        for(int j=1; j<=s2.size(); j++){
            if(s1[i-1] == s2[j-1])
                LCS[i][j] = LCS[i-1][j-1] + 1;
            else   
                LCS[i][j] = max(LCS[i-1][j], LCS[i][j-1]);
        }
    }

    // LCS 문자열 구하는 과정
    int i = s1.size();
    int j = s2.size();
    string result = "";
    while (LCS[i][j] != 0)
    {
        if(LCS[i][j] == LCS[i][j-1])
            j--;
        else if(LCS[i][j] == LCS[i-1][j])
            i--;
        else if(LCS[i][j] - 1 == LCS[i-1][j-1]){
            result = result + s1[i-1];
            i--;
            j--;
        }
    }
    cout << result;

    return 0;
}
```

## 편집 거리 알고리즘

두 문자열의 유사도를 판단하는 알고리즘 

- 삽입, 삭제, 수정을 몇 번이나 해서 두 문자열을 동일하게 만들 수 있는지 판단
- LCS와 유사한 모양을 가짐

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/algorithm/img/levenshtein_distance.png)

1. 문자가 같으면 dp[i][j] = dp[i-1][j-1]
2. 다르면 dp[i][j] = min( dp[i][j-1], min( dp[i-1][j], dp[i-1][j-1]))   + 1
    - 순서대로 추가, 삭제, 복사
    

```cpp
int main(){
    string s1, s2;
    int dp[100][100];

    // 초기화 부분 
    for(int i=0; i<=s1.size(); i++)
        dp[i][0] = 0;
    for(int i=0; i<=s2.size(); i++)
        dp[0][i] = 0;
    
    // LCS 길이 구하는 과정
    for(int i=1; i<= s1.size(); i++){
        for(int j=1; j<=s2.size(); j++){
            if(s1[i-1] == s2[j-1])
                dp[i][j] = dp[i-1][j-1];
            else   
                dp[i][j] = min(dp[i][j-1], min(dp[i-1][j], dp[i-1][j-1])) + 1;
        }
    }

		cout << dp[s1.size()][s2.size()];
}
```

---

참고자료 :

[https://velog.io/@emplam27/알고리즘-그림으로-알아보는-LCS-알고리즘-Longest-Common-Substring와-Longest-Common-Subsequence](https://velog.io/@emplam27/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-LCS-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-Longest-Common-Substring%EC%99%80-Longest-Common-Subsequence)

[https://hsp1116.tistory.com/41](https://hsp1116.tistory.com/41)