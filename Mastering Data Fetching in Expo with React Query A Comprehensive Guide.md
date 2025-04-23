# Mastering Data Fetching in Expo with React Query: A Comprehensive Guide

Building mobile applications with Expo and React Native is exciting, but efficiently managing data fetching, caching, and state updates can quickly become complex.  This is where React Query shines. It's a powerful library that simplifies these tasks, allowing you to focus on building a fantastic user experience.  This guide will walk you through using React Query with Expo, covering setup, common use cases, and best practices.

Ready to supercharge your Expo apps with efficient data fetching?  **Grab this course for free** and dive deep into React Query with Expo: [https://udemywork.com/react-query-with-expo](https://udemywork.com/react-query-with-expo)

## Why React Query with Expo?

Expo, a framework built around React Native, streamlines mobile development, providing a seamless experience for building and deploying cross-platform applications. However, handling data fetching, especially when dealing with remote APIs, presents challenges such as:

*   **Managing Loading States:**  Displaying loading indicators while data is being fetched to provide a smooth user experience.
*   **Caching:** Storing fetched data to avoid unnecessary network requests and improve performance.
*   **Error Handling:** Gracefully handling API errors and providing informative messages to the user.
*   **Data Synchronization:**  Keeping the UI in sync with the latest data, even when it changes on the server.
*   **Optimistic Updates:**  Updating the UI optimistically before the server confirms the changes, providing a more responsive feel.

React Query addresses these challenges head-on, offering a declarative and efficient way to manage server state in your Expo applications. It takes care of caching, background updates, retries, and much more, reducing boilerplate code and making your application more maintainable.

## Setting up React Query in your Expo Project

First, you'll need to install React Query:

```bash
npx expo install @tanstack/react-query
```

Next, wrap your root component (likely `App.js` or `App.tsx`) with the `QueryClientProvider`:

```jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import AppRoot from './AppRoot'; // Replace with your actual root component

const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <AppRoot />
    </QueryClientProvider>
  );
}
```

This sets up a global `QueryClient` instance, which manages the cache and provides access to React Query's functionality throughout your application. It also provides sensible defaults that you can override.

## Basic Data Fetching with `useQuery`

The `useQuery` hook is the core of React Query. It's used to fetch data and manage its state. Let's illustrate with a simple example of fetching a list of posts from a hypothetical API endpoint:

```jsx
import { useQuery } from '@tanstack/react-query';
import { View, Text, FlatList, ActivityIndicator } from 'react-native';

const fetchPosts = async () => {
  const response = await fetch('https://your-api.com/posts');
  if (!response.ok) {
    throw new Error('Network response was not ok');
  }
  return response.json();
};

const PostsList = () => {
  const { isLoading, error, data } = useQuery('posts', fetchPosts);

  if (isLoading) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  if (error) {
    return <Text>Error: {error.message}</Text>;
  }

  return (
    <FlatList
      data={data}
      keyExtractor={(item) => item.id.toString()}
      renderItem={({ item }) => (
        <View style={{ padding: 10, borderBottomWidth: 1, borderBottomColor: '#ccc' }}>
          <Text style={{ fontWeight: 'bold' }}>{item.title}</Text>
          <Text>{item.body}</Text>
        </View>
      )}
    />
  );
};

export default PostsList;
```

In this example:

1.  We define an `async` function `fetchPosts` to make the API request using `fetch`. Error handling is crucial here.
2.  We use `useQuery` with two arguments:
    *   A unique `queryKey` ('posts' in this case). This key is used for caching and identifying the query.
    *   The `fetchPosts` function to fetch the data.
3.  `useQuery` returns an object with properties like `isLoading`, `error`, and `data`. We use these to render the appropriate UI based on the query state.  Loading indicators and error messages enhance user experience.

## Mutations with `useMutation`

`useMutation` is used for performing actions that modify data on the server (e.g., creating, updating, or deleting data). Let's create a component to add a new post:

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { View, Text, TextInput, Button, Alert } from 'react-native';
import { useState } from 'react';

const createPost = async (newPost) => {
  const response = await fetch('https://your-api.com/posts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newPost),
  });
  if (!response.ok) {
    throw new Error('Failed to create post');
  }
  return response.json();
};

const AddPostForm = () => {
  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');
  const queryClient = useQueryClient();

  const mutation = useMutation(createPost, {
    onSuccess: () => {
      // Invalidate the 'posts' query to refetch data
      queryClient.invalidateQueries('posts');
      // Optionally, show a success message
      Alert.alert('Success', 'Post created successfully!');
      // Clear the form
      setTitle('');
      setBody('');
    },
    onError: (error) => {
      Alert.alert('Error', error.message);
    },
  });

  const handleSubmit = () => {
    mutation.mutate({ title, body });
  };

  return (
    <View style={{ padding: 10 }}>
      <Text style={{ fontWeight: 'bold', marginBottom: 5 }}>Add New Post</Text>
      <TextInput
        placeholder="Title"
        value={title}
        onChangeText={setTitle}
        style={{ borderWidth: 1, borderColor: '#ccc', padding: 8, marginBottom: 10 }}
      />
      <TextInput
        placeholder="Body"
        value={body}
        onChangeText={setBody}
        multiline
        style={{ borderWidth: 1, borderColor: '#ccc', padding: 8, marginBottom: 10 }}
      />
      <Button title="Create Post" onPress={handleSubmit} disabled={mutation.isLoading} />
      {mutation.isLoading && <Text>Creating post...</Text>}
    </View>
  );
};

export default AddPostForm;
```

Key points:

1.  We define an `async` function `createPost` that makes a POST request to our API.
2.  We use `useMutation` with the `createPost` function.
3.  The `onSuccess` callback is triggered when the mutation is successful. We use `queryClient.invalidateQueries('posts')` to refetch the 'posts' query, ensuring our list is updated with the new post.  This keeps your application consistent with server-side data.  A success message provides valuable feedback to the user.
4.  The `onError` callback handles errors during the mutation.
5.  We disable the button while the mutation is loading to prevent multiple submissions.

## Optimistic Updates

Optimistic updates enhance the user experience by updating the UI immediately as if the mutation was successful, even before receiving confirmation from the server. If the mutation fails, we revert the changes. This is particularly useful for actions like liking a post or deleting an item. While this can be tricky, React Query has got you covered.

To really solidify your understanding of optimistic updates and beyond, **unlock your free access** to the complete React Query with Expo course here: [https://udemywork.com/react-query-with-expo](https://udemywork.com/react-query-with-expo)

## Prefetching

React Query allows you to prefetch data before it's actually needed. This can significantly improve the perceived performance of your application.  For example, you could prefetch data when the user hovers over a link or navigates to a screen.

```jsx
import { useQueryClient } from '@tanstack/react-query';
import { Button } from 'react-native';

const PrefetchButton = () => {
  const queryClient = useQueryClient();

  const handlePrefetch = () => {
    queryClient.prefetchQuery('posts', fetchPosts);
  };

  return <Button title="Prefetch Posts" onPress={handlePrefetch} />;
};

export default PrefetchButton;
```

This will fetch the data in the background and cache it.  When the user actually navigates to the `PostsList` component, the data will be available immediately.

## Key Considerations and Best Practices

*   **Query Keys:** Use meaningful and unique query keys.  For dynamic queries, include the necessary parameters in the key (e.g., `['posts', { userId: 123 }]`).
*   **Stale Time:** Configure the `staleTime` option to control how long data is considered fresh.  This helps balance performance and data freshness.
*   **Refetch Intervals:** Use `refetchInterval` to automatically refetch data at regular intervals, keeping your UI up-to-date.  Be mindful of your API usage.
*   **Error Boundaries:** Wrap your components with error boundaries to catch and handle errors gracefully.
*   **Pagination and Infinite Scrolling:** React Query provides utilities for handling pagination and infinite scrolling efficiently.
*   **Custom Hooks:**  Create custom hooks to encapsulate your data fetching logic and make your code more reusable and maintainable.

## Beyond the Basics

React Query offers a wide range of advanced features, including:

*   **Dependent Queries:**  Fetching data based on the result of another query.
*   **Parallel Queries:**  Fetching multiple queries in parallel for improved performance.
*   **Retry Strategies:**  Configuring how React Query retries failed requests.
*   **Persisting the Cache:**  Persisting the cache to local storage to improve offline support and initial load times.

## Conclusion

React Query is a game-changer for managing data fetching in Expo applications. It simplifies complex tasks, improves performance, and enhances the user experience. By embracing React Query, you can focus on building compelling mobile apps with confidence.

Ready to elevate your Expo development skills? This comprehensive course covers all of these topics and more in detail. **Download it now for free**: [https://udemywork.com/react-query-with-expo](https://udemywork.com/react-query-with-expo)
