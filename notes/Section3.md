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

- 데이터가 useQuery와는 다릅니다.
- Object는 두 가지 프로퍼티를 가지고 있습니다.
    1. pages
        1. 페이지에 대한 Object 배열입니다.
    2. pageParams
        1. 실제로 널리 사용되지는 않습니다.
- 모든 쿼리는 Page Array에 대한 고유 Element를 가지고 있습니다.
- pageParams는 검색된 쿼리의 Key를 추적합니다.
    - 일반적으로 사용되지 않으며 여기서는 사용하지 않습니다.

### useInfiniteQuery Syntax

- pageParam은 queryFn의 파라미터입니다.
    - `useInfiniteQuery('sw-people', ({ pageParam = defaultUrl }) ⇒ fetchUrl(pageParam)`
    - pagination과는 달리 pageParam은 `React Query`가 관리합니다.
- useInfiniteQuery options
    - getNextPageParam: (lastPage, allPages)
        - 다음 페이지를 가져오는 방법을 알려줍니다.
        - allPages, lastPage를 알려줍니다.
            - 그리고 pageParam을 업데이트 합니다.
            - 데이터의 모든 페이지에 사용합니다. (allPages)
            - 여기서는 데이터의 마지막 페이지에 사용합니다. (next property를 사용해서)

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
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery(
    'sw-people',
    ({ pageParam = initialUrl }) => fetchUrl(pageParam),
    {
      getNextPageParam: (lastPage) => lastPage.next || undefined
    }
  );

  return (
    <InfiniteScroll loadMore={fetchNextPage} hasMore={hasNextPage}>
      {data.pages.map(pageData => {
				/* Components */
      })}
    </InfiniteScroll>
  );
}
```