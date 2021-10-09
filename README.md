# Apollo

### Setup1

```js
import { ApolloClient, ApolloProvider, InMemoryCache } from '@apollo/client';

const client = new ApolloClient({
  uri: // the url I want to request,
  cache: new InMemoryCache()
})

const App = () => {
  <ApolloProvider client={client}>
    // ...
  </ApolloProvider>
}
```

this is the first step setup, now you can make request to get the data from the server.



### Handle Authentication

`$. npm i apollo-link-context`

```js
import { ApolloClient, ApolloProvider, InMemoryCache, HttpLink } from '@apollo/client';
import { setContext } from 'apollo-link-context';

const httpLink = new HttpLink({ uri: // the url you want to request });
const authLink = setContext(async(req, {headers}) => {
  const token = localStorage.getItem('token');
  
  return {
    ...headers,
    headers: {
      Authorization: token ? `Bearer ${token}` : null
    }
  }
});

const link = authLink.concat(httpLink as any);
const client = new ApolloClient({
  link: (link as any),
  cache: new InMemoryCache()
})

const App = () => {
  <ApolloProvider client={client}>
    // ...
  </ApolloProvider>
}
```

### SignUp Page

`$. npm i formik yup`

`$. npm i -D @types/yup`

these packages are awesome, because the will help us in form validation

```js
import { gql, useMutation } from '@apollo/client';


```
