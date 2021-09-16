# react-query-reference

## Dev Tools

```jsx
import { ReactQueryDevtools } from "react-query/devtools";

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* The rest of your application */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

## Basic

```jsx
import { QueryClient, QueryClientProvider, useQuery } from "react-query";

const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}

function Example() {
  const { isLoading, error, data } = useQuery("repoData", () =>
    fetch("https://api.github.com/repos/tannerlinsley/react-query").then(
      (res) => res.json()
    )
  );

  if (isLoading) return "Loading...";

  if (error) return "An error has occurred: " + error.message;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>👀 {data.subscribers_count}</strong>{" "}
      <strong>✨ {data.stargazers_count}</strong>{" "}
      <strong>🍴 {data.forks_count}</strong>
    </div>
  );
}
```

## Movie Lists

```jsx
import { useQueryClient, useQuery, useMutation } from "react-query";
function MoviesList() {
  // Access the client
  const queryClient = useQueryClient();

  // Queries
  const query = useQuery("movies", getMovies);

  // Mutations
  const mutation = useMutation(postMovie, {
    onSuccess: () => {
      queryClient.invalidateQueries("movies");
    },
  });

  return (
    <div>
      <ul>
        {query.data.map((movie) => (
          <li key={movie.id}> {movie.title} </li>
        ))}
      </ul>
      <button
        onClick={() => {
          mutation.mutate({
            id: Date.now(),
            title: "Terminator",
          });
        }}
      >
        Add Movie
      </button>
    </div>
  );
}
```

## No State

```jsx
// ###############
// I don't store return data from query to react state. I just feel like that's an anti-pattern. You should just use the returned data as it is like below:
const { loading, data, error } = useQuery(SOME_QUERY);

// If you absolutely need to cache the mutated data you can do the below. But
// most of the time you won't need to use useMemo at all.
const cachedMutatedData = useMemo(() => {
  if (loading || error) return null;

  // mutate data here
  return data;
}, [loading, error, data]);

if (loading) return <div>Loader</div>;
if (error) return <div>Error</div>;

// safe to assume data now exist and you can use data.
const mutatedData = (() => {
  // if you want to mutate the data for some reason
  return data;
})();

return <YourComponent data={mutatedData} />;

// ###############
// But if you must store the returned data, could do below:
const { loading, data, error } = useQuery(SOME_QUERY);
const [state, setState] = React.useState([]);

React.useEffect(() => {
  // do some checking here to ensure data exist
  if (data) {
    // mutate data if you need to
    setState(data);
  }
}, [data]);
```

## Displaying Global Background Fetching Loading State

In addition to individual query loading states, if you would like to show a global loading indicator when any queries are fetching (including in the background), you can use the useIsFetching hook:

```jsx
import { useIsFetching } from "react-query";

function GlobalLoadingIndicator() {
  const isFetching = useIsFetching();

  return isFetching ? (
    <div>Queries are fetching in the background...</div>
  ) : null;
}
```

## Pagination

```jsx
import { useInfiniteQuery } from "react-query";

const { data, error, fetchNextPage, hasNextPage, isFetchingNextPage, status } =
  useInfiniteQuery("movies", getPopularMovies, {
    getNextPageParam: (lastPage) =>
      lastPage.page < lastPage.total_pages ? lastPage.page + 1 : undefined,
  });
```
