# RedwoodJS Blog Tutorial Guide (v8.4)

## **Foreword**  
(No commands; overview of objectives.)

---

## **Chapter 0: What is Redwood?**  
(No commands; high-level RedwoodJS architecture.)

---

## **Chapter 1: Getting Started**

### **Prerequisites**

```bash
node --version     # Node.js (>=14.15.0)
yarn --version     # Yarn
git --version      # Git
```

### **Installation & Starting Development**

```bash
yarn create redwood-app redwoodblog
cd redwoodblog
yarn install
yarn redwood dev
```

### **Redwood File Structure**  
(No commands; explains /api, /web, /scripts.)

### **Our First Page**

```bash
yarn rw generate page Home /
```

**web/src/pages/HomePage/HomePage.js**  
```jsx
import { Link, routes } from '@redwoodjs/router'

const HomePage = () => (
  <main>
    <h1>Welcome to RedwoodJS Blog</h1>
    <Link to={routes.about()}>Learn more about us</Link>
  </main>
)

export default HomePage
```

### **A Second Page and a Link**

```bash
yarn rw generate page About /about
```

**web/src/Routes.js**  
```jsx
import { Router, Route } from '@redwoodjs/router'
import HomePage from 'src/pages/HomePage'
import AboutPage from 'src/pages/AboutPage'

const Routes = () => (
  <Router>
    <Route path="/" page={HomePage} name="home" />
    <Route path="/about" page={AboutPage} name="about" />
  </Router>
)

export default Routes
```

### **Layouts**

```bash
yarn rw generate layout MainLayout
```

**web/src/pages/HomePage/HomePage.js**  
```jsx
import MainLayout from 'src/layouts/MainLayout/MainLayout'
import { Link, routes } from '@redwoodjs/router'

const HomePage = () => (
  <MainLayout>
    <h1>Welcome to RedwoodJS Blog</h1>
    <Link to={routes.about()}>About</Link>
  </MainLayout>
)

export default HomePage
```

---

## **Chapter 2: Getting Dynamic**

### **Cells**

```bash
yarn rw generate cell PostsCell
```

**web/src/components/PostsCell/PostsCell.js**  
```jsx
import gql from 'graphql-tag'

export const QUERY = gql`
  query PostsQuery {
    posts {
      id
      title
      content
    }
  }
`

export const Loading = () => <div>Loading posts...</div>
export const Empty = () => <div>No posts found.</div>
export const Failure = ({ error }) => <div>Error: {error.message}</div>
export const Success = ({ posts }) => (
  <ul>
    {posts.map((post) => (
      <li key={post.id}>
        <h2>{post.title}</h2>
        <p>{post.content}</p>
      </li>
    ))}
  </ul>
)
```

### **Side Quest: How Redwood Works with Data**  
(No commands; explains Prisma + GraphQL.)

### **Routing Params**

**web/src/Routes.js**  
```jsx
<Route path="/post/{id:Int}" page={PostPage} name="post" />
```

**web/src/pages/PostPage/PostPage.js**  
```jsx
const PostPage = ({ id }) => <div>Post ID: {id}</div>
export default PostPage
```

---

## **Chapter 3: Saving Data**

### **Building a Form**

**web/src/components/PostForm/PostForm.js**  
```jsx
import { useForm } from 'react-hook-form'
import { useMutation } from '@redwoodjs/web'
import { toast } from '@redwoodjs/web/toast'
import gql from 'graphql-tag'

const CREATE_POST_MUTATION = gql`
  mutation CreatePostMutation($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      content
    }
  }
`

const PostForm = () => {
  const { register, handleSubmit, errors } = useForm()
  const [createPost] = useMutation(CREATE_POST_MUTATION, {
    onCompleted: () => toast.success('Post created successfully!'),
    onError: (error) => toast.error(error.message),
  })

  const onSubmit = (data) => createPost({ variables: { input: data } })

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="title">Title:</label>
        <input name="title" ref={register({ required: true })} />
        {errors.title && <span>Title is required.</span>}
      </div>
      <div>
        <label htmlFor="content">Content:</label>
        <textarea name="content" ref={register({ required: true })} />
        {errors.content && <span>Content is required.</span>}
      </div>
      <button type="submit">Create Post</button>
    </form>
  )
}

export default PostForm
```

### **Saving Data**

**api/prisma/schema.prisma**  
```prisma
datasource DS {
  provider = "sqlite"
  url      = "file:./dev.db"
}

generator client {
  provider = "prisma-client-js"
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  createdAt DateTime @default(now())
}

model Comment {
  id        Int      @id @default(autoincrement())
  postId    Int
  content   String
  post      Post     @relation(fields: [postId], references: [id])
  createdAt DateTime @default(now())
}
```

```bash
yarn rw prisma migrate dev --name init
```

**api/src/services/posts/posts.js**  
```js
import { db } from 'src/lib/db'

export const posts = () => db.post.findMany()

export const createPost = ({ input }) => {
  return db.post.create({
    data: input,
  })
}
```

---

## **Chapter 4: Authentication**

```bash
yarn rw generate auth netlify
```

**web/src/components/NavBar/NavBar.js**  
```jsx
import { useAuth } from '@redwoodjs/auth'
import { Link } from '@redwoodjs/router'

const NavBar = () => {
  const { isAuthenticated, logIn, logOut } = useAuth()
  return (
    <nav>
      <Link to="/">Home</Link>
      {isAuthenticated ? (
        <button onClick={logOut}>Logout</button>
      ) : (
        <button onClick={logIn}>Login</button>
      )}
    </nav>
  )
}

export default NavBar
```

### **Deployment**

```bash
yarn rw build
```
(Deploy to Netlify, Vercel, or another host.)

---

## **Intermission**  
(No commands; recap and overview.)

---

## **Chapter 5: Frontend Magic**

### **Introduction to Storybook**

```bash
yarn rw storybook
```

### **Our First Story**

**web/src/components/MyComponent/MyComponent.stories.js**  
```jsx
import MyComponent from './MyComponent'

export default {
  title: 'Components/MyComponent',
  component: MyComponent,
}

export const Default = () => <MyComponent text="Hello Redwood!" />
```

### **Introduction to Testing**  
(No commands; overview of Jest.)

### **Our First Test**

**web/src/pages/HomePage/HomePage.test.js**  
```jsx
import { render, screen } from '@redwoodjs/testing/web'
import HomePage from './HomePage'

describe('HomePage', () => {
  it('renders the welcome message', () => {
    render(<HomePage />)
    expect(screen.getByText('Welcome to RedwoodJS Blog')).toBeInTheDocument()
  })
})
```

```bash
yarn rw test
```

---

## **Chapter 6: Building a Component the Redwood Way**

### **Multiple Comments**  
(No commands; adding multiple comments logic.)

### **Adding Comments to the Schema**

```bash
yarn rw prisma migrate dev --name add_comments
```

### **Creating a Comment Form**

**web/src/components/CommentForm/CommentForm.js**  
```jsx
import { useForm } from 'react-hook-form'
import { useMutation } from '@redwoodjs/web'
import { toast } from '@redwoodjs/web/toast'
import gql from 'graphql-tag'

const CREATE_COMMENT_MUTATION = gql`
  mutation CreateCommentMutation($input: CreateCommentInput!) {
    createComment(input: $input) {
      id
      content
    }
  }
`

const CommentForm = ({ postId }) => {
  const { register, handleSubmit, errors } = useForm()
  const [createComment] = useMutation(CREATE_COMMENT_MUTATION, {
    onCompleted: () => toast.success('Comment added!'),
    onError: (error) => toast.error(error.message),
  })

  const onSubmit = (data) => {
    createComment({ variables: { input: { ...data, postId } } })
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <label>Comment:
        <textarea name="content" ref={register({ required: true })} />
      </label>
      {errors.content && <span>Required</span>}
      <button type="submit">Submit Comment</button>
    </form>
  )
}

export default CommentForm
```

---

## **Chapter 7: Role-Based Access Control (RBAC)**

### **Securing the App**

**api/src/lib/auth.js**  
```js
export const requireAuth = (user) => {
  if (!user) {
    throw new Error('User must be authenticated')
  }
  return user
}
```

Use it in services:
```js
import { requireAuth } from 'src/lib/auth'
import { db } from 'src/lib/db'

export const createPost = ({ input }, { context }) => {
  requireAuth(context.currentUser)
  return db.post.create({
    data: input,
  })
}
```

### **Accessing currentUser in the API side**

```js
export const getCurrentUser = (decoded) => decoded
```

---

## **Afterword**  
(No commands; final thoughts and next steps.)

---

**This is the RedwoodJS v8.4 tutorial structure** (Chapters 0–7, Intermission, Afterword) **with all code, commands, and file references, minus extra narrative.**
