# RedwoodJS Blog Tutorial Guide (v8.4)

Welcome to the RedwoodJS Blog Tutorial! This guide will walk you through building a complete, full-stack blog application using RedwoodJS. By the end, you'll have a solid understanding of how Redwood works and the tools it provides.

## Table of Contents

1. [Foreword](#foreword)
2. [Chapter 0: What is Redwood?](#chapter-0-what-is-redwood)
3. [Chapter 1: Getting Started](#chapter-1-getting-started)
4. [Chapter 2: Getting Dynamic](#chapter-2-getting-dynamic)
5. [Chapter 3: Saving Data](#chapter-3-saving-data)
6. [Chapter 4: Authentication](#chapter-4-authentication)
7. [Intermission](#intermission)
8. [Chapter 5: Frontend Magic](#chapter-5-frontend-magic)
9. [Chapter 6: Building Components](#chapter-6-building-components)
10. [Chapter 7: Role-Based Access Control](#chapter-7-role-based-access-control)
11. [Afterword](#afterword)

## Foreword

RedwoodJS is a full-stack JavaScript framework designed to make web development more productive and enjoyable. It combines the best parts of React, GraphQL, and Prisma to provide a seamless development experience. In this tutorial, we'll build a blog application from scratch, learning the key concepts and features of RedwoodJS along the way.

## Chapter 0: What is Redwood?

RedwoodJS offers an integrated, full-stack framework that brings together several key technologies:

- **React** for the frontend UI
- **GraphQL** for data fetching
- **Prisma** for database access
- **Jest** for testing
- **Storybook** for component development

Key features that make Redwood special:

1. **Cell-based data fetching**: Automatic handling of loading, error, and empty states
2. **File-based routing**: Routes are defined by your file structure
3. **Services layer**: Clean separation of data access from UI components
4. **Generators**: Create pages, components, cells, and more with CLI commands
5. **Authentication**: Built-in support for various auth providers

Let's start building!

## Chapter 1: Getting Started

### Prerequisites

Make sure you have the following installed:

```bash
node --version     # Should be >=20.x
yarn --version     # Make sure Yarn is installed
git --version      # Make sure Git is installed
```

### Installation & Starting Development

Create a new Redwood app:

```bash
yarn create redwood-app redwoodblog
cd redwoodblog
yarn install
yarn redwood dev
```

Your browser should open to `http://localhost:8910` with the Redwood welcome page.

### Redwood File Structure

Let's explore the key directories in a Redwood app:

- **/api**: Backend code (GraphQL, database access)
  - `/db`: Database schema and migrations
  - `/src/services`: Business logic and data access
  - `/src/functions`: API endpoints (including GraphQL)
  
- **/web**: Frontend code (React)
  - `/src/pages`: Page components
  - `/src/components`: Reusable UI components
  - `/src/layouts`: Layout components that wrap pages
  
- **/scripts**: Utility scripts for tasks like seeding the database

### Our First Page

Let's update the homepage. First, let's generate a new Home page:

```bash
yarn rw generate page Home /
```

This creates a new file at `web/src/pages/HomePage/HomePage.tsx`. Let's update it:

```jsx
import { Metadata } from '@redwoodjs/web'
import { Link, routes } from '@redwoodjs/router'

const HomePage = () => {
  return (
    <>
      <Metadata title="Home" description="Home page" />
      <main>
        <h1>Welcome to RedwoodJS Blog</h1>
        <p>Find out more about RedwoodJS and what it can do for you!</p>
        <Link to={routes.about()}>Learn more about us</Link>
      </main>
    </>
  )
}

export default HomePage
```

### A Second Page and a Link

Now let's create an About page:

```bash
yarn rw generate page About /about
```

This creates a new file at `web/src/pages/AboutPage/AboutPage.tsx`. Let's update it:

```jsx
import { Metadata } from '@redwoodjs/web'

const AboutPage = () => {
  return (
    <>
      <Metadata title="About" description="About page" />
      <h1>About</h1>
      <p>
        This site was created to demonstrate my mastery of Redwood: Look on my
        works, ye mighty, and despair!
      </p>
    </>
  )
}

export default AboutPage
```

The routes in `web/src/Routes.tsx` should already be set up correctly:

```jsx
import { Router, Route, Set } from '@redwoodjs/router'
import BlogLayout from 'src/layouts/BlogLayout/BlogLayout'

const Routes = () => {
  return (
    <Router>
      <Set wrap={BlogLayout}>
        <Route path="/about" page={AboutPage} name="about" />
        <Route path="/" page={HomePage} name="home" />
      </Set>
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```

### Layouts

Now let's create a layout for our blog:

```bash
yarn rw generate layout Blog
```

This should already be created in your project as `web/src/layouts/BlogLayout/BlogLayout.tsx`. Let's make sure it has the navigation we need:

```jsx
import { Link, routes } from '@redwoodjs/router'

type BlogLayoutProps = {
  children?: React.ReactNode
}

const BlogLayout = ({ children }: BlogLayoutProps) => {
  return (
    <>
      <header>
        <h1>
          <Link to={routes.home()}>Redwood Blog</Link>
        </h1>
        <nav>
          <ul>
            <li>
              <Link to={routes.home()}>Home</Link>
            </li>
            <li>
              <Link to={routes.about()}>About</Link>
            </li>
          </ul>
        </nav>
      </header>
      <main>{children}</main>
    </>
  )
}

export default BlogLayout
```

## Chapter 2: Getting Dynamic

### Cells

Cells are a key concept in RedwoodJS. They handle the lifecycle of data fetching, including loading, error, and empty states.

Let's create a cell for fetching and displaying blog posts:

```bash
yarn rw generate cell Posts
```

This creates a new file at `web/src/components/PostsCell/PostsCell.tsx`. Let's update it:

```jsx
import type { PostsQuery } from 'types/graphql'
import type { CellSuccessProps, CellFailureProps } from '@redwoodjs/web'

export const QUERY = gql`
  query PostsQuery {
    posts {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Loading posts...</div>

export const Empty = () => <div>No posts yet. Create one!</div>

export const Failure = ({ error }: CellFailureProps) => (
  <div style={{ color: 'red' }}>Error: {error?.message}</div>
)

export const Success = ({ posts }: CellSuccessProps<PostsQuery>) => {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.body}</p>
          <div>Posted at: {new Date(post.createdAt).toLocaleString()}</div>
        </li>
      ))}
    </ul>
  )
}
```

### Side Quest: How Redwood Works with Data

Let's understand how data flows in a Redwood app:

1. **Prisma Schema**: Defines your database models in `api/db/schema.prisma`
2. **GraphQL SDL**: Defines the GraphQL schema in `api/src/graphql`
3. **Services**: Contains the business logic in `api/src/services`
4. **Cells**: Fetch data on the frontend using GraphQL

### Routing Params

Now let's create a page for viewing a single post:

```bash
yarn rw generate page Post "{id:Int}"
```

This creates a new file at `web/src/pages/PostPage/PostPage.tsx`. Let's update it:

```jsx
import { Metadata } from '@redwoodjs/web'
import PostCell from 'src/components/PostCell'

type PostPageProps = {
  id: number
}

const PostPage = ({ id }: PostPageProps) => {
  return (
    <>
      <Metadata title="Post" description="Individual post page" />
      <PostCell id={id} />
    </>
  )
}

export default PostPage
```

Then let's create a cell for fetching a single post:

```bash
yarn rw generate cell Post
```

And update it at `web/src/components/PostCell/PostCell.tsx`:

```jsx
import type { FindPostQuery, FindPostQueryVariables } from 'types/graphql'
import type { CellSuccessProps, CellFailureProps } from '@redwoodjs/web'

export const QUERY = gql`
  query FindPostQuery($id: Int!) {
    post: post(id: $id) {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Post not found</div>

export const Failure = ({ error }: CellFailureProps) => (
  <div style={{ color: 'red' }}>Error: {error?.message}</div>
)

export const Success = ({ post }: CellSuccessProps<FindPostQuery, FindPostQueryVariables>) => {
  return (
    <article>
      <header>
        <h2>{post.title}</h2>
      </header>
      <div>{post.body}</div>
      <div>Posted at: {new Date(post.createdAt).toLocaleString()}</div>
    </article>
  )
}
```

## Chapter 3: Saving Data

### Setting Up the Database

First, let's update our Prisma schema to define a Post model. Edit `api/db/schema.prisma`:

```prisma
datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

generator client {
  provider      = "prisma-client-js"
  binaryTargets = "native"
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  body      String
  createdAt DateTime @default(now())
}
```

Now run a migration to create the database tables:

```bash
yarn rw prisma migrate dev --name create_posts
```

### Building GraphQL and Services

Let's generate the SDL (Schema Definition Language) and service for our Post model:

```bash
yarn rw generate sdl Post
```

This creates:
- `api/src/graphql/posts.sdl.ts`: GraphQL schema
- `api/src/services/posts/posts.ts`: Database access functions

### Building a Form

Let's create a form for creating new posts:

```bash
yarn rw generate page NewPost /posts/new
```

This creates `web/src/pages/NewPostPage/NewPostPage.tsx`. Let's update it:

```jsx
import { Metadata } from '@redwoodjs/web'
import NewPost from 'src/components/NewPost/NewPost'
import { Link, routes } from '@redwoodjs/router'

const NewPostPage = () => {
  return (
    <>
      <Metadata title="New Post" description="Create a new blog post" />
      <h1>New Post</h1>
      <NewPost />
      <p>
        <Link to={routes.home()}>Return home</Link>
      </p>
    </>
  )
}

export default NewPostPage
```

Now let's create a component for the new post form:

```bash
yarn rw generate component NewPost
```

Update `web/src/components/NewPost/NewPost.tsx`:

```jsx
import { useState } from 'react'
import { useMutation } from '@redwoodjs/web'
import { toast } from '@redwoodjs/web/toast'
import { navigate, routes } from '@redwoodjs/router'
import {
  Form,
  FormError,
  FieldError,
  Label,
  TextField,
  TextAreaField,
  Submit,
} from '@redwoodjs/forms'

const CREATE_POST_MUTATION = gql`
  mutation CreatePostMutation($input: CreatePostInput!) {
    createPost(input: $input) {
      id
    }
  }
`

const NewPost = () => {
  const [createPost, { loading, error }] = useMutation(CREATE_POST_MUTATION, {
    onCompleted: () => {
      toast.success('Post created successfully')
      navigate(routes.home())
    },
    onError: (error) => {
      toast.error(error.message)
    },
  })

  const onSubmit = (data) => {
    createPost({ variables: { input: data } })
  }

  return (
    <div>
      <Form onSubmit={onSubmit} error={error}>
        <FormError error={error} />

        <Label name="title" errorClassName="error">
          Title
        </Label>
        <TextField
          name="title"
          validation={{ required: true }}
          errorClassName="error"
        />
        <FieldError name="title" className="error" />

        <Label name="body" errorClassName="error">
          Body
        </Label>
        <TextAreaField
          name="body"
          validation={{ required: true }}
          errorClassName="error"
        />
        <FieldError name="body" className="error" />

        <Submit disabled={loading}>Save</Submit>
      </Form>
    </div>
  )
}

export default NewPost
```

Let's update our homepage to show the posts and a link to create new posts:

```jsx
import { Metadata } from '@redwoodjs/web'
import { Link, routes } from '@redwoodjs/router'
import PostsCell from 'src/components/PostsCell'

const HomePage = () => {
  return (
    <>
      <Metadata title="Home" description="Home page" />
      <h1>RedwoodJS Blog</h1>
      <p>
        <Link to={routes.newPost()}>Create a new post</Link>
      </p>
      <PostsCell />
    </>
  )
}

export default HomePage
```

## Chapter 4: Authentication

RedwoodJS supports various authentication providers. For this tutorial, we'll use dbAuth which is Redwood's built-in, database-backed authentication system.

```bash
yarn rw setup auth dbAuth
```

This will:
1. Install the required packages
2. Create a User model in your Prisma schema
3. Add authentication routes
4. Set up the necessary components

Let's update our Prisma schema to include the User model:

```prisma
model User {
  id                  Int       @id @default(autoincrement())
  name                String?
  email               String    @unique
  hashedPassword      String
  salt                String
  resetToken          String?
  resetTokenExpiresAt DateTime?
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  body      String
  createdAt DateTime @default(now())
  userId    Int?
  user      User?    @relation(fields: [userId], references: [id])
}
```

Run a migration to update the database:

```bash
yarn rw prisma migrate dev --name add_user_model
```

Update the Blog layout to include login/logout functionality:

```jsx
import { useAuth } from '@redwoodjs/auth'
import { Link, routes } from '@redwoodjs/router'

type BlogLayoutProps = {
  children?: React.ReactNode
}

const BlogLayout = ({ children }: BlogLayoutProps) => {
  const { isAuthenticated, currentUser, logOut } = useAuth()

  return (
    <>
      <header>
        <h1>
          <Link to={routes.home()}>Redwood Blog</Link>
        </h1>
        <nav>
          <ul>
            <li>
              <Link to={routes.home()}>Home</Link>
            </li>
            <li>
              <Link to={routes.about()}>About</Link>
            </li>
            <li>
              <Link to={routes.newPost()}>New Post</Link>
            </li>
            {isAuthenticated ? (
              <li>
                <button onClick={logOut}>Logout</button>
              </li>
            ) : (
              <li>
                <Link to={routes.login()}>Login</Link>
              </li>
            )}
          </ul>
        </nav>
        {isAuthenticated && <div>Logged in as {currentUser.email}</div>}
      </header>
      <main>{children}</main>
    </>
  )
}

export default BlogLayout
```

## Intermission

Let's take a moment to review what we've built so far:

1. A basic blog with a home page and about page
2. A posts listing that fetches data from the database
3. A form for creating new posts
4. User authentication

Next, we'll dive into more advanced features like Storybook for component development, testing, and role-based access control.

## Chapter 5: Frontend Magic

### Introduction to Storybook

Storybook is a tool for developing UI components in isolation. It makes it easy to develop and test components without needing to run the entire application.

Start Storybook with:

```bash
yarn rw storybook
```

This will launch Storybook at `http://localhost:7910`.

### Our First Story

Let's create a simple component and its story:

```bash
yarn rw generate component Article
```

Update `web/src/components/Article/Article.tsx`:

```jsx
import { Link, routes } from '@redwoodjs/router'

interface ArticleProps {
  article: {
    id: number
    title: string
    body: string
    createdAt: string
  }
  summary?: boolean
}

const Article = ({ article, summary = false }: ArticleProps) => {
  return (
    <article>
      <header>
        <h2>
          <Link to={routes.post({ id: article.id })}>{article.title}</Link>
        </h2>
      </header>
      <div>
        {summary ? article.body.substring(0, 100) + '...' : article.body}
      </div>
      <div>Posted at: {new Date(article.createdAt).toLocaleString()}</div>
    </article>
  )
}

export default Article
```

Update its story `web/src/components/Article/Article.stories.tsx`:

```jsx
import Article from './Article'

export default {
  component: Article,
  title: 'Components/Article',
}

const Template = (args) => <Article {...args} />

export const Full = Template.bind({})
Full.args = {
  article: {
    id: 1,
    title: 'First Post',
    body: 'This is the body of the first post. It is a long body with lots of text.',
    createdAt: '2020-01-01T12:34:56Z',
  },
  summary: false,
}

export const Summary = Template.bind({})
Summary.args = {
  article: {
    id: 1,
    title: 'First Post',
    body: 'This is the body of the first post. It is a long body with lots of text.',
    createdAt: '2020-01-01T12:34:56Z',
  },
  summary: true,
}
```

### Testing with Jest

RedwoodJS comes with Jest configured for testing. Let's write a test for our Article component:

```jsx
import { render, screen } from '@redwoodjs/testing/web'
import Article from './Article'

describe('Article', () => {
  const article = {
    id: 1,
    title: 'Test Article',
    body: 'Test article body',
    createdAt: '2020-01-01T12:34:56Z',
  }

  it('renders a full article', () => {
    render(<Article article={article} />)
    
    expect(screen.getByText(article.title)).toBeInTheDocument()
    expect(screen.getByText(article.body)).toBeInTheDocument()
  })

  it('renders a summary of an article', () => {
    render(<Article article={article} summary={true} />)
    
    expect(screen.getByText(article.title)).toBeInTheDocument()
    expect(screen.getByText(article.body + '...')).toBeInTheDocument()
  })
})
```

Run the tests with:

```bash
yarn rw test
```

## Chapter 6: Building Components

Let's add comments to our blog posts. First, update the Prisma schema:

```prisma
model Comment {
  id        Int      @id @default(autoincrement())
  name      String
  body      String
  postId    Int
  post      Post     @relation(fields: [postId], references: [id])
  createdAt DateTime @default(now())
}

model Post {
  id        Int       @id @default(autoincrement())
  title     String
  body      String
  comments  Comment[]
  createdAt DateTime  @default(now())
  userId    Int?
  user      User?     @relation(fields: [userId], references: [id])
}
```

Run a migration:

```bash
yarn rw prisma migrate dev --name add_comments
```

Generate the SDL and service for comments:

```bash
yarn rw generate sdl Comment
```

Now, let's create a comment form component:

```bash
yarn rw generate component CommentForm
```

Update `web/src/components/CommentForm/CommentForm.tsx`:

```jsx
import { useState } from 'react'
import { useMutation } from '@redwoodjs/web'
import { toast } from '@redwoodjs/web/toast'
import {
  Form,
  FormError,
  Label,
  TextField,
  TextAreaField,
  Submit,
  FieldError,
} from '@redwoodjs/forms'

const CREATE_COMMENT_MUTATION = gql`
  mutation CreateCommentMutation($input: CreateCommentInput!) {
    createComment(input: $input) {
      id
      name
      body
      createdAt
    }
  }
`

const CommentForm = ({ postId }) => {
  const [createComment, { loading, error }] = useMutation(CREATE_COMMENT_MUTATION, {
    onCompleted: () => {
      toast.success('Comment added successfully')
      setHasSubmitted(true)
    },
    onError: (error) => {
      toast.error(error.message)
    },
  })

  const [hasSubmitted, setHasSubmitted] = useState(false)

  const onSubmit = (data) => {
    createComment({ variables: { input: { ...data, postId } } })
  }

  return (
    <div className="comment-form">
      <h3>Leave a Comment</h3>
      {hasSubmitted ? (
        <div>Thank you for your comment!</div>
      ) : (
        <Form onSubmit={onSubmit} error={error}>
          <FormError error={error} />

          <Label name="name" errorClassName="error">
            Name
          </Label>
          <TextField
            name="name"
            validation={{ required: true }}
            errorClassName="error"
          />
          <FieldError name="name" className="error" />

          <Label name="body" errorClassName="error">
            Comment
          </Label>
          <TextAreaField
            name="body"
            validation={{ required: true }}
            errorClassName="error"
          />
          <FieldError name="body" className="error" />

          <Submit disabled={loading}>Submit</Submit>
        </Form>
      )}
    </div>
  )
}

export default CommentForm
```

Now, let's create a component to display comments:

```bash
yarn rw generate component Comments
```

Update `web/src/components/Comments/Comments.tsx`:

```jsx
import { useMutation } from '@redwoodjs/web'
import { useState } from 'react'
import CommentForm from '../CommentForm/CommentForm'

const DELETE_COMMENT_MUTATION = gql`
  mutation DeleteCommentMutation($id: Int!) {
    deleteComment(id: $id) {
      id
    }
  }
`

const Comments = ({ comments, postId }) => {
  const [deleteComment] = useMutation(DELETE_COMMENT_MUTATION)

  const handleDelete = (id) => {
    if (confirm('Are you sure you want to delete this comment?')) {
      deleteComment({ 
        variables: { id },
        refetchQueries: ['FindPostQuery'] 
      })
    }
  }

  return (
    <div className="comments">
      <h2>Comments</h2>
      {comments.map((comment) => (
        <div key={comment.id} className="comment">
          <h3>{comment.name} commented:</h3>
          <p>{comment.body}</p>
          <time dateTime={comment.createdAt}>
            {new Date(comment.createdAt).toLocaleString()}
          </time>
          <button onClick={() => handleDelete(comment.id)}>Delete</button>
        </div>
      ))}
      <CommentForm postId={postId} />
    </div>
  )
}

export default Comments
```

Finally, update our PostCell to include comments:

```jsx
export const QUERY = gql`
  query FindPostQuery($id: Int!) {
    post: post(id: $id) {
      id
      title
      body
      createdAt
      comments {
        id
        name
        body
        createdAt
      }
    }
  }
`

export const Success = ({ post }) => {
  return (
    <>
      <article>
        <header>
          <h2>{post.title}</h2>
        </header>
        <div>{post.body}</div>
        <div>Posted at: {new Date(post.createdAt).toLocaleString()}</div>
      </article>
      <Comments comments={post.comments} postId={post.id} />
    </>
  )
}
```

## Chapter 7: Role-Based Access Control

### Securing the App

Let's implement role-based access control to secure our application. First, let's update our User model to include roles:

```prisma
model User {
  id                  Int       @id @default(autoincrement())
  name                String?
  email               String    @unique
  hashedPassword      String
  salt                String
  resetToken          String?
  resetTokenExpiresAt DateTime?
  roles               String    @default("user")
  posts               Post[]
}
```

Run a migration:

```bash
yarn rw prisma migrate dev --name add_user_roles
```

Next, let's update our auth configuration to support roles. In `api/src/lib/auth.ts`:

```typescript
import { AuthenticationError, ForbiddenError } from '@redwoodjs/graphql-server'

/**
 * Checks if the current user is authenticated
 */
export const isAuthenticated = () => {
  return context.currentUser !== undefined
}

/**
 * Checks if the current user has one of the specified roles
 */
export const hasRole = ({ roles }) => {
  if (!isAuthenticated()) {
    return false
  }
  
  const currentUserRoles = context.currentUser?.roles.split(',') || []
  
  return currentUserRoles.some((role) => roles.includes(role))
}

/**
 * Requires that the current user is authenticated and has the specified roles
 */
export const requireAuth = ({ roles } = {}) => {
  if (!isAuthenticated()) {
    throw new AuthenticationError("You don't have permission to do that.")
  }

  if (roles && !hasRole({ roles })) {
    throw new ForbiddenError("You don't have access to do that.")
  }

  return context.currentUser
}
```

Now, let's secure our Post service in `api/src/services/posts/posts.ts`:

```typescript
import { db } from 'src/lib/db'
import { requireAuth } from 'src/lib/auth'

export const posts = () => {
  return db.post.findMany()
}

export const post = ({ id }) => {
  return db.post.findUnique({
    where: { id },
  })
}

export const createPost = ({ input }) => {
  requireAuth({ roles: ['admin', 'editor'] })
  return db.post.create({
    data: {
      ...input,
      userId: context.currentUser.id,
    },
  })
}

export const updatePost = ({ id, input }) => {
  requireAuth({ roles: ['admin', 'editor'] })
  return db.post.update({
    data: input,
    where: { id },
  })
}

export const deletePost = ({ id }) => {
  requireAuth({ roles: ['admin'] })
  return db.post.delete({
    where: { id },
  })
}
```

Let's also secure our Comment service:

```typescript
import { db } from 'src/lib/db'
import { requireAuth } from 'src/lib/auth'

export const comments = ({ postId }) => {
  return db.comment.findMany({
    where: { postId },
  })
}

export const comment = ({ id }) => {
  return db.comment.findUnique({
    where: { id },
  })
}

export const createComment = ({ input }) => {
  return db.comment.create({
    data: input,
  })
}

export const deleteComment = ({ id }) => {
  requireAuth({ roles: ['admin', 'moderator'] })
  return db.comment.delete({
    where: { id },
  })
}
```

Finally, let's update our UI to show or hide features based on user roles:

```jsx
import { useAuth } from '@redwoodjs/auth'

// In your component
const { currentUser, hasRole } = useAuth()

// Then in your JSX
{hasRole(['admin', 'editor']) && (
  <Link to={routes.newPost()}>New Post</Link>
)}

{hasRole(['admin', 'moderator']) && (
  <button onClick={() => handleDelete(comment.id)}>Delete</button>
)}
```

## Afterword

Congratulations! You've built a complete blog application with RedwoodJS, featuring:

- Multiple pages and layouts
- Dynamic data fetching with Cells
- Database integration with Prisma
- Forms and data submission
- Authentication and authorization
- Component testing with Jest
- Component development with Storybook
- Role-based access control

This is just the beginning of what you can build with RedwoodJS. Some next steps you might want to explore:

1. **Deployment**: Deploy your app to Netlify, Vercel, or another hosting provider
2. **File uploads**: Add the ability to upload images for blog posts
3. **Markdown support**: Add markdown rendering for blog content
4. **Search functionality**: Implement search for posts and comments
5. **Pagination**: Add pagination for posts and comments
6. **Admin panel**: Build a dedicated admin area for managing content

RedwoodJS continues to evolve with new features and improvements. Check out the [official documentation](https://redwoodjs.com/docs) to stay up to date with the latest developments.

Happy coding!
