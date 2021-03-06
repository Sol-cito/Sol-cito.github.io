---
layout: post
title: "[Algorithm] 이분탐색(Binary Search)"
subtitle: "Algorithm & Data Structure"
background: '/img/bg_technology.jpg'
categories: technology-algorithm
---



### 이분탐색의 개념

이분탐색, 혹은 이진탐색이라 불리는 이 탐색 방법은 Worst-case 일 때 시간복잡도가 O(logN)인 알고리즘으로, 단순히 index 0~끝까지 linear search를 하는 것 보다 빠르다.



크기가 100인 Array[] 가 1부터 순서대로 있다고 가정했을 때, 숫자 100을 탐색하려면 linear search의 Worst Case는 **100회**가 된다.  1부터 계속 탐색을 해나가다가 맨 마지막 탐색시도 때 100을 찾게 되기 때문이다.

따라서 Linear Search의 Performance는 O(N)이다.



이분탐색의 방법은 다음과 같다.

만약 Array[]가 오름차순으로 정렬되어 있다면, 맨 가운데 숫자 M을 먼저 탐색하여 찾는 값보다 큰지 작은지 비교한다.

M이 찾는 값보다 크다면,  M의 왼쪽 50개가 다음 탐색의 대상이 되며, M이 찾는 값보다 작다면 M의 오른쪽 50개가 다음 탐색의 대상이 된다.

다음 턴의 M은 다음 탐색 대상 50개의 가운데 숫자가 된다.

그 다음 턴의 M은 다음 탐색 대상 25개의 가운데 숫자가 된다.

이런식으로 반씩 쪼개서 탐색을 한다면 전체 탐색 대상의 수가 N에서 N * (1/2)로 점차 줄게 되므로, 

Worse case에서(반씩 쪼개나가다가 전체 탐색 대상의 수가 1이 되어 비교연산을 1회만 수행할 때)는 

**N* (1/2)^k = 1** 의 식을 세울 수 있다.

이 때 k는 반으로 쪼갠 횟수이므로, k에 대해 식을 정리하고(양 변에 log를 씌우면), 시간 복잡도 함수로 나타내면

**T(n) = logN** 의 식이 성립된다.



---



### 이분탐색 Java로 직접 구현하여 Linear Search와 비교해보기

그럼 정말로 Binary Search가 Linear Search보다 빠를까? 얼마나 빠를까? 

직접 그 결과를 눈으로 보면 머릿속에 각인이 더 잘 될 터이니, 직접 Java로 구현하여 속도를 측정해보았다.



##### Binary Search Test source code

```Java
public class BinarySearch{
	public static void main(String[] args) {
		ArrayList<Long> arr = new ArrayList<Long>();
		/* 크기가 100,000,000인 배열 오름차순으로 생성 */
		for (long i = 0; i < 20000000L; i++) {
			arr.add(i);
		}
		/* 찾을 타겟 */
		long target = 19999999L;

		/* Binary Search */
		long Bstart = System.currentTimeMillis();
		int index = arr.size() / 2;
		System.out.println("B시작 : " + Bstart);
		Bsearch(arr, target, index);
		long Bend = System.currentTimeMillis();
		System.out.println("끝 : " + Bend);
		System.out.println("걸린 시간 : " + (double) (Bend - Bstart) / 1000 + "초");

		/* Linear Search */
		long Lstart = System.currentTimeMillis();
		System.out.println("L시작 : " + Lstart);
		Lsearch(arr, target);
		long Lend = System.currentTimeMillis();
		System.out.println("끝 : " + Lend);
		System.out.println("걸린 시간 : " + (double) (Lend - Lstart) / 1000 + "초");
	}

	public static void Bsearch(ArrayList<Long> arr, long target, int index) {
        int preIndex = arr.size(); //이전의 index를 저장하는 포인터
		while (true) {
			long middle = arr.get(index);
			if (middle == target) {
				break;
			}
			if (middle < target) {
				index = (index + preIndex) / 2;
			} else {
                preIndex = index;
				index = index / 2;
			}
		}
	}

	public static void Lsearch(ArrayList<Long> arr, long target) {
		for (long each : arr) {
			if (each == target) {
				return;
			}
		}
	}
}
```



20000000L 의 크기를 가진 ArrayList에서 19999999L 값을 Binary search와 Linear search로 각각 탐색하여 소요 시간을 System.currentTimeMillis() 로 측정하는 테스트 코드다. 

찾으려는 값이 19999999L 이기 때문에 search 둘 다 worse case이다.

위 코드 실행 시 이클립스 콘솔창에 나타나는 결과는 아래와 같다.

```
B시작 : 1584517595143
끝 : 1584517595144
걸린 시간 : 0.001초

L시작 : 1584517595153
끝 : 1584517595375
걸린 시간 : 0.222초
```

Binary search가 확실히 빠르다는 사실을 직접 확인할 수 있다. 



약 0.2초 차이는 별 것 아니라고 생각될 수도 있지만, 어떤 문제에서 이러한 탐색을 N회 반복적으로 수행해야 하는 경우라면? 

N을 곱한만큼 탐색 시간이 늘어나게 될 텐데, 만약 N이 100,000,000 같은 또 다른 거대한 수라면,

두 search의 탐색 시간 차이는 0.2 * 100,000,000 = 20,000,000초 -> **약 5555시간**이 되어버린다.

따라서 데이터의 크기와 탐색 횟수가 커질수록, Binary search를 사용하는 것이 성능이 훨씬 뛰어나다.



**하지만 무조건 그렇다는 건 아니다.** Binary Search에는 중요한 전제조건이 있기 때문이다.



---



### Binary Search와 Sorting

Binary Search 로직의 가장 중요한 포인트는 '양쪽으로의 분기'다. 

즉, 가운데 수보다 크거나 작을 때 각각 왼쪽, 오른쪽으로 분기하여 탐색을 다시 시도한다는 것이다.

따라서 이 경우, 나열된 수들은 무조건 **'오름차순/내림차순 정렬**'이 되어있어야 한다.

만약 정렬이 되어있지 않은 수들을 이진 검색한다면 우선 정렬을 해주어야 한다,



그런데 어떤 문제에서, 정렬 + 이진탐색을 한 뒤, 새로운 값이 추가되거나 빠지게 되어 다시 새로이 정렬을 해주어야 이진 탐색이 가능한 경우라면?

Linear search의 시간복잡도는 그대로 **O(N)** 이 될 것이다.

그러나 Binary search 의 시간복잡도는 **[(정렬의 시간복잡도) + logN]** 가 될 것이기에, 

O(N)가 소요되는 정렬 알고리즘을 쓴다면, 오히려 정렬을 하지않고 그대로 계속 Linear search를 하는 것이 더 나을 수도 있다!

따라서 이 경우는 정렬 알고리즘을 잘 골라서 써야만 할 것이다.



이분탐색을 활용하여 푼 알고리즘 문제는 아래와 같다.