# Section 3.

# Infinite Scroll

- Starwars API 사용

## Second Project: Infinite StarWars API

- Infinite Scroll
    - 데이터를 한 번에 가져오는 것 보다 스크롤에 따라 필요한만큼 가져오는게 훨씬 효율적입니다.
- 데이터를 가져올 때
    - 사용자가 버튼을 클릭할 때
    - 스크롤이 바닥에 닿았을 때
- 이 때 사용할 수 있는 useInfiniteQuery가 있다.

## useInfiniteQuery

- Pagination
    - 컴포넌트 state에 있는 current page를 Tracking
    - 새로운 쿼리가 page number를 업데이트
- useInfiniteQuery
    - next query는 다음과 같은 데이터의 일부로 반환됩니다.

    ```jsx
    {
    	"count": 37
    	"next": "http://swap.dev/api/species/?page=2",
    	"previous": null,
    	"results": [ ... ],
    }
    ```

### Shape of useInfiniteQuery Data

```jsx
const { data } = useInfiniteQuery(...);
data.pages // (1)
data.pageParams // (2)
```

- 데이터가 useQuery와는 다릅니다.
- Object는 두 가지 프로퍼티를 가지고 있습니다.
    1. pages
        1. 현재까지 불러온 페이지 데이터 배열입니다.
    2. pageParams
        1. 모든 페이지 매개변수를 포함하는 배열
        2. 실제로 널리 사용되지는 않습니다.

![Untitled](Section%203%20cf5eba22e8c1433aba78f924be0736e0/Untitled.png)

### useInfiniteQuery Syntax

- pageParam은 queryFn의 파라미터입니다.
    - `useInfiniteQuery('sw-people', ({ pageParam = defaultUrl }) ⇒ fetchUrl(pageParam)`
    - pagination과는 달리 pageParam은 `React Query`가 관리합니다.
- useInfiniteQuery의 options 매개변수
    - getNextPageParam: (lastPage, allPages)
        - 다음 페이지를 가져오는 방법을 알려줍니다.
        - allPages, lastPage를 알려줍니다.
            - 그리고 pageParam을 업데이트 합니다.
            - lastPage는 가장 최근에 받아왔던 데이터를 의미합니다.
            - allPages는 받아왔던 모든 데이터의 배열을 의미합니다.
            - 여기서는 lastPage에 결과값 중 next property를 사용합니다.

```jsx
const { data, fetchNextPage, hasNextpage } = useInfiniteQuery(
    'sw-people',
    ({ pageParam = initialUrl }) => fetchUrl(pageParam) // <--- pageParam
  );
```

- next query는 다음 데이터의 일부를 반환합니다.

```jsx
{
	"count": 37,
	"next": "http://swapi.dev/api/species/?page=2",
	"previous": null,
	"results": [ ... ]
}
```

### useInfiniteQuery는 Opject Properties를 반환합니다.

- fetchNextPage
    - 사용자가 다음 데이터를 필요로 할 때 마다 호출
- hasNextPage
    - getNextPageParam의 결과값을 기반으로 합니다.
    - `getNextPageParam: (lastPage) => lastPage.next || undefined` 여기서 lastPage.next이 undefined를 반환하면 데이터가 더이상 없음을 의미합니다.
- isFetchingNextPage
    - loading spinner를 표시하기 위해 사용
    - 우리는 이것을 isFetching, isFetchingNextPage과 구분지어서 이것이 얼마나 유용한지 살펴볼 예정입니다.

    ```jsx
    const { data, fetchNextPage, hasNextPage } = useInfiniteQuery();
    ```

### The Flow

1. Components mounts
    1. const { data } = useInfiniteScroll( ... )
        1. data: undefined
        2. pageParam: default
2. Fetch first page
    1. const { data } = useInfiniteScroll( ... )
        1. data.pages[0] = { ... }
        2. pageParam: default
3. getNextPageParam, Update PageParam
    1. getNextPageParam: (lastPage, allPages) ⇒ ...
        1. pageParam: "http://swapi.dev/api/species/?page=2"
    2. hasNextPage
        1. true
4. user scrolls / click button
    1. fetchNextPage 호출
    2. data.pages[1] = { ... }
    3. hasNextPage
        1. false
5. Finish

### React Infinite Scroller

- 여기서는 아래 라이브러리를 사용할 예정
    - [https://github.com/danbovey/react-infinite-scroller](https://github.com/danbovey/react-infinite-scroller)
- 이 라이브러리에서 제공하는 InfiniteScroll 컴포넌트에 prop로 다음 데이터를 넘겨줘야함
    - loadMore = {fetchNextPage}
    - hasMore = {hasNextpage}

```jsx
export function InfinitePeople() {
  const { data, fetchNextPage, hasNextPage, isLoading, isFetching, isError, error } = useInfiniteQuery(
    'sw-people',
    ({ pageParam = initialUrl }) => fetchUrl(pageParam),
    {
      getNextPageParam: (lastPage) => lastPage.next || undefined
    }
  );

  if (isLoading) return <div className="loading">Loading...</div>
  if (isError) return <div>Error! {error.toString()}</div>

  return (
    <>
      {isFetching && <div className="loading">Loading...</div>}
      <InfiniteScroll loadMore={fetchNextPage} hasMore={hasNextPage}>
        {data.pages.map(pageData => {
          return pageData.results.map(person => (
            <Person 
              key={person.name} 
              name={person.name} 
              hairColor={person.hair_color}
              eyeColor={person.eye_color}
            />
          ))
        })}
      </InfiniteScroll>
    </>
  );
}
```

### Summary

- pageParam
    - 다음 페이지를 불러오기 위한 파라미터
    - getNextPageParam 옵션의 결과값이 넘어옴
    - lastPage, allPages를 통해 받아온 값을 이용해서 pageParam으로 넘길 수 있음
- hasNextPage
    - getNextPageParam의 결과의 존재 유무를 이용해서 판단
    - 만약 getNextPageParam의 결과가 undefined면 false를 반환
- fetchNextPage함수를 이용해서 다음 함수를 불러옴
    - hasNextPage를 이용해서 언제 stop할지를 결정