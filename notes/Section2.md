# Section 2.

# Project blog-em-code-at-section-2-start

- useQuery를 사용하세요
- Account for error, loading and results
- Post와 Query key가 다른지 꼭 확인하세요
    - React Query는 cache / stale time을 위해 key를 사용
- 여기서 query 함수는 postId를 파라미터로 필요로 합니다.
    - () ⇒ fetchComments(post.id)
- Warning: 다른 포스트를 클릭하더라도 댓글은 갱신되지 않습니다. 댓글이 새로고침이 안되는 이유는 밑에서 다룰 예정입니다.

```jsx
export function PostDetail({ post }) {
  const { data, isLoading } = useQuery('fetchComments', () => fetchComments(post.id));

  return (
    <>
      {
        isLoading ? <div>Loading...</div> : (
          <>
            <h3 style={{ color: "blue" }}>{post.title}</h3>
            <button>Delete</button> <button>Update title</button>
            <p>{post.body}</p>
            <h4>Comments</h4>
            {data.map((comment) => (
              <li key={comment.id}>
                {comment.email}: {comment.body}
              </li>
            ))}
          </>
        )
      }
    </>
  );
}
```

### 왜 댓글이 Refresh되지 않을까요?

- 위 코드에서는 댓글에서 모든 Query가 같은 Key를 사용하고 있습니다.
- 이미 어떤 Key로 Fetching이 일어난 쿼리 데이터는 트리거가 발생할때만 다시 가져옵니다.
    - ex)
        - Component Re-mount될 때
        - 창이 다시 Focus 될 때
        - Refetch 함수가 실행될 때 ( queryClient.refetchQueries() )
        - 자동으로 Refetch할 때 (refetchInterval 사용)
            - [https://react-query.tanstack.com/examples/auto-refetching](https://react-query.tanstack.com/examples/auto-refetching)
        - mutation 후에 query를 invalidation시킬때 ( queryClient.invalidateQueries() )

### Solution?

- Option: 새 게시물 제목을 클릭할 때마다 새로고침을 트리거합니다.
    - 이것은 쉽지 않음..
    - 이건 우리가 정말 원하지 않는 방법...
- 우리는 캐시에 있는 데이터를 제거할 이유가 없습니다.
    - 우리는 다른 코멘트에 대해 같은 Query를 사용하지 않기 때문입니다.
- 쿼리 키에 post id를 포함합니다.
    - 쿼리별로 캐시를 가지고 있을 수 있습니다.
    - post id가 다르다면 같은 쿼리 캐시를 공유하지 않아야 합니다.
- 각 post에 대한 쿼리에 개별적으로 레이블을 지정하여 이를 수행할 수 있습니다.

### Arrays as Query Key

`**['comments', post.id]**`

- 이 경우 문자열 대신 쿼리 키에 대한 배열을 전달하여 이를 수행할 수 있습니다.
- 이 경우 키가 변경되면 쿼리가 이를 처리하고, post id가 업데이트되면 새로운 React Query를 만듭니다.

### Pagination

- 컴포넌트 state의 currentPage를 Tracking해서 구현해봅시다.
- page number가 포함된 Query Key를 사용합시다
- User는 next page 또는 previous page버튼을 클릭합니다.
    - 이 떄 currentPage 상태를 변경합니다.
    - 쿼리가 실행되는 것을 확인합니다.
- 이렇게하면 새 페이지가 표시되지만 각 페이지 사이에 Loading Indicator가 표시됩니다.
    - UX를 생각하면 다음 페이지를 미리 가져오는 것이 좋습니다.

```jsx
const { data, isError, error, isLoading } = useQuery(
    ["posts", currentPage], 
    () => fetchPosts(currentPage), 
    {
      staleTime: 2000
    }
  );
```

### Prefetching

- Prefetch
    - 캐시를 위해 데이터를 가져온다.
    - 기본적으로 데이터는 가져오는 즉시 stale상태가 됩니다.
    - refetching 하는 동안 캐시를 이용해서 데이터를 표시할 수 있습니다.
        - 물론 캐시가 만료되지 않은 경우
- Prefetch는 Pagination 뿐 아니라 사용자가 미래에 필요로 할 것이라고 생각하는 모든 데이터를 가져오는데 적용할 수 있습니다.
- prefetchQuery는 queryClient의 메서드입니다. 이를 사용하기 위해 useQueryClient hook을 이용해서 queryClient를 가져온 후 이를 이용해서 prefetchQuery를 실행합니다.
- 이전 페이지로 가더라도 data를 유지하고 있도록 `keepPreviousData: true`를 설정합니다.

사용 예제)

```jsx
useEffect(() => {
    const nextPage = currentPage + 1;
		queryClient.prefetchQuery(['posts', nextPage], () => fetchPosts(nextPage));
  }, [currentPage, queryClient]);
```

ref) [https://react-query.tanstack.com/reference/QueryClient#queryclientprefetchquery](https://react-query.tanstack.com/reference/QueryClient#queryclientprefetchquery)

### isFetching vs isLoading

- isfetching
    - 비동기 쿼리 함수(API 등)가 아직 resolve 되지 않았을 때
- isLoading
    - 캐시된 데이터가 없으면서 fetching중 일 때

![Untitled](Section%202%20eb086c3af7cb4fd4bafada162ac453d4/Untitled.png)

### Mutations

- mutations는 server에 데이터를 업데이트 하는 네트워크 요청을 만듭니다.
- React Query를 사용하면 서버 호출이 성공했을거라고 가정하고 업데이트 했다가 에러가 발생하면 롤백하는 기능도 제공합니다.
- 성공한 후 쿼리를 무효화시킬 수 있습니다. 그러면 서버에서 refetch가 일어나서 데이터를 최신 상태로 유지합니다.

### useMutation

- useQuery와 비슷하지만 mutate함수를 반환합니다.
- query key를 필요로 하지 않습니다.
- isLoading은 있지만 isFetching은 존재하지 않습니다.
- 기본적으로 실패 시 재시도 하지 않습니다. 물론 재시도도 가능합니다.
- useMutation은 mutate를 호출할 때 인수를 받을 수 있습니다.

[https://react-query.tanstack.com/guides/mutations](https://react-query.tanstack.com/guides/mutations)

### Summary

- useQuery
    - 데이터를 위해 사용되고, isLoading, isFetching, error를 포함하는 object를 반환
- staleTime : 쿼리 데이터를 새로 고쳐야하는지 여부를 결정
- cacheTime: inactive 후에 데이터를 보유하고 싶은 시간
- Query key를 배열로도 관리할 수 있다.
- Pagination, Pre-Fetching에 대해 배웠다.
- Query 상태
    - fetching - 요청 중인 쿼리
    - fresh - 만료되지 않은 쿼리. 컴포넌트가 마운트, 업데이트되어도 데이터를 다시 요청하지 않는다
    - stale - 만료된 쿼리. 컴포넌트가 마운트, 업데이트되면 데이터를 다시 요청한다.
    - inactive - 사용하지 않는 쿼리. 일정 시간이 지나면 가비지 컬렉터가 캐시에서 제거한다
    - delete - 가비지 컬렉터에 의해 캐시에서 제거된 쿼리