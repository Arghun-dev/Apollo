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

Profile.js

```js
import { useQuery } from '@apollo/client'; 

export const ME_QUERY = gql`
  query me {
    me {
      id
      profile {
        id
        bio
        location
        website
        avatar
      }
    }
  }
`

function Profile() {
  const { loading, error, data } = useQuery(ME_QUERY);
  
  if (loading) return <div>loading</div>;
  
  if (error) return <div>{error.message}</div>;
  
  return (
   <div>
     // render data here
   </div>
  )
}
```


### Create Profile Component

CreateProfile.tsx

```js
import { gql, useMutation } from '@apollo/client';
import { Form, Field, Formik, ErrorMessage } from 'formik';

const CREATE_PROFILE_MUTATION = gql`
  muutation createProfile($bio: String!, $location: String, $website: String, avatar: String) {
     createProfile(bio: $bio, location: $location, website: $website, avatar: $avatar) {
       id
     }
  }
`

interface ProfileValues {
   bio: string;
   location: string;
   website: string;
   avatar: string;
}

const CreateProfile = () => {
  // here when you call `createProfile` after that `refetchQueries` will be called and `refetchQueries` here says that call `ME_QUERY`
  const [createProfile] = useMutation(CREATE_PROFILE_MUTATION, {
     refetchQueries: [{ query: ME_QUERY }]
  });
  
  const initialValues: ProfileValues = {
    bio: '',
    location: '',
    website: '',
    avatar: ''
  } 
  
  const submit = async (values, { setSubmitting }) => {
     setSubmitting(true);
     await createProfile({ variables: values });
     setSubmitting(false);
  }
  
  return (
     <Formik
       initialValues={initialValues}
       onSubmit={submit}
       validationSchema={validationSchema}
     >
        <Form>
          <Field name="bio" type="text" as="textArea" placeholder="Bio" />
          <ErrorMessage name="bio" component={'div'} />
          <Field name="location" type="location" placeholder="Location" />
          <ErrorMessage name="location" component={'div'} />
          <Field name="Website" type="website" placeholder="website" />
          <ErrorMessage name="Website" component={'div'} />
          <button type="submit">Submit</button>
       </Form>
     </Formik>
  )
}
```

## Update Profile

```js
const UPDATE_PROFILE_MUTATION = gql`
  mutation updateProfile($id: Int!, $bio: String, $location: String, $website: String, $avatar: String) {
     updateProfile(id: $id, bio: $bio, location: $location, website: $website, avatar: $avatar) {
       id
     }
  }
`
```

**If you notice in GraphQL we don't say put post delete get and many other stuff, we just call the method name which has been assigned in backend, for example `createprofile`, `updateProfile`**


## upload profile

```js
const inputFile = useRef(null);
const [image, setImage] = useState("");
const [imageLoading, setImageLoading] = useState(false);

const uploadImage = async(e) => {
  const files = e.target.files;
  const data = new FormData();
  data.append('file', files[0]);
  setImageLoading(true);
  const res = await fetch(process.env.END_POINT, {
    method: 'POST',
    body: data
  })
  const file = await res.json();
  setImage(file);
  setImageLoading(false);
}

<input
  type="file"
  name="file"
  onChange={uploadImage}
  ref={inputFile}
  style={{ display: "none" }}
/>
{imageLoading ? <div>Loading....</div> : (
 <>
   <span onClick={() => inputFile.current.click()}>
     <img
       src={data.me.Profile.avatar}
     />
   </span>
 </>
)}
```

Tweet.tsx

```js
import { gql, useMutation } from '@apollo/client';

const CREATE_TWEET_MUTATION = gql`
  mutation createTweet($content: !String) {
    createTweet(content: $content) {
      id
    }
  }
`

const Tweet = () => {
  const [createTweet] = useMutation(CREATE_TWEET_MUTATION, {
    refetchQueries: [{ query: ME_QUERY }]
  })
}
```
