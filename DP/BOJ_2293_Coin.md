# BOJ 2293 동전1

## Fact

1. n가지 종류의 동전이 있다.
2. 각 동전을 적절히 사용해서 그 가치의 합이 k원 이 되도록 할 수 있는 경우의 수를 구해야 한다.

* 조건
  1. 각각의 동전은 몇 개라도 사용할 수 있다.
  2. 사용한 동전의 구성이 같고 순서가 다른 경우는 하나의 경우로 취급한다.
* 입력
  * 첫째 줄에 n과 k가 주어진다. (1<=n<=100, 1<=k<=10,000)
  * 다음 n개의 줄에는 각각 동전의 가치가 주어진다. (0 < 동전의 가치 <= 100,000)
* 출력
  * 첫째 줄에 경우의 수를 출력한다. (경우의 수 <= 2^31)

## Overview

가치가 다른 n가지의 동전을 조합하여 k원을 만드는 경우의 수를 구하는 문제

* 같은 구성, 다른 순서인 조합이 여러개 있을 경우, 하나로 본다.

## Algorithm

#### Try

1. `dp[n] = dp[n-1] + dp[dp[n-1]+coin[n]] ` 실패

   * [BOJ 2293의 예제 입력](https://www.acmicpc.net/problem/2293)과 같이 k = 10 인 경우, 

     `1 + 9 ,  2 + 8 , 5 + 5`의 3가지로 나누어 풀면 된다고 생각했으나 

   * 중복을 처리하지 못한다.

2. check 라는 변수 하나를 더 추가해 봤지만 실패

   * 중복된 값을 찾아주기 위해 변수 하나를 더 도입해 사용한 동전의 갯수가 같은지 check해줄 생각이었으나
   * 그냥 int형 변수 하나만으로 처리하기엔 생각보다 복잡했다.

3. 2D array를 선언해 값을 조정해주려 하였으나 실패

   * `dp[i][j] : i번쨰 동전으로 j원을 만들어주자! `라는 생각이었으나
   * 100 * 10000의 array를 만들어야 해서 mem 용량 초가가 된다.
   * 그리고 생각해보니 i번째 동전만 사용한다고 되는게 아니라, i번째까지 사용하여 만들어야 했다.

4. 왜 틀렸는지 아직도 모르겠는 source code    **해결!!**

```c++
dp[0] = 1;								//coin[i](코인의 가치) = k원 인 경우
for(int i = 1; i <= n; i++)
	for(int j = coin[i]; j <= k; j++)	//안쪽 for문을 cost[i]로 초기화
		dp[j] += dp[j -coin[i]];

cout << dp[k];
```

* 처음에는 1,2,5를 입력했을 때 답이 나와서 감격스러웠는데, BOJ에서 계속 `틀렸습니다`가 떠서 곰곰히 생각해보니, 

* 만약, 첫번째로 입력 된 cost의 값이 나중에 입력된 cost의 값보다 더 크다면, 계산이 누락된다.

  입력된 동전의 가치가 ` 2, 5, 1` 이라면  `1`일 때의 값이 **누락**되고,

  입력된 동전의 가치가  `5, 2, 1` 이라면  `1, 2`일 때의 값이 **누락**될 수밖에 없었다. 

* 그래서 slt의 sort를 이용해 정렬을 다시 해준 후, 돌려도 봤지만 결과는 `틀렸습니다`였다.

  * for문 앞에 `sort(coin, coin+n);` 추가

##### dp를 선언할 때 array를 전부 0으로 초기화 해주니 `맞았습니다!` 를 만났다!!

##### 결국 sort는 필요가 없었다.

#### 접근 방법(Try 4)

* 모든 동전의 경우의 수를 구하자!
  | coin\k |   1   |         2         |       3       |           4            |              5              |
  | :----: | :---: | :---------------: | :-----------: | :--------------------: | :-------------------------: |
  |  1원   | 1가지 |       1가지       |     1가지     |         1가지          |            1가지            |
  |  2원   | 1가지 | **2가지**(11, 20) | 2가지(111,21) | **3가지**(1111,211,22) |    3가지(11111,2111,221)    |
  |  5원   | 1가지 |       2가지       |     2가지     |         3가지          | **4가지**(11111,2111,221,5) |

  | coin\k |         6          |         7          |        8         |     9     |         10         |
  | ------ | :----------------: | :----------------: | :--------------: | :-------: | :----------------: |
  | 1원    |       1가지        |       1가지        |      1가지       |   1가지   |       1가지        |
  | 2원    | **4가지**   (+222) |       4가지        | **5가지**(+2222) |   5가지   | **6가지**(+22222)  |
  | 5원    | **5가지**   (+51)  | **6가지**(+511,52) |    **7가지**     | **8가지** | **10가지**   (+55) |

  * 점화식을 얻기 위해 먼저 표에서 규칙을 찾아본다.

    1. 1원일 때는 가지수가 늘 1가지이다.
    2. 1원과 2원을 사용할 수 있을때, 2원 단위로 가지수가 1가지씩 늘어난다.
       * 가치가 2인 동전을 새롭게 사용할 수 있게 된 것이다.
    3. 1,2,5원을 사용할 수 있을 때, 4원을 만들기 위해서는 5원짜리 동전이 필요하지 않지만, 5원 이상의 k값을 만들 때는 5원을 사용해서 새로운 경우의 수를 만들 수 있으므로 1가지씩 늘어난다.
       * 가치가 5인 동전을 새롭게 사용할 수 있게 된 것이다.

  * 최종적으로 k원을 만들기 위해,

    1. 처음 들어오는 coin의 가치로 만들 수 있는 경우의 수를 구한다.
    2. 그 다음 들어오는 coin의 가치로 만들 수 있는 경우의 수를 계속해서 더한다.
       * 이때, dp 개념을 사용하여 이전에 들어온 coin으로 k원을 만들 수 있는 경우의 수를 memoization해준다면, 문제를 해결할 수 있다.

  * 따라서, **점화식**은

    * `dp[j] += dp[j-coin[i]]` 이다.

    조금 더 이해하기 쉽게 설명하자면,

    * coin[i]번째부터 n가지의 동전을 전부 사용할 때까지 i를 +1씩 증가시키면서,

      `for(i = 0 ; i < n ; i++)`

      * 현재까지의 가치의 합 j가 최종적으로 k원이 될 때까지 j를 +1씩 증가시키고

        `for(j = coin[i] ; j <= k ; j++)` : 처음 가치의 합 j 는 coin[i] 이다.

        * 현재까지의 가치를 구할 수 있는 경우의 수 dp[j]에, 

          현재까지의 가치의 합(j) - 새롭게 들어온 coin[i]의 가치를 뺀 경우의 수를 

          계속해서 누적해 준다.

          `dp[j] = dp[ j - coin[i] ]`


### Time Complexity

O(n^2)

### Space Complexity

O(n)