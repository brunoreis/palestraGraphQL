- [JS frontend with Apollo Client, React Apollo, React and Semantic UI React](https://gitlab.com/bruno.p.reis/nosso-jardim-client)

# Frontend

## Driving Forces

React is a very nice frontend technology and has some paradigm shifts for a frontend tech. **It's beauty lies in the fact that you always have a predictable view for a specific model state.** The hadaches with a lot of binds and listeneres are mostly gone and the code is a lot more clean and therefore maintainable. 

Talking in other terms, a React frontend is a pure function where you pass a model and get a view. Together with Redux, that has a similar approach for state management, they make a very cool tool for the frontend apps, but something was missing.  

**The R & R couple (React and Redux) still lack a good way to integrate with assynchronous data**. So, I started a quest on what to use there. I researched a lot of redux patterns and libs to do this job, but they all seemed to require to much to do the job. 

Looking for solutions, I was hearing a lot of buzz about Relay, and I decided to give it a try. Specially because I like the facebook policy of releasing code they actually use. So I started studying relay.  

Relay docs were too cryptic to me and I gave up on it on my first try. A little later, I met Apollo, backed by the meteor guys. I found they had a better documentation and community, so I decided to give it a try.

I started my first app using a GraphQL client. Using it **I discovered that Apollo was able to do a lot more than just fetching data from the server. It's cache layer is very well managed, it is able to normalize and denormalize data into there, it can manage subscriptions and also can agregate queries to save requests.**  

Using it I was able to mantain a very nice and organised codebase, backed by a very powerfull tool to integrate assynchronous data with my view. **To me, the declarative way of composing the views with HOCs wrapping the views fitted like a glove in my hands.**
## Installing the frontend

The frontend is a completely separated repo and can even be hosted in a completely different environment than the backend. Maybe a CDN?! Please, clone it now and make it run on your machine. 

1 - clone the repository
2 - npm install
3 - go to your browser

If you take a look at scripts/start.js you will see that it uses WebpackDevServer. On my machine it will start on localhost:3000. The symfony server, started with server:run, runs into port 8000 and I was getting a lot of cors issues.

## Overview

You start wetting your feet and dressing your flippers and diving mask again. The next dive will be at the frontend. So, before that, let's get a brief overview of the frontend. 

Views are built using react. React allows us to have an excelent and clear code with nice component composition. It has a virtual doom and, everytime the model changes, it will render the view again. But it will do it virtually first, calculate what is really needed to change on the actual browser dom, and, having made that "diff", it will just render/change those parts.

Most of the code is organized under src/routes, where you see the pages of our system. Inside these folders we have the ducks.js file that's where we concentrate most of the logic that belongs to redux. 

[Redux|http://redux.js.org/] is our frontend state managemnt tool. We use it to hold and manage frontend information. It will also pass a single state object to react, that will re-render itself on every change. 

That state management is indeed not so much used directly in this app, because we use a very powerful tool to ingegrate with the GraphQL server, that is the Apollo Client, to be more specific, the [React Apollo|http://dev.apollodata.com/react/].

Apollo will manage our data, and has a lot of usefull features you can check on their [home page|http://dev.apollodata.com/]. It allows us to wrap our react components using [HOC|https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e] to inject data and functions in it. 

To handle queries, Apollo uses a declarative api, where you define your query and it will inject the results into your component. You have a lot of very usefull configuration options that you can check on their [excelent docs|http://dev.apollodata.com/react/queries.html].

To handle mutations, Apollo uses a imperative approach, where you can also [configure|http://dev.apollodata.com/react/mutations.html] a lot of nice features. Injected mutations are promisses that you can configure with .then() calls to customize what will happen after they run. 

Mutations and Queries results are integrated into the cache layer on the frontend. After that, they are structured to server our components. A great feature on the system is that when the cache is updated, all the components that apollo can identify that use those values are automatically updated. This, to be honest, works almost like magic in a lot of cases. 

Ok, so we have React (react) + Apollo (apollo-client) + React Apollo (react-apollo) in the front. Another packages that are worth to note are the [Semantic|http://react.semantic-ui.com/introduction] one, that is responsible for the fantastic and easy to use frontend components, and the [react-router|https://reacttraining.com/react-router/], that is responsible for the frontend routing.

Ok. Enough talk. Let's dive in. 

## Our next destination: MessageStatus View Component

Ok, you can now blow your snorkel. We are gonna look at the MessageStatus component now. 

[img](https://raw.githubusercontent.com/brunoreis/palestraGraphQL/master/images/messageStatus.png)

This component has the total messages information and also has a combo where we can select the peer we want to use as a filter to list the messages below. BTW, just in case you don't speak brazilian portuguese, "mensagens" means "messages".

So, take a look at the MessageStatus.js code. It's stored in the routes/MessagesPage/components folder. It's a react component. And it's a simple function. So it will receive these props: 

```js
({
	selectedMessages,
	peers,
	peerId,
	setPeer,
	messagesStatus:ms,
	clearSelection
})
```

and return what will be rendered. The good catch here is that it will always return the same result for the same parameter values. So it behaves like a pure function, isolating the view logic and making it easily and isolated testable.


## Integrating data with containers

So far, it's "just" react! Blu sea, no big waves or strong sea currents. Here is where things start getting fun. If you look at the MessagesPage component, it doen not include MessageStatus, but instead, it includes the <MessageStatusContainer/> component. 

Containers are a special kind of components, where data integration and injections are made. They use HOCs to wrap react components and add behaviour to it. If you look at the MessageStatusContainer you can notice 4 hocs being added. 

```js
export default compose(
	connect(
	    (state,ownProps) => ({ 
	        selectedMessages: state.messages.selectedMessages,
	        peerId: state.messages.peerId
	    }),
	    (dispatch) => ({
	        clearSelection: () => dispatch(clearSelection()),
	        setPeer: (peerId) => dispatch(setPeer(peerId))
	    })
	),
	graphql(
        PEERS_QUERY,
        {
            props: ({data:{loading,error,peers}}) => {
                let ret = {
                    loading,
                    error,
                    peers,
                };
                return ret;
            }
        }
    ),
    graphql(
        MESSAGES_STATUS_QUERY,
        {
        	options: ({peerId}) => ({variables:{peerId:peerId}}),
            skip: (ownProps) => ownProps.peerId === '',
            props: ({data:{loading,error,messagesStatus}}) => {
                let ret = {
                    loading,
                    error,
                    messagesStatus,
                };
                return ret;
            }
        }
    ),
	displayLoaderAndError
)(MessageStatus)

```

The 'compose' method helps cleaning the syntax here, but what is really happening here is this: 

connect( (...)=>({...}) , (...)=>({...}) ) (
	graphql( PEERS_QUERY , {...} )(
		graphql( MESSAGES_STATUS_QUERY , {...} )(
			displayLoaderAndError(MessageStatus)
		)
	)
)

In other words: 

```
displayLoaderAndError is wrapped by the result of 

graphql( MESSAGES_STATUS_QUERY , {...} ) that is wrapped by the result of 

graphql( PEERS_QUERY , {...} ) that is wrapped by the result of

connect( (...)=>({...}) , (...)=>({...}) )
```

That has a practical effect that is that the props declared on connect are available to the peers_query, the props on the peers_query are available to the messages_status_query, and the props on the messages_status_query are available to the displayLoaderAndError wrapper. 

And, to finalize, props on the displayLoaderAndError wrapper, together with all the rest, are all available to our MessageStatus component. 

I know, I know. This is hard to understand at first time. Well, at least it was to me. Specially because functions that return functions [1|https://davidwalsh.name/javascript-functions], [2|https://en.wikipedia.org/wiki/Higher-order_function] is not a very common pattern. 

So, I made myself a little rule of thumb that is fantastic and very simple: Props flow from top to bottom. That's it!

Let's look at the peer list query first. 

## Diving deepen into the PEERS query

Take a breath and dive again. First, if you looked at the complete source code, you probably noticed the inclusion of that query: 

```
import PEERS_QUERY from '../queries/peers.graphql'
```

Let's look at that file: 

```
query peers{
    peers {
        id,
        printName,
        peerType
    }
}
```

That's a very simple file. It has only the GraphQL query. And this is a simple one. We just list all peers and that's it. Try pasting this on your graphiQL to see it's results. Talking about workflow of a new feature, after having a service on the server, testing the query on graphiQL is normally my next step. Even before wrapping the component. 

So that is passed to this call

```
graphql(
    PEERS_QUERY,
    {
        props: ({data:{loading,error,peers}}) => {
            let ret = {
                loading,
                error,
                peers,
            };
            return ret;
        }
    }
)
```

that will end wrapping the MessageStatus component. You should make a detailed read [here|http://dev.apollodata.com/react/queries.html] later, but to help you 
understand it now, all that is made there is to convert a {data:{loading,error,peers}} objec in a flat {loading,error,peers} objec before passing it down to displayLoaderAndError and then to MessageStatus. 

Well, that does not really matter now. The most important part here is to understand that we have wrapped our component with a declarative query. Why do I put too much enphasys on the "declarative" word here? So that you understand that the query is not necessarily happening at that moment. 

We declare that that query will be needed to our component. That's our job. Apollo will judge when to run that query. It might even not run it. How come? Will it just refuse to work!? No. That's sure not the case. But, Apollo has a caching layer and, depending on the [fetchPolicy|http://dev.apollodata.com/react/api-queries.html#graphql-config-options-fetchPolicy], Apollo might see that cache has all we need and then return the cached result to us, saving us a web request. 

Ok, undertanding that we now can look at the two other implications of our query declaration. 

1 - Apollo will query our server. 

Whenever it "feels" it should, Apollo will send that query to the server, grab the results back (the same you saw on graphiQL) and put it into the cache. When Apollo gets data from the server, it normalizes that data to save it on the cache. 

2 - "Peers" prop will be injected into our component. 

That's why we can expect for the 'peers' prop on MessageStatus. And, better than that, we can be sure of it's structure and types, right? 

If you trace the query to the server, you will see it's called on the peers field on the schema. Let's look at it: 

```php - Query.types.yml
			peers: 
                type: "[Peer]"
                resolve: "@=service('app.resolver.peers').find([value])"

```

The brackets wrapping the type declaration are telling we should expect an array of objects of the Peer type. 

The peer type is declared into Peer.types.yml: 

```yml
Peer:
    type: object
    config:
        description: "Chanels of comunication."
        fields:
            id:
                type: "ID!"
                description: "The id of the selected peer."
            printName:
                type: "String"
                description: 'Nome do grupo/outro participante da conversa'
            peerType:
                type: "String"
                description: "Original peer type on it's origin"
            peerId:
                type: "String"
                description: 'Original ID from where it was exported'
```

So, if the peers array returned from our graphQL server, and it schema declares this type for the Peer type, we can expect that it has being validated on the server and has the correct format an types. Having this safety net to help our work is very very helpful. Especially on that place, that is the client-server communication. 

## Integration between Apollo containers and also Redux

What else we can learn since we are already here? Since we've at this depth, let's look arount. Maybe we can find an oyster with a pearl. 

Let's take another look at the parameters received by MessageStatus and see what else we can learn by tracking them:

```js - MessageStatus.js
({
	selectedMessages,
	peers,
	peerId,
	setPeer,
	messagesStatus:ms,
	clearSelection
})

``` 

Let's look at our container again: 

```js
export default compose(
	connect(
	    (state,ownProps) => ({ 
	        selectedMessages: state.messages.selectedMessages,
	        peerId: state.messages.peerId
	    }),
	    (dispatch) => ({
	        clearSelection: () => dispatch(clearSelection()),
	        setPeer: (peerId) => dispatch(setPeer(peerId))
	    })
	),
	graphql(
        PEERS_QUERY,
        {
            props: ({data:{loading,error,peers}}) => {
                let ret = {
                    loading,
                    error,
                    peers,
                };
                return ret;
            }
        }
    ),
    graphql(
        MESSAGES_STATUS_QUERY,
        {
        	options: ({peerId}) => ({variables:{peerId:peerId}}),
            skip: (ownProps) => ownProps.peerId === '',
            props: ({data:{loading,error,messagesStatus}}) => {
                let ret = {
                    loading,
                    error,
                    messagesStatus,
                };
                return ret;
            }
        }
    ),
	displayLoaderAndError
)(MessageStatus)

```

So, 'peerId' and 'selectedMessages' are comming from the redux store. Not the Apollo one, but ours own store. BTW, it's configured on App.js, as explained [here|http://dev.apollodata.com/react/redux.html]. Connect will inject it up in the HOCs chain and it will be injected into our component. Please remember our rule of thumb: props flow from up to down. 

Also, 'setPeer' and 'clearSelection' are comming from connect. They are actions that can be dispatched to the store to change the peer and to clear the selection. Notice that this is the only touching point between our component and these actions/props. That's the main responsibility of the container, to server as this brigde between data and a react component. 

The last one is 'messagesStatus', that is injected by the the 'MESSAGES_STATUS_QUERY'. This is very similar to the peers query and we don't need to get into much detail here. But, we have two important details to note on that query configurations. 

The 'skip' config receives a function. If that returns true, that query is skipped. In practical terms it shows a way to not query for messages while no peer is selected. 

The 'options' config return an object whose 'variables' field contains the object that will be passed to the query to be executed. Let's look at that query declaration: 

```graphql - messagesStatus.graphql
query messagesStatus($peerId: String){
    messagesStatus (peerId: $peerId){
    	id,
        cleanMessagesCount,
        messagesCount,
        newestMessageDate,
        oldestMessageDate
    }
}

```
 
So, depending on the value of 'peerId', that query will look for messages of that channel. Back to the options code, let's see where that come from... 

```
options: ({peerId}) => ({variables:{peerId:peerId}}),
```

The function passed to options has this signature: (ownProps) => configObject. So, using the object spread operator, we extract the peerId from that object ownProps and pass it to the {variables:{peerId}}. I'm being purposely repetitive here, but, where did that peerId in ownProps came from? Again, props flow from top to bottom, or from outer to inner wrapped components, so, it's comming from connect, that grabs it from the store. 

So we just learned how to integrate a Redux managed prop into Apollo, right? This is something very usefull and one of the points I've struglled a lot on my learning journey on this. 

## Verification Point

Back to our boat. Let's take a breath. So far we've looked at the PHP Server, learned how it's structured, how it defines our schema and how to handle errors and write tests using our proposed architecture. After having that backend set up, we dived into the frontend. We learned about it's structure and components and looked closer at a query declaration. Being there, we also looked at how to integrate our normal redux store with apollo. 

So far the trip was fun, and it's gonna continue on that pace, if you have the guts to follow with me. We will now take a look at an existing mutation to understand how it works. And, to finalize our article, we will take a look at performance considerations and learn how to improve performance, keeping our flexibility to grow and refactor fast when needed. 


## Mutations - Let's improve our informations on the the thread 

We are going to work on the mutation that relates the subtopics to the threads. Now you can choose a subtopic and add it to a thread. You can't say anything else. We are going

[screen capture]

To write this mutations we are going to follow some [principles about designing good mutations|https://dev-blog.apollodata.com/designing-graphql-mutations-e09de826ed97] that made sense to me when I read about it. 





Let's go fishing - Building our First Mutation
===============================================

#Refactor mutations to use single arg and exclusive return type
# https://facebook.github.io/jest/

Material excelente sobre connections
https://dev-blog.apollodata.com/explaining-graphql-connections-c48b7c3d6976