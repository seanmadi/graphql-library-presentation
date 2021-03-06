### Comparing GraphQL Client Libraries in React
#### Apollo and Relay Modern

---

### Why use a GraphQL Client Library?
* Constructs HTTP request with query and variables
* Pluggable with middlewares and afterwares
* Makes subscriptions easy
* Caching, keep UI consistent
* Integrate with React components

---

### Apollo
* Much lower learning curve
* Fragments provided to queries via string interpolation
* Not strict about structure, flexible
* Queries can be executed anywhere
* Built on top of redux

---

### Relay Modern
* Uses GraphQL Relay Specification for queries
* Faster from better caching
* Uses a compiler, relay-compiler, that introduces a build step
* Colocate fragments with components, called containers
* Spread fragments are already in scope
* Strict about structure and naming conventions
* Use QueryRender component to execute queries
* Doesn't play well with redux

---

### Apollo Setup
Add dependencies
```
yarn add apollo-client
yarn add react-apollo
yarn add graphql-tag
```

---

### Setup client
App.js
```javascript
import ApolloClient, { createNetworkInterface } from 'apollo-client'
import { ApolloProvider } from 'react-apollo'

const networkInterface = createNetworkInterface({
  uri: '__API_ENDPOINT__'
})
const client = new ApolloClient({
  networkInterface
})

class App extends Component {
  render() {
    return (
      <ApolloProvider client={client}>
        <ListPage />
      </ApolloProvider>
    )
  }
}
```

---

### Add Query to ListPage
ListPage.js
```javascript
import { graphql } from 'react-apollo'
import gql from 'graphql-tag'

const FeedQuery = gql`query allPosts {
  allPosts(orderBy: createdAt_DESC) {
    id
    name
  }
}`

class ListPage extends Component {
  render () {
    const { data: { loading, allPosts, refetch }} = this.props
    if (loading) {
      return (<div>Loading</div>)
    }
    return (
      <div className="container">
        <div className="posts">
          {allPosts.map((post) =>
            <Post key={post.id} post={post} refresh={() => refetch()} />
          )}
        </div>
      </div>
    )
  }
}

export default graphql(FeedQuery)(ListPage)
```

---

### Post Component
Post.js
```javascript
class Post extends Component {
  render () {
    const { id, name } = this.props.post
    return (
      <div key={id} className="post">
        <h3>{name}</h3>
      </div>
    )
  }
}
```

---

### Relay Modern Setup
Add dependencies
```
yarn add react-relay
yarn add relay-compiler --dev
yarn add babel-plugin-relay --dev
```
Add Relay plugin to Babel
```javascript
"babel": {
  "presets": [
    "react-app"
  ],
  "plugins": [
    "relay"
  ]
},
```

---

### Setup the Environment
```javascript
import {
  Environment,
  Network,
  RecordSource,
  Store,
} from 'relay-runtime'

const store = new Store(new RecordSource())

const network = Network.create((operation, variables) => (
  fetch('__API_ENDPOINT__', {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      query: operation.text,
      variables,
    }),
  }).then(response => (
    response.json()
  ))
))

const environment = new Environment({
  network,
  store,
})

export default environment
```

---

### Create Fragment Containers
Post Component
```javascript
import {
  createFragmentContainer,
  graphql
} from 'react-relay'

class Post extends Component {
  render() {
    return (
      <div>{this.props.post.description}</div>
    )
  }
}

export default createFragmentContainer(Post, graphql`
  fragment Post_post on Post {
    id
    description
    imageUrl
  }
`)
```

---

### Create Fragment Containers
Post List Component
```javascript
import {
  createFragmentContainer,
  graphql
} from 'react-relay'

class ListPage extends Component {
  render() {
    return (
      {this.props.viewer.allPosts.edges.map(({node}) =>
        <Post key={node.__id} post={node} />
      )}
    )
  }
}

export default createFragmentContainer(ListPage, graphql`
  fragment ListPage_viewer on Viewer {
    allPosts(last: 100, orderBy: createdAt_DESC) @connection(key: "ListPage_allPosts", filters: []) {
      edges {
        node {
          ...Post_post
        }
      }
    }
  }
`)
```

---

### Use Query Renderer to compose the query
App.js
```javascript
import {
  QueryRenderer,
  graphql
} from 'react-relay'
import environment from './Environment'

const AppAllPostQuery = graphql`
  query AppAllPostQuery {
    viewer {
      ...ListPage_viewer
    }
  }
`
class App extends Component {
  render() {
    return (
      <QueryRenderer
        environment={environment}
        query={AppAllPostQuery}
        render={({error, props}) => {
          if (error) {
            return <div>{error.message}</div>
          } else if (props) {
            return <ListPage viewer={props.viewer} />
          }
          return <div>Loading</div>
        }}
      />
    )
  }
}
```

---

### Compile your GraphQL queries
Get your GraphQL server schema
```
npm install -g get-graphql-schema
get-graphql-schema __RELAY_API_ENDPOINT__ > ./schema.graphql
relay-compiler --src ./src --schema ./schema.graphql
```
Relay-compiler compiles queries into Javascript lazy requires located at
```
./src/__generated__
```

---

# Questions?
