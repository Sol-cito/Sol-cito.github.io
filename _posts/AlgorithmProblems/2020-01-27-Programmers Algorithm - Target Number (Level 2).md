---
layout: post
title: 프로그래머스 탐색 - 타겟넘버 (Level 2)
subtitle: Algorithm Solution
background: '/img/bg_technology.jpg'
categories: technology-algorithm

---


##### 문제 설명
n개의 음이 아닌 정수가 있습니다. 이 수를 적절히 더하거나 빼서 타겟 넘버를 만들려고 합니다. 예를 들어 [1, 1, 1, 1, 1]로 숫자 3을 만들려면 다음 다섯 방법을 쓸 수 있습니다.

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3
사용할 수 있는 숫자가 담긴 배열 numbers, 타겟 넘버 target이 매개변수로 주어질 때 숫자를 적절히 더하고 빼서 타겟 넘버를 만드는 방법의 수를 return 하도록 solution 함수를 작성해주세요.

##### 제한사항
주어지는 숫자의 개수는 2개 이상 20개 이하입니다.
각 숫자는 1 이상 50 이하인 자연수입니다.
타겟 넘버는 1 이상 1000 이하인 자연수입니다.
입출력 예
numbers	target	return
[1, 1, 1, 1, 1]	3	5

출처 : https://programmers.co.kr/learn/courses/30/lessons/43165




---

#### My solution (Java)

```java
class Solution {
	static int count = 0;

	public int solution(int[] numbers, int target) {
		int answer = 0;
		find(0, 0, numbers, target);
		answer = count;
		return answer;
	}

	public static void find(int start, int sum, int[] numbers, int target) {
		if (start >= numbers.length) { // 끝까지 왔을 때
			if (sum == target) {
				count++;
			}
			return;
		}
		int plus = sum + numbers[start];
		int minus = sum - numbers[start];

		find(start + 1, plus, numbers, target);
		find(start + 1, minus, numbers, target);
	}
}
```



---

#### Feedback

```
+1 과 (-1) 두 가지 경우를 반복해서 탐색하는, 탐색 문제의 기본이다.
나는 아직 이 시점에서 스택과 큐를 익히지 않았기 때문에 재귀 함수를 이용해서 풀었다.
재귀함수를 분기하면 자연스럽게 DFS를 만들 수 있다. 이것이 코드 아래쪽에 find 재귀함수를 두 개 생성한 이유이다. 

즉, 재귀함수가 한 번 호출될 때 마다, 하위 재귀함수는 두 갈래로 분기되어 각각 호출된다. 이 때 파라미터로 받는 sum이 위 int에서 plus 와 minus 로 각각 계산되어 들어오기 때문에 +1 와 (-1)의 두 가지 경우가 저절로 생성되어 분기를 타게 된다.

처음 이 문제를 접했을 때는 재귀함수와 탐색 개념을 익히지 않았을 때라 for문으로만 풀려고 고민했었는데, 너너무나 경우의 수가 많아서 for문을 일일이 쓰기 불가능하다는 것을 깨달았다(아니, 쓸 수도 있지만 내 깜냥으로는 for문으로 풀 수가 없었다. for문으로만 풀 수 있는 방법이 있을까?). 재귀와 탐색의 개념을 조금이라도 익히고 나니 쉽게 풀리는 문제였다.
```

