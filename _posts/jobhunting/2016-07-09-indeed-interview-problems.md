---
layout: post
title: Indeed 2017 Online Test
category: Job Hunting
tags: [job, algorithm, interview]
---
{% include JB/setup %}

### Tables and Pieces

> There is a 6×6 table. Place a number of pieces on the table to meet the following conditions:

> - There is to be either zero or one piece in each square of the table.
> - Each column in the table is to have exactly three pieces.
> - Each row in the table is to have exactly three pieces.
>
> There may already be pieces in some of the squares of the table. When si,j is 'o', there is already a piece in the square at the jth column of the ith row. When that is not the case, si,j is '.' and there is no piece in that square. Find out the number of ways to place pieces in the empty squares, which satisfies the conditions. Two ways are considered different if they have at least one square which contains a piece in one way and doesn't contain a piece in the other way.

#### Constrains
- si,j is 'o' or '.'

#### Input
Inputs are provided from standard inputs in the following form.

> s1,1 … s1,6<br>
> :<br>
> s6,1 … s6,6

#### Output
Output the number of ways that pieces can be placed on the empty squares to satisfy the conditions.

Input Example #1 <br>
...... <br>
.o.... <br>
...o.. <br>
...o.. <br>
...... <br>
...... <br>

Output Example #1 <br>
32692

Input Example #2 <br>
oooo.. <br>
...... <br>
...... <br>
...... <br>
...... <br>
...... <br>

Output Example #2 <br>
0

#### Solution

DFS遍历所有可能，每行最多有C(6, 3)=20种可能，因此复杂度为O(n^3), 计算总次数为20^6=64*10^6.

```c++
#include<stdio.h>
#include<iostream>
using namespace std;

const int N = 6;
const int M = 3;

char board[N][8];
int rowCount[N] = {0, 0, 0, 0, 0, 0};
int colCount[N] = {0, 0, 0, 0, 0, 0};

int posGroup[M];

void initPosGroup(int* posGroup, int posCount) {
	for(int i=0;i<posCount;++i)
		posGroup[i] = i;

	--posGroup[posCount-1];
}

bool nextPosGroup(int* posGroup, int posCount) {
	for(int i=posCount-1;i>=0;--i) {
		if(posGroup[i] < N-posCount+i) {
			++posGroup[i];
			for(int j=i+1;j<posCount;++j)
				posGroup[j] = posGroup[j-1]+1;

			return true;
		}
	}

	return false;
}

bool check(int row, int* posGroup, int posNum) {
	for(int i=0;i<posNum;++i) {
		if(board[row][posGroup[i]] == 'o' || colCount[posGroup[i]]+1>M)
			return false;
	}

	return true;
}

void update(int row, int* posGroup, int posNum) {
	for(int i=0;i<posNum;++i) {
		board[row][posGroup[i]] = 'o';
		++colCount[posGroup[i]];
	}
}

void inUpdate(int row, int* posGroup, int posNum) {
	for(int i=0;i<posNum;++i) {
		board[row][posGroup[i]] = '.';
		--colCount[posGroup[i]];
	}
}

int recursive(int row) {
	if(row == 6) return 1;
	int posNum = M-rowCount[row];

	int posGroup[posNum];
	initPosGroup(posGroup, posNum);

	int ans = 0;
	while(nextPosGroup(posGroup, posNum)) {
		if(check(row, posGroup, posNum)) {
			update(row, posGroup, posNum);

			ans += recursive(row+1);

			inUpdate(row, posGroup, posNum);
		}
	}

	return ans;
}

int main() {
	for(int i=0;i<N;++i) {
		scanf("%s", board[i]);

		for(int j=0;j<N;++j) {
			if(board[i][j] == 'o') {
				++rowCount[i];
				++colCount[j];
			}
		}
	}

	cout<<recursive(0)<<endl;

	return 0;
}
```

### Balance Parentheses

> Given a string S contains only '(' and ')'. A balance parentheses (BP) is that:
> - An empty string is a BP;
> - If x is a BP, '('x')' is a BP;
> - If x and y are BP, xy is a BP;
>
> To make S a BP, insert '(' or ')' at ith position cost A[i]. Calculate the minimum cost to make S a BP.

**Solution:**

1. 首先计算minAForward[i] = min{A[0]...A[i]}, minABackward[i] = min{A[i]...A[N]};
2. 正向遍历S, 对'('和')'计数，当')'的数目超过'('时，在该位置插入'(', 并清空计数，重新开始；
3. 插入')'时是倒向遍历S，并在'('的数目超过')'时插入')', 需要注意的是插入')'是在当前位置的下一位置.

注意: 在S的尾部也可以插入新字符，因此计算时不要忽略A[N].

```c++
#include<stdio.h>
#include<iostream>
using namespace std;

char S[300005];
int A[300005];
int minAForward[300005];
int minABackward[300005];

int main() {
	int N;
	scanf("%d", &N);
	scanf("%s", S);

	for(int i=0;i<=N;++i) scanf("%d", &A[i]);

	minAForward[0] = A[0];
	for(int i=1;i<=N;++i) minAForward[i] = min(minAForward[i-1], A[i]);
	minABackward[N] = A[N];
	for(int i=N-1;i>=0;--i) minABackward[i] = min(minABackward[i+1], A[i]);

	long long cost = 0;

	int bla = 0;
	for(int i=0;i<N;++i) {
		bla += S[i]==')' ? -1 : 1;
		if(bla < 0) {
			cost += minAForward[i];
			bla = 0;
		}
	}

	bla = 0;
	for(int i=N;i>0;--i) {
		bla += S[i-1]=='(' ? 1 : -1;
		if(bla > 0) {
			cost += minABackward[i];
			bla = 0;
		}
	}

	cout<<cost<<endl;
	return 0;
}
```

Test case:

> 8 <br>
> ))())(() <br>
> 1 2 3 4 5 6 7 8 9

Answer:

> 10
