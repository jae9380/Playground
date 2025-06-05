#HackerRank #SQL

# [Advanced Select - Binary Tree Nodes](https://www.hackerrank.com/challenges/binary-search-tree-1/problem?isFullScreen=true)

## 문제 
BST 라는 테이블이 주어진다. 해당 테이블은 2개의 열 N과 P를 포함한다.     
여기서 N은 이진 트리의 노드를 나타내고, P는 해당 노드 N의 부모 노드를 나타낸다.

| Column | Type    |
|:------:|:-------:|
| N      | Integer |
| P      | Integer |
이진 트리의 각 노드 값을 기준으로 정렬하고, 해당 노드의 타입을 찾아야 한다.

### Node Type
* **Root** : 해당 노드가 루트 노드인 경우
* **Leaf** : 해당 노드가 말단 노드인 걍우
* **Inner** : 해당 노드가 루트도 아니고 리프도 아닌 경우


---
## 쿼리
```sql
select n,
    case
        when p is null then 'Root'
        when n not in (select distinct p from BST where p is not null) then 'Leaf'
        else 'Inner'
    end as NodeType
from BST
order by n;
```

### 사용된 쿼리

```sql
CASE
    WHEN 조건1 THEN 결과1
    WHEN 조건2 THEN 결과2
    ...
    ELSE 결과N
END
```
`CASE`는 위에서 부터 조건문을 검증한다. 해당 조건문이  `true`가 될 경우 대응하는 `THEN`이 동작하고, 만약 해당 조건이 `false`일 경우 다음 조건식으로 넘어가게 된다.     

그리고 모든 조건에 대해서 참이 없을 경우 `ELSE`절의 결과를 반영하게 된다.

```sql
	when n not in (select distinct p from BST where p is not null) 
```

해당 부분 쿼리는 n이 서브 쿼리에 포되지 않았는지 확인하게 된다. 

---

#### 서브 쿼리
```sql
select distinct p from BST
```

해당 부분은 BST에서 중복없이 P의 값을 추출한다.  해당 값을 추출할 떄 추가적인 조건이 붙게 된다.    

```sql
WHERE p IS NOT NULL
```

이는 P의 값이 Null일 경우 해당 경우는 제외하고, Null이 아닌 경우만 동작을 하게 된다.


---

`CASE`절이 끝나고 바로 다음으로 이어지는 쿼리이다.    
```sql
    end as NodeType
```

`CASE` 쿼리를 통해서 산출되는 값 `Root`, `Leaf`, `Inner` 중 하나를 `NodeType` 칼럼으로 부르겠다는 의미가 된다.    

그리고 마지막으로 데이터를 n의 값을 기준으로 오름차순으로 정렬하겠다는 의미가 된다.
```sql
order by n;
```

