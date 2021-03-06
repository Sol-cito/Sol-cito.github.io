---
layout: post
title: 프로그래머스 그래프 - 사이클 제거 (Level 4)
subtitle: Algorithm Solution
background: '/img/bg_technology.jpg'
categories: technology-algorithm

---


##### 문제 설명
n개의 노드로 구성된 그래프에서 하나의 노드만을 제거해서 사이클이 없도록 만들고 싶습니다.

노드의 개수 n, 노드간의 연결 edges가 매개변수로 주어질 때, 노드를 딱 하나 제거해서 그래프를 사이클이 없도록 만들 수 있다면 
해당 노드의 번호를 return 하도록 solution 함수를 완성하세요.
단, 그런 노드가 없다면 0을 return하고, 여러 개라면 노드의 번호의 합을 return하세요.

##### 제한사항

노드 번호는 1 부터 시작합니다.
주어진 그래프에 반드시 하나 이상의 사이클은 있습니다.
노드간의 연결에는 방향이 없습니다.
주어지는 노드의 수는 2개 이상 5,000개 이하입니다.
연결의 수는 1개 이상 n(n + 1)/2개 이하입니다.
입출력 예

n	edges	result
4	[[1,2],[1,3],[2,3],[2,4],[3,4]]	5
8	[[1,2],[2,3],[3,4],[4,5],[5,6],[6,7],[7,8],[8,1],[2,7],[3,6]]	0


출처 : https://programmers.co.kr/learn/courses/30/lessons/49188


#### My solution (Java)

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Stack;

class Solution {
	HashMap<Integer, Boolean> visit = new HashMap<Integer, Boolean>();
	HashMap<Integer, Node> nodeMap = new HashMap<Integer, Node>();
	ArrayList<Integer> suspect = new ArrayList<Integer>();

	public int solution(int n, int[][] edges) {
		int answer = 0;

		/* n개의 node를 만들고 nodelist에 넣고, visit을 false로 만든다. */
		for (int i = 0; i < n; i++) {
			ArrayList<Integer> linked = new ArrayList<Integer>();
			Node node = new Node(i + 1, linked);
			nodeMap.put(i + 1, node);
			visit.put(i + 1, false);
		}

		/* 각 노드의 linked에 연결되어있는 다른 노드를 담는다 */
		for (int i = 0; i < edges.length; i++) {
			for (int j = 0; j < 1; j++) {
				nodeMap.get(edges[i][j]).linked.add(edges[i][j + 1]);
				nodeMap.get(edges[i][j + 1]).linked.add(edges[i][j]);
			}
		}

		/* 노드를 하나씩 빼면서 사이클이 있는지 DFS 탐색한다 */
		ArrayList<Integer> resultSet = new ArrayList<Integer>();
		for (int i = 1; i <= n; i++) {
			boolean result = false;
			Stack<Node> stack = new Stack<Node>();
			if (i == 1) {
				result = DFS(2, i, stack, n);
			} else {
				result = DFS(1, i, stack, n);
			}
			if (result == true) {
				resultSet.add(i);
			}
			/* for문이 돌 때마다 visit의 node를 전부 false로 바꿔줌 */
			for (int j = 1; j <= visit.size(); j++) {
				visit.replace(j, false);
			}
		}
		if (resultSet.size() == 0) {
			answer = 0;
		} else {
			for (int each : resultSet) {
				answer += each;
			}
		}
		return answer;
	}

	public boolean DFS(int start, int takenOut, Stack<Node> stack, int n) {
		HashMap<Integer, Boolean> pushCheck = new HashMap<Integer, Boolean>();
		for (int i = start; i <= n; i++) {
			/* 해당 노드가 takenOut이거나, 이미 방문헀으면 continue */
			if (i == takenOut || visit.get(i)) {
				continue;
			}
			stack.push(nodeMap.get(i));
			pushCheck.put(i, true);

			while (true) {
				if (stack.isEmpty()) {
					break;
				}
				/* 꺼내고 방문체크 */
				Node node = stack.pop();
				visit.replace(node.id, true);
				/* 연결노드 체크해서 stack에 담기 */
				for (int j = 0; j < node.linked.size(); j++) {
					try {
						if (visit.get(node.linked.get(j)) == false && node.linked.get(j) != takenOut) {
							if (pushCheck.get(node.linked.get(j)) == true) {
								return false;
							}
							stack.push(nodeMap.get(node.linked.get(j)));
							pushCheck.put(node.linked.get(j), true);
						}
					} catch (NullPointerException e) {
						if (visit.get(node.linked.get(j)) == false && node.linked.get(j) != takenOut) {
							stack.push(nodeMap.get(node.linked.get(j)));
							pushCheck.put(node.linked.get(j), true);
						}
					}
				}
			}
		}
		return true;
	}
	
	class Node {
		int id;
		ArrayList<Integer> linked;

		public Node(int id, ArrayList<Integer> linked) {
			this.id = id;
			this.linked = linked;
		}
	}
}
```



#### Feedback

```
정확도 테스트는 모두 통과했으나(로직은 맞았다는 뜻), 효율성 테스트를 하나도 통과하지 못했다.
그도 그럴것이, 주어진 노드를 하나씩 빼면서 사이클이 있는지 없는지 DFS로 탐색을 해보는데, 
이 과정만 해도 반목문이 몇 개 는 겹쳐지기에 시간복잡도가 굉장히 크기 때문이다.
'사이클이 존재하는지를 수학적으로 판별하는 방법'이 있다면 그것이 best solution일텐데, 
딱히 생각나지 않아서 완전탐색으로 접근했다. 그러나 효율성이...
이 문제는 다른 사람들의 풀이 & 정답을 참고하면서 다시 풀어볼 예정이다.
```

