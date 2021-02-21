---
layout: post
title:  "Dynamic Programming"
date:   2021-02-17 20:19:35 +0900
categories: [Algorithm]
---

백준으로 DP(Dynamic Programming, 동적계획법)를 연습해봤습니다.  
DP란 어떤 문제를 부분 문제로 쪼갤 수 있고, 부분 문제의 최적해의 합이 전체 문제의 최적해가 되는 경우 사용할 수 있는 방법입니다. 


# [백준 10942 - 팰린드롬?](https://www.acmicpc.net/problem/10942)

- 팰린드롬이란 거꾸로 읽어도 그대로인 'abcba'나 '다시합창합시다' 같은 문자열입니다. 
- 주어진 문자열의 S번째 문자부터 E번째 문자까지 떼어낸 부분 문자열이 팰린드롬인지를 검사해야 합니다.
- 우선 모든 경우의 수를 다 구해놓고 그 다음 답을 출력합니다. 

```
#include <iostream>
#include <vector>
using namespace std;

void check_palindromes(vector<int> &nums, vector<vector<bool> > &dp) {
    int N = nums.size();
    for (int i = 0; i < N; i++) {
        dp[i][i] = true;
    }
    for (int i = 0; i < N-1; i++){
        dp[i][i+1] = (nums[i] == nums[i+1]);
    }
    for (int dif = 2; dif < N; dif++) {
        for (int start = 0; start + dif < N; start++) {
            int end = start + dif;
            if (dp[start+1][end-1]) {
                dp[start][end] = (nums[start] == nums[end]);
            }
        }
    }
}

int main(void) {
    int N, M, S, E;
    scanf("%d", &N);
    vector<int> nums(N);
    vector<vector<bool> > is_palindrome(N, vector<bool>(N, false));
    for (int i = 0; i < N; i++) {
        scanf("%d", &nums[i]);
    }

    check_palindromes(nums, is_palindrome);

    scanf("%d", &M);
    for (int i = 0; i < M; i++) {
        scanf("%d %d", &S, &E);
        printf("%c\n", is_palindrome[S-1][E-1] ? '1' : '0');
    }
    
    return 0;
}
```

# [백준 1311 - 할 일 정하기 1](https://www.acmicpc.net/problem/1311)

- DP 인데 bitmask를 함께 사용
- DP 배열 크기가 크므로 스택이 아닌 힙에 할당해야 함
- 가능한 모든 경우를 보는 것이므로, 최대 20 * pow(2, 20) 크기의 배열을 사용
- 이 문제를 풀고 나서 검색해보니 hungarian algorithm이라는 방법을 사용하면 훨씬 더 효율적으로 풀 수 있는 듯합니다. 어려워 보여서 다음으로 미룹니다. 

```
// DP with bitmask
#include <iostream>
#include <vector>
#define INT_MAX 987654321
using namespace std;

// dp 배열크기는 최대  20 * pow(2, 20) * integer 4 byte => 80MB
// 문제 메모리 조건이 128MB이므로 가능
int dfs_with_bitmask(int num_of_works, int left_works, int bitmask, 
        vector<vector<int> > &costs, vector<vector<int> > &dp) {
    
    // for (int i = 0; i < num_of_works; i++) {
    //     for (int j = 0; j < (1 << num_of_works); j++) {
    //         printf("%d ", dp[i][j]);
    //     }
    //     printf("\n");
    // }
    // printf("--------\n");
    
    if (left_works <= 0){
        // end of recursion
        return 0;
    }
    int curr_idx = left_works-1;
    if (dp[curr_idx][bitmask]) {
        // already computed
        return dp[curr_idx][bitmask];
    }

    int result = INT_MAX;
    for (int i = 0; i < num_of_works; i++){
        if(not (bitmask & (1 << i))) {
            bitmask |= (1 << i);
            result = min(result, 
                        dfs_with_bitmask(num_of_works, left_works-1, bitmask, costs, dp) + costs[curr_idx][i]);
            bitmask &= ~(1 << i);
        }
    }

    dp[curr_idx][bitmask] = result;
    return result;
}

int main(void) {
    int num_of_work; // == num of people
    scanf("%d", &num_of_work);
    vector<bool> visited(num_of_work, false);
    vector<vector<int> > costs(num_of_work, vector<int>(num_of_work));
    vector<vector<int> > dp(num_of_work, vector<int>(1 << (num_of_work), 0));        

    for (int i = 0; i < num_of_work; i++) {
        for (int j = 0; j < num_of_work; j++) {
            scanf("%d", &costs[i][j]);
        }
    }

    int initial_bitmask = 0;
    printf("%d", dfs_with_bitmask(num_of_work, num_of_work, initial_bitmask, costs, dp));

    return 0;
}
```


# [백준 17404 - RGB거리 2](https://www.acmicpc.net/problem/17404)
- 처음에는 굉장히 비효율적으로 풀고 통과했습니다. 풀다 보니 이렇게 푸는 게 아닌 것 같은데... 싶었습니다. 
- 그 다음에는 제대로 풀었습니다. 처음 코드와 비교해보면, 처음 코드는 불필요하게 dp 값들을 저장하고, 보지 않아도 되는 경우의 수들을 체크하느라 메모리 사용량도 많고 시간도 오래 걸립니다. 무엇보다 코드가 쓸데없이 깁니다... 


- 비효율적인 코드

```
#include <iostream>
#include <vector>
#define INT_MAX 987654321
using namespace std;


class RGB {
public:
    int r, g, b;
    RGB() : r(0), g(0), b(0) {}
};

class Case {
public:
    int rr, rg, rb, gr, gg, gb, br, bg, bb;
    Case() {
        rr = rg = rb = gr = gg = gb = br = bg = bb = INT_MAX;
    }
    int get_min () {
        int result = INT_MAX;
        result = min(result, rg);
        result = min(result, rb);
        result = min(result, gr);
        result = min(result, gb);
        result = min(result, br);
        result = min(result, bg);
        return result;
    }
};

int main(void) {
    int num_of_houses;
    scanf("%d", &num_of_houses);
    vector<RGB> costs(num_of_houses, RGB());
    vector<vector<Case> > dp(num_of_houses, vector<Case>(num_of_houses, Case()));

    for (int i = 0; i < num_of_houses; i++) {
        scanf("%d %d %d", &(costs[i].r), &(costs[i].g), &(costs[i].b));
    }

    for (int i = 0; i < num_of_houses; i++) {
        dp[i][i].rr = costs[i].r;
        dp[i][i].gg = costs[i].r;
        dp[i][i].bb = costs[i].r;
    }

    for (int i = 0; i < num_of_houses-1; i++)
    {
        dp[i][i+1].rg = costs[i].r + costs[i+1].g;
        dp[i][i+1].rb = costs[i].r + costs[i+1].b;
        dp[i][i+1].gr = costs[i].g + costs[i+1].r;
        dp[i][i+1].gb = costs[i].g + costs[i+1].b;
        dp[i][i+1].br = costs[i].b + costs[i+1].r;
        dp[i][i+1].bg = costs[i].b + costs[i+1].g;
    }
    
    for (int dif = 2; dif < num_of_houses; dif++) {
        for (int start = 0; start + dif < num_of_houses; start++) {
            int end = start + dif;
            dp[start][end].rr = min(dp[start][end].rr, dp[start][end-1].rb + costs[end].r);
            dp[start][end].rr = min(dp[start][end].rr, dp[start][end-1].rg + costs[end].r);

            dp[start][end].rb = min(dp[start][end].rb, dp[start][end-1].rr + costs[end].b);
            dp[start][end].rb = min(dp[start][end].rb, dp[start][end-1].rg + costs[end].b);

            dp[start][end].rg = min(dp[start][end].rg, dp[start][end-1].rr + costs[end].g);
            dp[start][end].rg = min(dp[start][end].rg, dp[start][end-1].rb + costs[end].g);

            dp[start][end].gr = min(dp[start][end].gr, dp[start][end-1].gg + costs[end].r);
            dp[start][end].gr = min(dp[start][end].gr, dp[start][end-1].gb + costs[end].r);

            dp[start][end].gg = min(dp[start][end].gg, dp[start][end-1].gr + costs[end].g);
            dp[start][end].gg = min(dp[start][end].gg, dp[start][end-1].gb + costs[end].g);

            dp[start][end].gb = min(dp[start][end].gb, dp[start][end-1].gr + costs[end].b);
            dp[start][end].gb = min(dp[start][end].gb, dp[start][end-1].gg + costs[end].b);

            dp[start][end].br = min(dp[start][end].br, dp[start][end-1].bg + costs[end].r);
            dp[start][end].br = min(dp[start][end].br, dp[start][end-1].bb + costs[end].r);

            dp[start][end].bg = min(dp[start][end].bg, dp[start][end-1].br + costs[end].g);
            dp[start][end].bg = min(dp[start][end].bg, dp[start][end-1].bb + costs[end].g);

            dp[start][end].bb = min(dp[start][end].bb, dp[start][end-1].br + costs[end].b);
            dp[start][end].bb = min(dp[start][end].bb, dp[start][end-1].bg + costs[end].b);
        }
        
    }
    
    printf("%d", dp[0][num_of_houses-1].get_min());
    return 0;
}
```

- 제대로 푼 코드

```
#include <iostream>
#include <vector>
#define INT_MAX 987654321
using namespace std;


int main(void) {
    int num_of_houses;
    scanf("%d", &num_of_houses);
    vector<vector<int> > costs(num_of_houses, vector<int>(3));
    for (int i = 0; i < num_of_houses; i++) {
        scanf("%d %d %d", &(costs[i][0]), &(costs[i][1]), &(costs[i][2]));
    }

    int result = INT_MAX;

    // color: 0: red, 1: green, 2: blue
    for (int first_color = 0; first_color < 3; first_color++) {
        vector<vector<int> > dp(num_of_houses, vector<int>(3, INT_MAX));
        dp[0][first_color] = costs[0][first_color];

        for (int house_idx = 1; house_idx < num_of_houses; house_idx++) {
            dp[house_idx][0] = min(dp[house_idx-1][1], dp[house_idx-1][2]) + costs[house_idx][0];
            dp[house_idx][1] = min(dp[house_idx-1][0], dp[house_idx-1][2]) + costs[house_idx][1];
            dp[house_idx][2] = min(dp[house_idx-1][0], dp[house_idx-1][1]) + costs[house_idx][2];
        }
        
        for (int curr_color = 0; curr_color < 3; curr_color++) {
            if (first_color != curr_color){
                result = min(result, dp[num_of_houses-1][curr_color]);
            }
        }   
    }
    
    printf("%d", result);
    
    return 0;
}
```

# [백준 1086 - 박성원](https://www.acmicpc.net/problem/1086)

