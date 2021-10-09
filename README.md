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
import { ErrorMessage, Field, Form, Formik } from 'formik';
import * as Yup from 'yup';
import { useHistory } from 'react-router-dom'; 

const SIGNUP_MUTATION = gql`
  mutation signup($name: String, $email: String!, $password: String!) {
    signup(name: $name, email: $email, password: $password) {
      token
    }
  }
`

interface SignupValues {
  name: string;
  email: string;
  password: string;
  confirmPassword: string;
}

const Signup = () => {
   const history = useHistory();
   const [signup, {data}] = useMutation(SIGNUP_MUTATION);
   
   const submit = async (values, { setSubmitting }) => {
     setSubmitting(true);
     const response = await signup({
       variables: values
     })
     localStorage.setItem("token", response.data.signup.token);
     setSubmitting(false);
   }
   
   const validationSchema = Yup.object({
     email: Yup.string().email("Invalid email address").required(),
     name: Yup.string().max(20, "Must be 20 characters or less").min(5, "Must be at least 5 characters"),
     password: Yup.string().max(8, "Must be at least 8 characters"),
     confirmPassword: Yup.string().oneOf([Yup.ref("password"), null], "Passwords Must match")
   })
   
   const initialValues: SignupValues = {
    name: '',
    email: '',
    password: '',
    confirmPassword: ''
   }

   return (
     <Formik>
       initialValues={initialValues}
       validationSchema={validationSchema}
       onSubmit={submit}
     />
       <Form>
         <Field name="email" type="email" />
         <ErrorMessage name="email" component={'div'} />
         <Field name="name" type="text" />
         <ErrorMessage name="name" component={'div'} />
         <Field name="password" type="password" />
         <ErrorMessage name="poassword" component={'div'} />
         <Field name="confirmPassword" type="password" />
         <ErrorMessage name="confirmPassword" component={'div'} />
         <button type="submit">Signup</button>
       </Form>
     </Formik>
   )
}

export default Signup;
```

## Protected Routes

create a component called `IsAuthenticated.tsx`

```js
import React from 'react';
import { Redirect } from 'react-router-dom';

interface Props {
  children?: React.ReactNode;
}

const IsAuthenticated = ({ children }: Props) => {
  if (!localStorage.getItem("token")) <Redirect to={{ pathname: '/login' }} />;
  
  return <>{children}</>;
}
```

then wrap the Routes which you want to protected, with `IsAuthenticated`
