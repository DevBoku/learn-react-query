# Section 1.

# Client State vs Server State

- Client State
    - 웹 브라우저 세션과 관련된 모든 정보를 의미
        - 언어, 배경(light, dark)
        - 사용자의 상태
- Server State
    - 서버에 저장되지만 클라이언트에 표시하는데 필요한 정보
        - ex) 데이터베이스에 저장하고 있는 게시물 데이터

# What problem does React Query Solve?

- 서버에서 데이터를 가져올 때 React Query를 거쳐서 가져옵니다.
- React Query는 데이터를 관리하고 캐시를 업데이트 하고 표시하는 것은 개발자의 몫
- React Query는 데이터를 Key를 이용해서 가져오고 새로운 데이터를 가져오고 싶을 떄는 이를 무효화시킬 수 있다.

# Plus...

- React Query는 서버에서 데이터를 가져올 때 로드 및 오류 상태를 관리합니다.
- Pagination 또는 infinite scroll을 가져올 수 있는 도구를 제공합니다.
- 데이터를 Prefetching해서 캐시에 넣을 수 있습니다.
- 서버 데이터를 업데이트 하기 위한 mutation도 가능합니다.
- 같은 요청을 여러번 요청하지 않고 하나로 관리할 수 있습니다.
- 에러가 났을 때 재시도하는 기능도 제공하고 있습니다.

First Project! Blog-em Ipsum

- Gets Data from [https://jsonplaceholder.typicode.com/](https://jsonplaceholder.typicode.com/)
    - 블로그 서버 API 제공
    - 업데이트도 가능 (그러나 실제로 서버 데이터를 변경하지는 않음)
- React Query dev tools 적용
- Pagination 적용
- Prefetching
    - 다음 페이지를 미리 가져오는 기능 추가
- Mutations
    - 서버의 데이터를 변경하는법 확인

### 1. Query Client, Query Client Provider 추가

```jsx
import { Posts } from "./Posts";
import "./App.css";

import { QueryClient, QueryClientProvider } from 'react-query';
const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Blog Posts</h1>
        <Posts />
      </div>
    </QueryClientProvider>
  );
}

export default App;
```

### 2. useQuery 사용

```jsx
export function Posts() {
  const [currentPage, setCurrentPage] = useState(0);
  const [selectedPost, setSelectedPost] = useState(null);

  // replace with useQuery
  const { data } = useQuery("posts", fetchPosts);
  if (!data) return <div />

  return (
    <>
      <ul>
        {data.map((post) => (
          <li
            key={post.id}
            className="post-title"
            onClick={() => setSelectedPost(post)}
          >
            {post.title}
          </li>
        ))}
      </ul>
		</>
	)
}
```

- 파라미터
    - 첫 번째 파라미터 : Query Key
    - 두 번쨰 파라미터 : Promise 함수
- fetch 전까지 data는 undefined이다.
- ref)
    - [https://react-query.tanstack.com/reference/useQuery](https://react-query.tanstack.com/reference/useQuery)

### 3. loading indicator를 위한 정보

- isLoading, isError를 사용하면 우리가 데이터를 로드하는 지 여부와 에러가 발생했는지 여부를 확인할 수 있다.
- isLoading vs isFetching
    - isFetching
        - 비동기 쿼리가 resolve되지 않았으며 이 경우 가져오기가 완료되지 않았음을 의미. 하지만 캐시된 데이터는 가져올 수 있음
    - isLoading
        - 아직 해결되지 않았지만 캐시된 데이터도 없을 때, 표시할 데이터가 없음
- React Query는 default로 retry를 3번 시도함
- error를 가져오면 throw된 실제 error객체를 가져올 수 있음

```jsx
const { data, isError, error, isLoading } = useQuery("posts", fetchPosts);
if (isLoading) return <h3>Loading...</h3>
if (isError) return <h3>Oops, something went wrong {error.toString()}</h3>
```

### React Query Dev Tools

- Query key로 볼 수 있음
    - status of queris
    - last updated timestamp
- Data explorer
- Query explorer
- React dev tools는 기본으로 포함되어있지 않음. production에서는 react query가 표시되지 않는다.
    - [https://react-query.tanstack.com/devtools](https://react-query.tanstack.com/devtools)

```jsx
import { Posts } from "./Posts";
import "./App.css";

import { QueryClient, QueryClientProvider } from 'react-query';
import { ReactQueryDevtools } from 'react-query/devtools';

const queryClient = new QueryClient();

function App() {
  return (
    // provide React Query client to App
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Blog Posts</h1>
        <Posts />
      </div>
      <ReactQueryDevtools />
    </QueryClientProvider>
  );
}
```

# Stale time vs Cache time

### Stale Data

- 데이터가 stale(부실한)하다는 것이 무엇을 의미하고 왜 중요합니까?
- Data는 오로지 stale data만 refetch해옵니다.
    - 예를 들어 리렌더링 된 컴포넌트나 window가 다시 focus됐을 때
    - staleTime은 데이터가 얼마나 오래동안 있을 것인지의 max age를 나타냄
- 블로그 게시물의 경우 10초 정도 오래된 데이터는 기꺼이 허용할 것! (가정임)

```jsx
const { data, isError, isLoading } = useQuery("posts", fetchPosts, { staleTime: 2000 });
```

- StaleTime의 기본값이 0인 이유는?
    - 실수로 클라이언트에 오래된 데이터가 있을 가능성이 훨씬 줄어든다.
    - ref) [https://twitter.com/tannerlinsley/status/I1385258144549330952](https://twitter.com/tannerlinsley/status/1385258144549330952)

### Stale time vs Cache Time

- staleTime은 re-fetching을 위한 고려 사항입니다.
- Cache는 나중에 재사용될 데이터입니다.
    - 따라서 특정 query에 active한 useQuery가 없는 경우 cold storage로 이동
    - 캐시 데이터는 cacheTime이 지난 이후에 만료됨 (기본값: 5분)
        - 마지막으로 active한 useQuery이후 얼마나 지났는지
        - 페이지에 표시되는 구성 요소가 이 특정 항목에 대해 사용된 쿼리를 사용하고 있는 기간
        - staleTime은 기본값 0분
    - Cache가 만료되면 서버의 가장 최근 데이터로 새로 고칠 수 있도록 데이터 가져오기를 중지하지는 않습니다. 새 데이터를 가져오는 동안 이전에 가져온 Cache 데이터를 표시할 수 있습니다.