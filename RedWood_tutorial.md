# RedwoodJS v8.4 Blog Tutorial (TypeScript)

## Chapter 1: First Steps

### Installation

```bash
yarn create redwood-app --ts ./redwoodblog
cd redwoodblog
yarn install
yarn redwood dev
```

### First Page

```bash
yarn redwood generate page home /
```

Update CSS styling:

```css title="web/src/index.css"
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica,
    Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol';
}
ul {
  list-style-type: none;
  margin: 1rem 0;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 1rem 0 0;
}
h1 > a {
  text-decoration: none;
  color: black;
}
button,
input,
label,
textarea {
  display: block;
  outline: none;
}
label {
  margin-top: 1rem;
}
.error {
  color: red;
}
input.error,
textarea.error {
  border: 1px solid red;
}
.form-error {
  color: red;
  background-color: lavenderblush;
  padding: 1rem;
  display: inline-block;
}
.form-error ul {
  list-style-type: disc;
  margin: 1rem;
  padding: 1rem;
}
.form-error li {
  display: list-item;
}
.flex-between {
  display: flex;
  justify-content: space-between;
}
.flex-between button {
  display: inline;
}
```

### Second Page

```bash
yarn redwood generate page about
```

Update the HomePage:

```tsx title="web/src/pages/HomePage/HomePage.tsx"
import { Link, routes } from '@redwoodjs/router'
import { Metadata } from '@redwoodjs/web'

const HomePage = () => {
  return (
    <>
      <Metadata title="Home" description="Home page" />

      <header>
        <h1>Redwood Blog</h1>
        <nav>
          <ul>
            <li>
              <Link to={routes.about()}>About</Link>
            </li>
          </ul>
        </nav>
      </header>
      <main>Home</main>
    </>
  )
}

export default HomePage
```

Update the AboutPage:

```tsx title="web/src/pages/AboutPage/AboutPage.tsx"
import { Link, routes } from '@redwoodjs/router'
import { Metadata } from '@redwoodjs/web'

const AboutPage = () => {
  return (
    <>
      <Metadata title="About" description="About page" />

      <header>
        <h1>Redwood Blog</h1>
        <nav>
          <ul>
            <li>
              <Link to={routes.about()}>About</Link>
            </li>
          </ul>
        </nav>
      </header>
      <main>
        <p>
          This site was created to demonstrate my mastery of Redwood: Look on my
          works, ye mighty, and despair!
        </p>
        <Link to={routes.home()}>Return home</Link>
      </main>
    </>
  )
}

export default AboutPage
```

### Layouts

```bash
yarn rw g layout blog
```

```tsx title="web/src/layouts/BlogLayout/BlogLayout.tsx"
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

Update Routes:

```tsx title="web/src/Routes.tsx"
import { Router, Route, Set } from '@redwoodjs/router'
import BlogLayout from 'src/layouts/BlogLayout'

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

Update HomePage:

```tsx title="web/src/pages/HomePage/HomePage.tsx"
import { Metadata } from '@redwoodjs/web'

const HomePage = () => {
  return (
    <>
      <Metadata title="Home" description="Home page" />
      Home
    </>
  )
}

export default HomePage
```

Update AboutPage:

```tsx title="web/src/pages/AboutPage/AboutPage.tsx"
import { Metadata } from '@redwoodjs/web'

const AboutPage = () => {
  return (
    <>
      <Metadata title="About" description="About page" />

      <p>
        This site was created to demonstrate my mastery of Redwood: Look on my
        works, ye mighty, and despair!
      </p>
    </>
  )
}

export default AboutPage
```

## Chapter 2: Getting Dynamic

### Creating the Database Schema

```bash
yarn rw prisma migrate dev
```

Update schema.prisma:

```prisma title="api/db/schema.prisma"
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

Name the migration "create post"

### Creating a Post Editor

```bash
yarn rw g scaffold post
```

### Your First Cell

```bash
yarn rw g cell Articles
```

Update the cell:

```tsx title="web/src/components/ArticlesCell/ArticlesCell.tsx"
import type { ArticlesQuery, ArticlesQueryVariables } from 'types/graphql'

import type {
  CellFailureProps,
  CellSuccessProps,
  TypedDocumentNode,
} from '@redwoodjs/web'

export const QUERY: TypedDocumentNode<ArticlesQuery, ArticlesQueryVariables> =
  gql`
    query ArticlesQuery {
      articles: posts {
        id
        title
        body
        createdAt
      }
    }
  `

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({
  error,
}: CellFailureProps<ArticlesQueryVariables>) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({
  articles,
}: CellSuccessProps<ArticlesQuery, ArticlesQueryVariables>) => {
  return (
    <>
      {articles.map((article) => (
        <article key={article.id}>
          <header>
            <h2>{article.title}</h2>
          </header>
          <p>{article.body}</p>
          <div>Posted at: {article.createdAt}</div>
        </article>
      ))}
    </>
  )
}
```

Update the HomePage:

```tsx title="web/src/pages/HomePage/HomePage.tsx"
import { Metadata } from '@redwoodjs/web'

import ArticlesCell from 'src/components/ArticlesCell'

const HomePage = () => {
  return (
    <>
      <Metadata title="Home" description="Home page" />
      <ArticlesCell />
    </>
  )
}

export default HomePage
```

### Routing Parameters

```bash
yarn rw g page Article
```

Update Routes:

```tsx title="web/src/Routes.tsx"
import { Router, Route, Set } from '@redwoodjs/router'
import ScaffoldLayout from 'src/layouts/ScaffoldLayout'
import BlogLayout from 'src/layouts/BlogLayout'

const Routes = () => {
  return (
    <Router>
      <Set wrap={ScaffoldLayout} title="Posts" titleTo="posts" buttonLabel="New Post" buttonTo="newPost">
        <Route path="/posts/new" page={PostNewPostPage} name="newPost" />
        <Route path="/posts/{id:Int}/edit" page={PostEditPostPage} name="editPost" />
        <Route path="/posts/{id:Int}" page={PostPostPage} name="post" />
        <Route path="/posts" page={PostPostsPage} name="posts" />
      </Set>
      <Set wrap={BlogLayout}>
        <Route path="/article/{id:Int}" page={ArticlePage} name="article" />
        <Route path="/about" page={AboutPage} name="about" />
        <Route path="/" page={HomePage} name="home" />
      </Set>
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```

Update ArticlesCell:

```tsx title="web/src/components/ArticlesCell/ArticlesCell.tsx"
import { Link, routes } from '@redwoodjs/router'
import type { ArticlesQuery, ArticlesQueryVariables } from 'types/graphql'

import type {
  CellFailureProps,
  CellSuccessProps,
  TypedDocumentNode,
} from '@redwoodjs/web'

export const QUERY: TypedDocumentNode<ArticlesQuery, ArticlesQueryVariables> =
  gql`
    query ArticlesQuery {
      articles: posts {
        id
        title
        body
        createdAt
      }
    }
  `

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({
  error,
}: CellFailureProps<ArticlesQueryVariables>) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({
  articles,
}: CellSuccessProps<ArticlesQuery, ArticlesQueryVariables>) => {
  return (
    <>
      {articles.map((article) => (
        <article key={article.id}>
          <header>
            <h2>
              <Link to={routes.article({ id: article.id })}>{article.title}</Link>
            </h2>
          </header>
          <p>{article.body}</p>
          <div>Posted at: {article.createdAt}</div>
        </article>
      ))}
    </>
  )
}
```

```bash
yarn rw g cell Article
```

Update ArticleCell:

```tsx title="web/src/components/ArticleCell/ArticleCell.tsx"
import type { FindArticleQuery, FindArticleQueryVariables } from 'types/graphql'

import type {
  CellFailureProps,
  CellSuccessProps,
  TypedDocumentNode,
} from '@redwoodjs/web'

export const QUERY: TypedDocumentNode<
  FindArticleQuery,
  FindArticleQueryVariables
> = gql`
  query FindArticleQuery($id: Int!) {
    article: post(id: $id) {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({
  error,
}: CellFailureProps<FindArticleQueryVariables>) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({
  article,
}: CellSuccessProps<FindArticleQuery, FindArticleQueryVariables>) => {
  return (
    <article>
      <header>
        <h2>{article.title}</h2>
      </header>
      <div>{article.body}</div>
      <div>Posted at: {article.createdAt}</div>
    </article>
  )
}
```

Update ArticlePage:

```tsx title="web/src/pages/ArticlePage/ArticlePage.tsx"
import { Metadata } from '@redwoodjs/web'
import ArticleCell from 'src/components/ArticleCell'

interface Props {
  id: number
}

const ArticlePage = ({ id }: Props) => {
  return (
    <>
      <Metadata title="Article" description="Article page" />

      <ArticleCell id={id} />
    </>
  )
}

export default ArticlePage
```

## Chapter 3: Saving Data

### Adding a Contact Form

```bash
yarn rw g sdl Contact --no-crud
yarn rw g page contact
```

Update contact schema:

```prisma title="api/db/schema.prisma"
model Contact {
  id        Int @id @default(autoincrement())
  name      String
  email     String
  message   String
  createdAt DateTime @default(now())
}
```

```bash
yarn rw prisma migrate dev
```

Update BlogLayout:

```tsx title="web/src/layouts/BlogLayout/BlogLayout.tsx"
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
            <li>
              <Link to={routes.contact()}>Contact</Link>
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

Update ContactPage:

```tsx title="web/src/pages/ContactPage/ContactPage.tsx"
import { Metadata } from '@redwoodjs/web'
import {
  FieldError,
  Form,
  FormError,
  Label,
  TextField,
  TextAreaField,
  Submit,
  SubmitHandler,
  useForm,
} from '@redwoodjs/forms'
import { useMutation } from '@redwoodjs/web'
import { toast, Toaster } from '@redwoodjs/web/toast'

import {
  CreateContactMutation,
  CreateContactMutationVariables,
} from 'types/graphql'

const CREATE_CONTACT = gql`
  mutation CreateContactMutation($input: CreateContactInput!) {
    createContact(input: $input) {
      id
    }
  }
`

interface FormValues {
  name: string
  email: string
  message: string
}

const ContactPage = () => {
  const formMethods = useForm()

  const [create, { loading, error }] = useMutation<
    CreateContactMutation,
    CreateContactMutationVariables
  >(CREATE_CONTACT, {
    onCompleted: () => {
      toast.success('Thank you for your submission!')
      formMethods.reset()
    },
  })

  const onSubmit: SubmitHandler<FormValues> = (data) => {
    create({ variables: { input: data } })
  }

  return (
    <>
      <Metadata title="Contact" description="Contact page" />

      <Toaster />
      <Form
        onSubmit={onSubmit}
        config={{ mode: 'onBlur' }}
        error={error}
        formMethods={formMethods}
      >
        <FormError
          error={error}
          wrapperClassName="form-error"
        />

        <Label name="name" className="block text-sm text-gray-600 uppercase">
          Name
        </Label>
        <TextField
          name="name"
          validation={{ required: true }}
          className="block w-full p-1 border rounded text-sm"
          errorClassName="block w-full p-1 border rounded text-sm error"
        />
        <FieldError name="name" className="error" />

        <Label
          name="email"
          className="block mt-4 text-sm text-gray-600 uppercase"
        >
          Email
        </Label>
        <TextField
          name="email"
          validation={{
            required: true,
            pattern: {
              value: /^[^@]+@[^.]+\..+$/,
              message: 'Please enter a valid email address',
            },
          }}
          className="block w-full p-1 border rounded text-sm"
          errorClassName="block w-full p-1 border rounded text-sm error"
        />
        <FieldError name="email" className="error" />

        <Label
          name="message"
          className="block mt-4 text-sm text-gray-600 uppercase"
        >
          Message
        </Label>
        <TextAreaField
          name="message"
          validation={{ required: true }}
          className="block w-full p-1 border rounded h-24 text-sm"
          errorClassName="block w-full p-1 border rounded h-24 text-sm error"
        />
        <FieldError name="message" className="error" />

        <Submit
          disabled={loading}
          className="block mt-4 bg-blue-500 text-white text-xs font-semibold uppercase tracking-wide rounded px-3 py-2 disabled:opacity-50"
        >
          Save
        </Submit>
      </Form>
    </>
  )
}

export default ContactPage
```

Update contacts service:

```tsx title="api/src/services/contacts/contacts.ts"
import type { QueryResolvers, MutationResolvers } from 'types/graphql'

import { validate } from '@redwoodjs/api'

import { db } from 'src/lib/db'

export const contacts: QueryResolvers['contacts'] = () => {
  return db.contact.findMany()
}

export const contact: QueryResolvers['contact'] = ({ id }) => {
  return db.contact.findUnique({
    where: { id },
  })
}

export const createContact: MutationResolvers['createContact'] = ({ input }) => {
  validate(input.email, 'email', { email: true })
  return db.contact.create({
    data: input,
  })
}
```

Update contacts SDL:

```tsx title="api/src/graphql/contacts.sdl.ts"
export const schema = gql`
  type Contact {
    id: Int!
    name: String!
    email: String!
    message: String!
    createdAt: DateTime!
  }

  type Query {
    contacts: [Contact!]! @requireAuth
    contact(id: Int!): Contact @requireAuth
  }

  input CreateContactInput {
    name: String!
    email: String!
    message: String!
  }

  input UpdateContactInput {
    name: String
    email: String
    message: String
  }

  type Mutation {
    createContact(input: CreateContactInput!): Contact! @skipAuth
  }
`
```

## Chapter 4: Authentication & Deployment

### Authentication

```bash
yarn rw setup auth dbAuth
```

Update User model:

```prisma title="api/db/schema.prisma"
model User {
  id                  Int       @id @default(autoincrement())
  name                String?
  email               String    @unique
  hashedPassword      String
  salt                String
  resetToken          String?
  resetTokenExpiresAt DateTime?
  roles               String    @default("moderator")
}
```

```bash
yarn rw prisma migrate dev
```

Update Routes:

```tsx title="web/src/Routes.tsx"
import { PrivateSet, Router, Route, Set } from '@redwoodjs/router'
import ScaffoldLayout from 'src/layouts/ScaffoldLayout'
import BlogLayout from 'src/layouts/BlogLayout'

import { useAuth } from './auth'

const Routes = () => {
  return (
    <Router useAuth={useAuth}>
      <PrivateSet unauthenticated="home" roles="admin">
        <Set wrap={ScaffoldLayout} title="Posts" titleTo="posts" buttonLabel="New Post" buttonTo="newPost">
          <Route path="/admin/posts/new" page={PostNewPostPage} name="newPost" />
          <Route path="/admin/posts/{id:Int}/edit" page={PostEditPostPage} name="editPost" />
          <Route path="/admin/posts/{id:Int}" page={PostPostPage} name="post" />
          <Route path="/admin/posts" page={PostPostsPage} name="posts" />
        </Set>
      </PrivateSet>
      <Set wrap={BlogLayout}>
        <Route path="/article/{id:Int}" page={ArticlePage} name="article" />
        <Route path="/contact" page={ContactPage} name="contact" />
        <Route path="/about" page={AboutPage} name="about" />
        <Route path="/" page={HomePage} name="home" />
      </Set>
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```

Update BlogLayout:

```tsx title="web/src/layouts/BlogLayout/BlogLayout.tsx"
import { Link, routes } from '@redwoodjs/router'
import { Toaster } from '@redwoodjs/web/toast'

import { useAuth } from 'src/auth'

type BlogLayoutProps = {
  children?: React.ReactNode
}

const BlogLayout = ({ children }: BlogLayoutProps) => {
  const { isAuthenticated, currentUser, logOut } = useAuth()

  return (
    <>
      <Toaster />
      <header className="relative flex justify-between items-center py-4 px-8 bg-blue-700 text-white">
        <h1 className="text-5xl font-semibold tracking-tight">
          <Link
            className="text-blue-400 hover:text-blue-100 transition duration-100"
            to={routes.home()}
          >
            Redwood Blog
          </Link>
        </h1>
        <nav>
          <ul className="relative flex items-center font-light">
            <li>
              <Link
                className="py-2 px-4 hover:bg-blue-600 transition duration-100 rounded"
                to={routes.about()}
              >
                About
              </Link>
            </li>
            <li>
              <Link
                className="py-2 px-4 hover:bg-blue-600 transition duration-100 rounded"
                to={routes.contact()}
              >
                Contact
              </Link>
            </li>
            <li>
              {isAuthenticated ? (
                <div>
                  <button type="button" onClick={logOut} className="py-2 px-4">
                    Logout
                  </button>
                </div>
              ) : (
                <Link to={routes.login()} className="py-2 px-4">
                  Login
                </Link>
              )}
            </li>
          </ul>
          {isAuthenticated && (
            <div className="absolute bottom-1 right-0 mr-12 text-xs text-blue-300">
              {currentUser.email}
            </div>
          )}
        </nav>
      </header>
      <main className="max-w-4xl mx-auto p-12 bg-white shadow rounded-b">
        {children}
      </main>
    </>
  )
}

export default BlogLayout
```

Update auth.ts:

```tsx title="api/src/lib/auth.ts"
import { db } from './db'

export const getCurrentUser = async (session) => {
  return await db.user.findUnique({
    where: { id: session.id },
    select: { id: true, email: true, roles: true },
  })
}
```

### Netlify Deployment Setup

```bash
yarn rw setup deploy netlify
```

Create a .env file with your DATABASE_URL and set SESSION_SECRET:

```bash
yarn rw g secret
```

## Chapter 5: Component Testing & Storybook

### Create a Comment Component

```bash
yarn rw g component Comment
```

Update Comment component:

```tsx title="web/src/components/Comment/Comment.tsx"
import { useAuth } from 'src/auth'

const formattedDate = (datetime: ConstructorParameters<typeof Date>[0]) => {
  const parsedDate = new Date(datetime)
  const month = parsedDate.toLocaleString('default', { month: 'long' })
  return `${parsedDate.getDate()} ${month} ${parsedDate.getFullYear()}`
}

interface Props {
  comment: {
    name: string
    createdAt: string
    body: string
  }
}

const Comment = ({ comment }: Props) => {
  const { hasRole } = useAuth()
  const moderate = () => {
    if (confirm('Are you sure?')) {
      // Delete functionality will be added later
    }
  }

  return (
    <div className="bg-gray-200 p-8 rounded-lg relative">
      <header className="flex justify-between">
        <h2 className="font-semibold text-gray-700">{comment.name}</h2>
        <time className="text-xs text-gray-500" dateTime={comment.createdAt}>
          {formattedDate(comment.createdAt)}
        </time>
      </header>
      <p className="text-sm mt-2">{comment.body}</p>
      {hasRole('moderator') && (
        <button
          type="button"
          onClick={moderate}
          className="absolute bottom-2 right-2 bg-red-500 text-xs rounded text-white px-2 py-1"
        >
          Delete
        </button>
      )}
    </div>
  )
}

export default Comment
```

Update Comment.stories.tsx:

```tsx title="web/src/components/Comment/Comment.stories.tsx"
import Comment from './Comment'

export const defaultView = () => {
  return (
    <Comment
      comment={{
        id: 1,
        name: 'Rob Cameron',
        body: 'This is the first comment!',
        createdAt: '2020-01-01T12:34:56Z',
        postId: 1,
      }}
    />
  )
}

export const moderatorView = () => {
  mockCurrentUser({
    id: 1,
    email: 'moderator@moderator.com',
    roles: 'moderator',
  })

  return (
    <Comment
      comment={{
        id: 1,
        name: 'Rob Cameron',
        body: 'This is the first comment!',
        createdAt: '2020-01-01T12:34:56Z',
        postId: 1,
      }}
    />
  )
}

export default { title: 'Components/Comment' }
```

Update Comment.test.tsx:

```tsx title="web/src/components/Comment/Comment.test.tsx"
import { render, screen, waitFor } from '@redwoodjs/testing'

import Comment from './Comment'

const COMMENT = {
  id: 1,
  name: 'John Doe',
  body: 'This is my comment',
  createdAt: '2020-01-02T12:34:56Z',
  postId: 1,
}

describe('Comment', () => {
  it('renders successfully', () => {
    render(<Comment comment={COMMENT} />)

    expect(screen.getByText(COMMENT.name)).toBeInTheDocument()
    expect(screen.getByText(COMMENT.body)).toBeInTheDocument()
    const dateExpect = screen.getByText('2 January 2020')
    expect(dateExpect).toBeInTheDocument()
    expect(dateExpect.nodeName).toEqual('TIME')
    expect(dateExpect).toHaveAttribute('datetime', COMMENT.createdAt)
  })

  it('does not render a delete button if user is logged out', async () => {
    render(<Comment comment={COMMENT} />)

    await waitFor(() =>
      expect(screen.queryByText('Delete')).not.toBeInTheDocument()
    )
  })

  it('renders a delete button if the user is a moderator', async () => {
    mockCurrentUser({
      id: 1,
      email: 'moderator@moderator.com',
      roles: 'moderator',
    })

    render(<Comment comment={COMMENT} />)

    await waitFor(() => expect(screen.getByText('Delete')).toBeInTheDocument())
  })
})
```

### Create a CommentsCell

```bash
yarn rw g cell Comments
```

Update CommentsCell:

```tsx title="web/src/components/CommentsCell/CommentsCell.tsx"
import type { CommentsQuery, CommentsQueryVariables } from 'types/graphql'

import type {
  CellFailureProps,
  CellSuccessProps,
  TypedDocumentNode,
} from '@redwoodjs/web'

import Comment from 'src/components/Comment'

export const QUERY: TypedDocumentNode<CommentsQuery, CommentsQueryVariables> =
  gql`
    query CommentsQuery($postId: Int!) {
      comments(postId: $postId) {
        id
        name
        body
        postId
        createdAt
      }
    }
  `

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div className="text-center text-gray-500">No comments yet</div>

export const Failure = ({
  error,
}: CellFailureProps<CommentsQueryVariables>) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({
  comments,
}: CellSuccessProps<CommentsQuery, CommentsQueryVariables>) => {
  return (
    <div className="space-y-8">
      {comments.map((comment) => (
        <Comment key={comment.id} comment={comment} />
      ))}
    </div>
  )
}
```

Update CommentsCell.mock.ts:

```ts title="web/src/components/CommentsCell/CommentsCell.mock.ts"
export const standard = () => ({
  comments: [
    {
      id: 1,
      name: 'Rob Cameron',
      body: 'First comment',
      postId: 1,
      createdAt: '2020-01-02T12:34:56Z',
    },
    {
      id: 2,
      name: 'David Price',
      body: 'Second comment',
      postId: 2,
      createdAt: '2020-02-03T23:00:00Z',
    },
  ],
})
```

## Chapter 6: Comments Feature

### Create the Comment Model

```prisma title="api/db/schema.prisma"
model Comment {
  id        Int      @id @default(autoincrement())
  name      String
  body      String
  post      Post     @relation(fields: [postId], references: [id])
  postId    Int
  createdAt DateTime @default(now())
}

model Post {
  id        Int       @id @default(autoincrement())
  title     String
  body      String
  comments  Comment[]
  createdAt DateTime  @default(now())
}
```

```bash
yarn rw prisma migrate dev
```

```bash
yarn rw g sdl Comment
```

Update comments SDL:

```tsx title="api/src/graphql/comments.sdl.ts"
export const schema = gql`
  type Comment {
    id: Int!
    name: String!
    body: String!
    post: Post!
    postId: Int!
    createdAt: DateTime!
  }

  type Query {
    comments(postId: Int!): [Comment!]! @skipAuth
  }

  input CreateCommentInput {
    name: String!
    body: String!
    postId: Int!
  }

  input UpdateCommentInput {
    name: String
    body: String
    postId: Int
  }

  type Mutation {
    createComment(input: CreateCommentInput!): Comment! @skipAuth
    deleteComment(id: Int!): Comment! @requireAuth(roles: "moderator")
  }
`
```

Update comments service:

```tsx title="api/src/services/comments/comments.ts"
import { ForbiddenError } from '@redwoodjs/graphql-server'

import { db } from 'src/lib/db'

const verifyOwnership = async ({ id }) => {
  if (await adminPost({ id })) {
    return true
  } else {
    throw new ForbiddenError("You don't have access to this post")
  }
}

export const comments = ({ postId }) => {
  return db.comment.findMany({ where: { postId } })
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

export const deleteComment = async ({ id }) => {
  requireAuth({ roles: 'moderator' })
  return db.comment.delete({
    where: { id },
  })
}

export const Comment = {
  post: (_obj, { root }) => {
    return db.comment.findFirst({ where: { id: root?.id } }).post()
  },
}
```

### Create CommentForm Component

```bash
yarn rw g component CommentForm
```

Update CommentForm:

```tsx title="web/src/components/CommentForm/CommentForm.tsx"
import type {
  CreateCommentMutation,
  CreateCommentMutationVariables,
} from 'types/graphql'

import {
  Form,
  FormError,
  Label,
  TextField,
  TextAreaField,
  Submit,
  SubmitHandler,
} from '@redwoodjs/forms'
import type { TypedDocumentNode } from '@redwoodjs/web'
import { useMutation } from '@redwoodjs/web'
import { toast } from '@redwoodjs/web/toast'

import { QUERY as CommentsQuery } from 'src/components/CommentsCell'

const CREATE: TypedDocumentNode<
  CreateCommentMutation,
  CreateCommentMutationVariables
> = gql`
  mutation CreateCommentMutation($input: CreateCommentInput!) {
    createComment(input: $input) {
      id
      name
      body
      createdAt
    }
  }
`

interface FormValues {
  name: string
  body: string
}

interface Props {
  postId: number
}

const CommentForm = ({ postId }: Props) => {
  const [hasPosted, setHasPosted] = useState(false)
  const [createComment, { loading, error }] = useMutation(CREATE, {
    onCompleted: () => {
      setHasPosted(true)
      toast.success('Thank you for your comment!')
    },
    refetchQueries: [{ query: CommentsQuery, variables: { postId } }],
  })

  const onSubmit: SubmitHandler<FormValues> = (input) => {
    createComment({ variables: { input: { postId, ...input } } })
  }

  return (
    <div className={hasPosted ? 'hidden' : ''}>
      <h3 className="font-light text-lg text-gray-600">Leave a Comment</h3>
      <Form className="mt-4 w-full" onSubmit={onSubmit}>
        <FormError
          error={error}
          titleClassName="font-semibold"
          wrapperClassName="bg-red-100 text-red-900 text-sm p-3 rounded"
        />
        <Label
          name="name"
          className="block text-xs font-semibold text-gray-500 uppercase"
        >
          Name
        </Label>
        <TextField
          name="name"
          validation={{ required: true }}
          className="block w-full p-1 border rounded text-sm "
          errorClassName="block w-full p-1 border rounded text-sm error"
        />
        <FieldError name="name" className="error" />

        <Label
          name="body"
          className="block mt-4 text-xs font-semibold text-gray-500 uppercase"
        >
          Comment
        </Label>
        <TextAreaField
          name="body"
          validation={{ required: true }}
          className="block w-full p-1 border rounded h-24 text-sm"
          errorClassName="block w-full p-1 border rounded h-24 text-sm error"
        />
        <FieldError name="body" className="error" />

        <Submit
          disabled={loading}
          className="block mt-4 bg-blue-500 text-white text-xs font-semibold uppercase tracking-wide rounded px-3 py-2 disabled:opacity-50"
        >
          Submit
        </Submit>
      </Form>
    </div>
  )
}

export default CommentForm
```

Update CommentForm.stories.tsx:

```tsx title="web/src/components/CommentForm/CommentForm.stories.tsx"
import CommentForm from './CommentForm'

import type {
  CreateCommentMutation,
  CreateCommentMutationVariables,
} from 'types/graphql'

export const generated = () => {
  mockGraphQLMutation<CreateCommentMutation, CreateCommentMutationVariables>(
    'CreateCommentMutation',
    (variables, { ctx }) => {
      const id = Math.floor(Math.random() * 1000)
      ctx.delay(1000)

      return {
        createComment: {
          id,
          name: variables.input.name,
          body: variables.input.body,
          createdAt: new Date().toISOString(),
        },
      }
    }
  )

  return <CommentForm postId={1} />
}

export default { title: 'Components/CommentForm' }
```

### Update Article Component

Create a reusable Article component:

```bash
yarn rw g component Article
```

Update Article component:

```tsx title="web/src/components/Article/Article.tsx"
import { Link, routes } from '@redwoodjs/router'

import CommentForm from 'src/components/CommentForm'
import CommentsCell from 'src/components/CommentsCell'

import type { Post } from 'types/graphql'

const truncate = (text: string, length: number) => {
  return text.substring(0, length) + '...'
}

const Article = ({ article, summary = false }) => {
  return (
    <article>
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.article({ id: article.id })}>{article.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(article.body, 100) : article.body}
      </div>
      {!summary && (
        <div className="mt-12">
          <CommentForm postId={article.id} />
          <div className="mt-12">
            <CommentsCell postId={article.id} />
          </div>
        </div>
      )}
    </article>
  )
}

export default Article
```

Update ArticlesCell to use the Article component:

```tsx title="web/src/components/ArticlesCell/ArticlesCell.tsx"
import type { ArticlesQuery, ArticlesQueryVariables } from 'types/graphql'

import type {
  CellFailureProps,
  CellSuccessProps,
  TypedDocumentNode,
} from '@redwoodjs/web'

import Article from 'src/components/Article'

export const QUERY: TypedDocumentNode<ArticlesQuery, ArticlesQueryVariables> =
  gql`
    query ArticlesQuery {
      articles: posts {
        id
        title
        body
        createdAt
        user {
          name
        }
      }
    }
  `

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({
  error,
}: CellFailureProps<ArticlesQueryVariables>) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({
  articles,
}: CellSuccessProps<ArticlesQuery, ArticlesQueryVariables>) => {
  return (
    <>
      {articles.map((article) => (
        <Article key={article.id} article={article} summary={true} />
      ))}
    </>
  )
}
```

Update ArticleCell to use the Article component:

```tsx title="web/src/components/ArticleCell/ArticleCell.tsx"
import type { FindArticleQuery, FindArticleQueryVariables } from 'types/graphql'

import type {
  CellFailureProps,
  CellSuccessProps,
  TypedDocumentNode,
} from '@redwoodjs/web'

import Article from 'src/components/Article'

export const QUERY: TypedDocumentNode<
  FindArticleQuery,
  FindArticleQueryVariables
> = gql`
  query FindArticleQuery($id: Int!) {
    article: post(id: $id) {
      id
      title
      body
      createdAt
      user {
        name
      }
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({
  error,
}: CellFailureProps<FindArticleQueryVariables>) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({
  article,
}: CellSuccessProps<FindArticleQuery, FindArticleQueryVariables>) => {
  return <Article article={article} />
}
```

## Chapter 7: RBAC & currentUser

### Adding Roles to User

Update the Post schema:

```prisma title="api/db/schema.prisma"
model Post {
  id        Int      @id @default(autoincrement())
  title     String
  body      String
  comments  Comment[]
  user      User     @relation(fields: [userId], references: [id])
  userId    Int
  createdAt DateTime @default(now())
}

model User {
  id                  Int @id @default(autoincrement())
  name                String?
  email               String @unique
  hashedPassword      String
  salt                String
  resetToken          String?
  resetTokenExpiresAt DateTime?
  roles               String @default("moderator")
  posts               Post[]
}
```

```bash
yarn rw prisma migrate dev
```

### Update Posts Service

Create AdminPosts SDL:

```tsx title="api/src/graphql/adminPosts.sdl.ts"
export const schema = gql`
  type Query {
    adminPosts: [Post!]! @requireAuth(roles: ["admin"])
    adminPost(id: Int!): Post @requireAuth(roles: ["admin"])
  }

  input CreatePostInput {
    title: String!
    body: String!
  }

  input UpdatePostInput {
    title: String
    body: String
  }

  type Mutation {
    createPost(input: CreatePostInput!): Post! @requireAuth(roles: ["admin"])
    updatePost(id: Int!, input: UpdatePostInput!): Post!
      @requireAuth(roles: ["admin"])
    deletePost(id: Int!): Post! @requireAuth(roles: ["admin"])
  }
`
```

Update Posts SDL:

```tsx title="api/src/graphql/posts.sdl.ts"
export const schema = gql`
  type Post {
    id: Int!
    title: String!
    body: String!
    createdAt: DateTime!
    user: User!
  }

  type Query {
    posts: [Post!]! @skipAuth
    post(id: Int!): Post @skipAuth
  }
`
```

Create AdminPosts Service:

```tsx title="api/src/services/adminPosts/adminPosts.ts"
import { ForbiddenError } from '@redwoodjs/graphql-server'

import { db } from 'src/lib/db'

const verifyOwnership = async ({ id }) => {
  if (await adminPost({ id })) {
    return true
  } else {
    throw new ForbiddenError("You don't have access to this post")
  }
}

export const adminPosts = () => {
  return db.post.findMany({ where: { userId: context.currentUser.id } })
}

export const adminPost = ({ id }) => {
  return db.post.findFirst({
    where: { id, userId: context.currentUser.id },
  })
}

export const createPost = ({ input }) => {
  return db.post.create({
    data: { ...input, userId: context.currentUser.id },
  })
}

export const updatePost = async ({ id, input }) => {
  await verifyOwnership({ id })

  return db.post.update({
    data: input,
    where: { id },
  })
}

export const deletePost = async ({ id }) => {
  await verifyOwnership({ id })

  return db.post.delete({
    where: { id },
  })
}
```

Update Posts Service:

```tsx title="api/src/services/posts/posts.ts"
import { db } from 'src/lib/db'

export const posts = () => {
  return db.post.findMany()
}

export const post = ({ id }) => {
  return db.post.findUnique({ where: { id } })
}

export const Post = {
  user: (_obj, { root }) =>
    db.post.findFirst({ where: { id: root.id } }).user(),
}
```

Update the scaffold components:

```tsx title="web/src/components/Post/EditPostCell/EditPostCell.tsx"
export const QUERY = gql`
  query EditPostById($id: Int!) {
    post: adminPost(id: $id) {
      id
      title
      body
      createdAt
    }
  }
`
```

```tsx title="web/src/components/Post/PostCell/PostCell.tsx"
export const QUERY = gql`
  query FindPostById($id: Int!) {
    post: adminPost(id: $id) {
      id
      title
      body
      createdAt
    }
  }
`
```

```tsx title="web/src/components/Post/PostsCell/PostsCell.tsx"
export const QUERY = gql`
  query POSTS {
    posts: adminPosts {
      id
      title
      body
      createdAt
    }
  }
`
```

### Update Comment Delete Functionality

```tsx title="web/src/components/Comment/Comment.tsx"
import type {
  Comment as IComment,
  DeleteCommentMutation,
  DeleteCommentMutationVariables,
} from 'types/graphql'

import type { TypedDocumentNode } from '@redwoodjs/web'
import { useMutation } from '@redwoodjs/web'

import { QUERY as CommentsQuery } from 'src/components/CommentsCell'
import { useAuth } from 'src/auth'

const DELETE: TypedDocumentNode<
  DeleteCommentMutation,
  DeleteCommentMutationVariables
> = gql`
  mutation DeleteCommentMutation($id: Int!) {
    deleteComment(id: $id) {
      postId
    }
  }
`

const formattedDate = (datetime: ConstructorParameters<typeof Date>[0]) => {
  const parsedDate = new Date(datetime)
  const month = parsedDate.toLocaleString('default', { month: 'long' })
  return `${parsedDate.getDate()} ${month} ${parsedDate.getFullYear()}`
}

interface Props {
  comment: Pick<IComment, 'postId' | 'id' | 'name' | 'createdAt' | 'body'>
}

const Comment = ({ comment }: Props) => {
  const { hasRole } = useAuth()
  const [deleteComment] = useMutation(DELETE, {
    refetchQueries: [
      {
        query: CommentsQuery,
        variables: { postId: comment.postId },
      },
    ],
  })

  const moderate = () => {
    if (confirm('Are you sure?')) {
      deleteComment({
        variables: { id: comment.id },
      })
    }
  }

  return (
    <div className="bg-gray-200 p-8 rounded-lg relative">
      <header className="flex justify-between">
        <h2 className="font-semibold text-gray-700">{comment.name}</h2>
        <time className="text-xs text-gray-500" dateTime={comment.createdAt}>
          {formattedDate(comment.createdAt)}
        </time>
      </header>
      <p className="text-sm mt-2">{comment.body}</p>
      {hasRole('moderator') && (
        <button
          type="button"
          onClick={moderate}
          className="absolute bottom-2 right-2 bg-red-500 text-xs rounded text-white px-2 py-1"
        >
          Delete
        </button>
      )}
    </div>
  )
}

export default Comment
```

```
