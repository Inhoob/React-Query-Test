# React-Query 연습

## Create Query Client

- 쿼리와 서버의 데이터 캐시를 관리하는 클라이언트

## useQuery

- 서버에서 데이터를 가져오는 훅

```javascript
const { data, isLoading, isError } = useQuery(["posts"], fetchPosts);
```

Query는 queryKey와 비동기함수로 구성된다.

## isFetching vs isLoading

isFetching : async query function not resolved
isLoading : no cached data. 즉 fetching 중이면서 cached data도 없어서 보여줄 것이 없을때 사용.
즉 화면에 아무것도 보여줄 게 없을 땐 isLoading이 더 적합함.

## Stale Data(만료된 데이터)

- Data refetch의 trigger = component remount, window refocus. 하지만 stale data에만 실행됨.
- staleTime => 웹사이트에 표시된 데이터가 10초까지는 괜찮다면 staleTime를 10초로 설정해도 된다.

```javascript
const { data, isError, isLoading } = useQuery(["posts"], fetchPosts, {
  staleTime: 2000,
});
```

staleTime의 기본값은 0ms이다. 항상 최신데이터로 업데이트하는것이 일정시간마다 업데이트되는것보단 낫기 때문.

## Cache Data

- 캐시는 나중에 사용될 수도 있는 데이터를 의미한다.
- useQuery가 활성화되어있지 않으면 그것은 'cold storage'로 가는데 이 상태에서 cacheTime 이후에 데이터는
  가비지 콜렉팅이 된다. 데이터가 캐시에 있는 동안에는 Fetching시 사용될 수 있다.
  캐시 타임은 기본이 5 minutes 로 설정되어 있는데 이것은 새로운 fetching이 되는동안 빈 화면을 보는 것보다는
  과거의 화면을 보는것이 더 낫기 때문.

## Why don't comments refresh?

```javascript
const { data, isError, isLoading, error } = useQuery(["comments"], () =>
  fetchComments(post.id)
);
```

- 모든 키가 같은 queryKey를 가지고있다.('comments')
- 알려진 쿼리키의 데이터는 오직 trigger가 있어야만 refetching이 일어난다.

이 경우 위의코드를 아래처럼 수정하면 된다.

```javascript
const { data, isError, isLoading, error } = useQuery(
  ["comments", post.id],
  () => fetchComments(post.id)
);
```

## 모든 refetching trigger 정리

- component remount
- window refocus
- running refetch function
- automated refetch
- query invalidation after a mutation

## prefetch란?

- adds data to cache
- automatically stale(configurable)
- shows while re-fetching(cache가 expired되지 않아야 하기 때문에 cacheTime을 유저가 페이지에 머무는 시간을 고려해서 설정해야 함)

```javascript
const queryClient = useQueryClient();
useEffect(() => {
  if (currentPage < maxPostPage) {
    const nextPage = currentPage + 1;
    queryClient.prefetchQuery(["posts", nextPage], () => fetchPosts(nextPage));
  }
}, [currentPage, queryClient]);
```

위의 코드와 같이 queryClient.prefetchQuery를 통해 다음 페이지의 내용을 미리 저장해서 로딩을 없앨 수 있다.

## useMutation

- mutate function을 return 한다.
- query key가 필요하지 않다. 이것은 query가 아니라 mutation이기 때문에.
- isFetching 이 없다.(isLoading만 존재)
- useQuery는 기본적으로 3회 재시도를 하지만 useMutation의 기본값은 1회 실패시 끝(설정 가능)

```javascript
async function deletePost(postId) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/postId/${postId}`,
    { method: "DELETE" }
  );
  return response.json();
}

const deleteMutation = useMutation((postId) => deletePost(postId));
```

그 이후에 JSX에서 deleteMutation.isError 등으로 사용할 수 있다. 구조분해할당을 하지 않아도
