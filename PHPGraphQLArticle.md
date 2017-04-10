# Diving into a PHP GraphQL Server. 

# Introduction

### Is this for me?

In this article, we will show the development of a new feature in a GraphQL server built in PHP. Our goal is to explore the architecture used on the PHP server by adding a small functionality to the application. 

If you are considering to set up a GraphQL server in PHP, you will sure find a good example here. When I was studying to build this app, I felt that there were missing code examples on the community, especially in PHP.

It's worth noticing the code we are going to explore is open sourced and you can also use it as your own foundation. In there you have a lot of code examples to look that might be usefull for your own study. 

As they say, **a repo is worth a thousand words**!

- [PHP backend repository](https://gitlab.com/bruno.p.reis/nosso-jardim) with Symfony, Doctrine and Overblog GraphQL

Don't take the example here as "Best Practices". It's impossible to talk about "Best Practices" in a so fast evolving technology scenario, especially for GraphQL based libs, since it was released two year ago. But, I sure did a good work researching and finding better ways to do things.

* BTW, the client application is also open sourced [here](https://gitlab.com/bruno.p.reis/nosso-jardim-client).

### Decisions taken

Please, allow me an overview before getting into the code. 

I was studying Apollo Client to integrate data into my React Frontend. Apollo is a very recent technology, build by the Meteor Development Group. It's a client for JS applications. In my specific case, for a React application. 

React and Redux make a great pair for frontend development, but there is a fundamental missing point, not in their scope, that is the data integration with server. To be more specific, the assynchronous integration. 

And that is where Apollo rules. It has a great set of tools like queries, caching, mutations, optimistic UI, subscriptions, pagination, server-side rendering, prefetching, and more. You can [check it out on their site](http://dev.apollodata.com/) if you wish. 

And, to use Apollo, I had to learn [GraphQL](http://graphql.org/learn/) because Apollo is built to be backed by a GraphQL server. 

I usually use PHP and Symfony on the backend, so that was my natural choice. 

So I went in a quest to see if I could find good libs in PHP to help on the job. And, lucky me, thanks fot these both projects ([1](https://github.com/webonyx/graphql-php),[2](https://github.com/Youshido/GraphQL)) I found mature libs that do their job in a excelent manner. 

I started using Youshido's lib and later migrated to Webonyx. To integrate with symfony, I used the [OverblogGraphQLBundle](https://github.com/overblog/GraphQLBundle). Having that GraphQL server layer in place, meanwhile I was evolving with the app, I tried a lot of layer compositions and code organizations to find a balanced and clean one. 

After a lot of refining, I was then able to come up with a simple way to structure the app, putting most of my focus on the business. This structure looks even better to me than tradicional ones with controllers and routes. 

And that was a fantastic bonus to me. I was looking for a GraphQL server with the only purpose to serve the frontend, but found a lot more. A good code structure, easily testable, and very well documented by default. 

Talking about tests, another decision taken was to run most tests over the API layer, what also proved to be a very good decision. In fact, my working proccess of a new feature now allways starts from the schema going to the tests on the top of that schema. Those are the beginning step prior to actual implementation of the features.  

I'll guide you through the App in the same order I use to develop a new feature. So that you can understand the app and also the proccess that is working fine for me. 

After we see a complete development cycle we will also take a look at some technical details. In special those required to integrate the libs we are using together. 

### The App 

The view is usually the easiest part to understand an application domain. So let's look at it before we install the backend and look at it's API. 

We are gonna work on a Knowledge Management App. The main purpose of this app is to organize knowledge that is shared through Telegram and other channels in the future. 

The mais screen, as shown below, is where you organize messages in specific threads or conversations and put tags on those threads. 

![Create a Thread and Tag it!](./images/createThread.gif)

This App gives us interesting data structures to serve us as an example:

1. A paginated list - Messages
2. A non paginated list - Threads
3. A tree - Subtopics (not shown in the gif)
4. Lots of simple views, menus and buttons.
5. Forms and data edition - Add Tag, Edit Subtopic, Add Subtopic, and so on...

These examples are all on the repository and can demonstrate with code how to handle these data structures on both front and backend. 


### Installation

Code and images are available through the article. But, I encourage you to clone the repo anyway, to play with it and look at the rest of the code available there. 

1. [Clone the backend repo.](https://gitlab.com/bruno.p.reis/nosso-jardim)
2. Run composer install and configure your parameters
3. Import Fixtures and Data
4. Start the PHP server
5. Read the next section so that you can look at GraphiQL knowing the app

TODO: rewrite readme in english to help these steps

## The Development Cycle

This are the steps I usually take when I add a new functionality to the system:

1. Define the functionality
2. Define the Schema
3. Write test(s)
4. Making test pass - Improving the schema and the resolver
5. Refactor

### Defining the functionality

Our task will be taking this: 

<img src="./images/tagFormBefore.png" width="600">

And turning into this: 

<img src="./images/tagFormAfter.png" width="600">

So  we are goint to add extra information about the classification (tagging) of a thread in a specific subtopic. 

Let's look at GraphiQL. GraphiQL is a tool where you can see the documentation of a GraphQL server and also run queries and mutations in it. If you started the server it should be running under 127.0.0.1:8000/graphiql or any similar location. 

Let's see the mutation that is used to insert an Information. Information is an entity in our system. It's the relationship between a Thread and a Subtopic. You can also understand it as a tag. 

The mutation is called "informationRegisterForThread". 

<img src="./images/informationRegisterForThreadBefore.png" width="400">

You can see it expects a required id (ID!) and also an InformationInput object. If you click on that InformationInput, you will see it's schema: 

<img src="./images/informationInputBefore.png" width="400">

> BTW, I like putting the noun before the verb in order to aggregate mutations on the docs. That's a workaround I've found due to the non nested characterist of mutations. 

It might seem funny or unnecessary to have a nested InformationInput object into those args. Specially because it now contains only one subtopicId field. This is, indeed, a good practice when [designing a mutation](https://dev-blog.apollodata.com/designing-graphql-mutations-e09de826ed97) because you reserve names for future expansion of the schema and also simplify the API on the client. 

And this will help us now. 

We need to add a new input field to that mutation, to register that extra 'about' text, and we can add that field inside our InformationInput object. So let's start by changing our schema: 

```json
#InformationInput.types.yml

# from ...

InformationInput:
    type: input-object
    config:
        fields:
            subtopicId:
                type: "ID!"

# to ...

InformationInput:
    type: input-object
    config:
        description: "This represents the information added to classify or tag a thread"
        fields:
            about:
                type: "String"
                description: 'Extra information beyond the subtopic'
            subtopicId:
                type: "ID!"
                description: 'The subtopic that represents where in the knowledge tree we classify the thread.'

```

We have added the 'about' field. We also improved docs with 'description' fields. Let's see our docs now: 

<img src="./images/informationInputAfter.png" width="400">

If you click on "about" and "subtopicId", you will be able to read the descriptions added for those fields. Ain't that beautiful? We are writting our app and writting our API docs at the same time in the exact same place. Cool!

Now we may add a new field 'about' when calling our mutation. Since our new field is not mandatory, our app should still be running just fine. 

Our schema is created. Before we actually implement the saving of that data, what about a little TDD? 

The scema is easily visible on the frontend and I feel it's ok to write it without any tests. But the resolver action is something that sure deserves a test to help us move faster. 

### Writing tests

Most of my tests run against the GraphQL layer. Doing so, so they also test the schema because if some wrong data is sent, errors will be returned. 

To run the test, I'm needing to clear the cache everytime, so I'm running this line: 

```
bin/console cache:clear --env=test;phpunit tests/AppBundle/GraphQL/Informations/Mutations/InformationRegisterForThreadTest.php
```

Let's write our test to add the 'about' data in our query and see if it is returned back when we read the thread with it's informations. We already have a test in place for that Mutation. Let's look at it to align our understanding on how we are testing: 

```php 
    # Tests\AppBundle\GraphQL\Informations\Mutations\InformationRegisterForThreadTest

    function helper() {
        return new InformationTestHelper($this);
    }

    /** @test */
    public function shouldSaveSubtopicId()
    {
        $h = $this->helper();

        $s1 = $h->SUBTOPICS_REGISTER_FIRST_LEVEL(['name'=>"Planta"])('0.subtopics.0.id');
        $s2 = $h->SUBTOPICS_REGISTER_FIRST_LEVEL(['name'=>"Construcao"])('0.subtopics.1.id');

        $t1 = $h->createThread();
        $t2 = $h->createThread();
        $t3 = $h->createThread();

        $h->INFORMATION_REGISTER_FOR_THREAD([
                'threadId'=>$t1,
                'information'=>['subtopicId'=>$s1]
        ]);

        $h->INFORMATION_REGISTER_FOR_THREAD([
                'threadId'=>$t1,
                'information'=>['subtopicId'=>$s2]
        ]);

        $h->INFORMATION_REGISTER_FOR_THREAD([
                'threadId'=>$t3,
                'information'=>['subtopicId'=>$s2]
        ]);

        $informations = $h->THREAD(['id'=>$t1])('thread.informations');

        $this->assertCount(2,$informations);
        $this->assertEquals($s1,$informations[0]['subtopic']['id']);
        $this->assertEquals($s2,$informations[1]['subtopic']['id']);
        
        $informations = $h->THREAD(['id'=>$t2])('thread.informations');
        $this->assertCount(0,$informations);

        $informations = $h->THREAD(['id'=>$t3])('thread.informations');
        $this->assertCount(1,$informations);

        $this->assertEquals($s2,$informations[0]['subtopic']['id']);
    }
*/ 
```

The helper (InformationTestHelper) is responsible by calling the queries on the GraphQL layer and return a function. It returns a function so that we can call it with a json path to grab what we need. This pattern, function returning a function, may seem a little tricky at first, but it pays the cost with the clarity we get from it. 

Let's refactor a little and you will see what I'm talking about: 


```php
/** @test */
    
    function createThreadsAndSubtopics() {
        $h = $this->helper();

        $s1 = $h->SUBTOPICS_REGISTER_FIRST_LEVEL(['name'=>"Planta"])('0.subtopics.0.id');
        $s2 = $h->SUBTOPICS_REGISTER_FIRST_LEVEL(['name'=>"Construcao"])('0.subtopics.1.id');

        $t1 = $h->createThread();
        $t2 = $h->createThread();
        $t3 = $h->createThread();

        return [$s1,$s2,$t1,$t2,$t3];
    }

    public function shouldSaveSubtopicId()
    {
        $h = $this->helper();

        list($s1,$s2,$t1,$t2,$t3) = $this->createThreadsAndSubtopics();

        $h->INFORMATION_REGISTER_FOR_THREAD([
                'threadId'=>$t1,
                'information'=>['subtopicId'=>$s1]
        ]);

        $h->INFORMATION_REGISTER_FOR_THREAD([
                'threadId'=>$t1,
                'information'=>['subtopicId'=>$s2]
        ]);

        $h->INFORMATION_REGISTER_FOR_THREAD([
                'threadId'=>$t3,
                'information'=>['subtopicId'=>$s2]
        ]);

        list(
            $informations,
            $s1ReadId,
            $s2ReadId
        ) = $h->THREAD([
            'id'=>$t1
        ])(
            'thread.informations',
            'thread.informations.0.subtopic.id',
            'thread.informations.1.subtopic.id'
        );

        $this->assertCount( 2 , $informations );
        $this->assertEquals( $s1 , $s1ReadId );
        $this->assertEquals( $s2 , $s2ReadId );
        
        
        $this->assertCount(
            0,
            $h->THREAD(['id'=>$t2])('thread.informations')
        );

        $informations = $h->THREAD(['id'=>$t3])('thread.informations');
        $this->assertCount(1,$informations);

        $this->assertEquals($s2,$informations[0]['subtopic']['id']);
    }
```

So this:
```php
$informations[1]['subtopic']['id']
```

Now is returned as 
```php
$s2ReadId
```
in response to the query 
```php
'thread.informations.1.subtopic.id'
``` 
that was made through JsonPath into the response that came fron the GraphQL layer. Now that you know how tests are working, let's test for the new field we are going to add. 

We will register an information with data in the 'about' field. After that we will load that thread back, query that field's value and assert it is equal to the original string. 

```php
    # Tests\AppBundle\GraphQL\Informations\Mutations\InformationRegisterForThreadTest

    /** @test */
    public function shouldSaveAbout()
    {
        $h = $this->helper();

        list($s1,$s2,$t1) = $this->createThreadsAndSubtopics();

        $h->INFORMATION_REGISTER_FOR_THREAD([
                'threadId'=>$t1,
                'information'=>[ # information field
                    'subtopicId'=>$s1,
                    'about'=>'Nice information about that thing.' # about field
                ]
        ]);

        $savedText = $h->THREAD(['id'=>$t1])( # query for the thread
            'thread.informations.0.about' # query the result for that field
        );

        $this->assertEquals( 'Nice information about that thing.' , $savedText ); # check it 
    }
```

Now, let's follow in a TDD way, running the test and answering to it's requests. 


### Making test pass - Improving the schema and the resolver

Run this test and it will fail saying that it could not query the 'about' field on the response returned by the THREAD query. So let's add it there. 

```php 
    # Tests\AppBundle\GraphQL\TestHelper.php

    function THREAD($args = [],$getError = false, $initialPath = '') {
        $q ='query thread($id:ID!){
                thread(id:$id){
                    id
                    messages{
                        id
                        text
                    }
                    informations{
                        id
                        about # <-- ADDED THIS FIELD
                        subtopic{
                            id
                        }
                    }
                }
            }';
        return $this->runner->processResponse($q,$args,$getError,$initialPath);
    }
```

Doing so, I get this error: 

```
    [message] => Cannot query field "about" on type "Information".
```

And that's correct. We added 'about' to the InformationInput type. Now we need to add it to the Information GraphQL type:

```yml
#Information.types.yml
            about:
                type: "String"
                description: "extra information" 
```

Now that Information has the 'about' field, we run the test and get a: 

```
Failed asserting that null matches expected 'Nice information about that thing.'
```

So, we can understand GraphQL is ok, because we did not get any validation errors from the data being sent or retreived back. So we can work on the resolvers now. 

In order to complete our mission, we need to 

1. save it when the mutation is received and 
2. read it on the THREAD query.   

```php
# 1 - AppBundle\Resolver\InformationsResolver

    public function registerForThread($args)
    {
        $thread = $this->repo('AppBundle:Thread')->find( $args['threadId'] );
        $subtopic = $this->repo('AppBundle:Subtopic')->find( $args['information']['subtopicId'] );
        $about = $args['information']['about'];
        $info = new Information();
        $info->setSubtopic($subtopic);
        $info->setAbout($about);
        $thread->addInformation($info);
        $this->em->persist($info);
        $this->em->flush();
        $this->em->refresh($thread);
        return $thread;
    }

#AppBundle\Entity\Information

    @ORM\Column(name="about", type="text", nullable=true)
````

And then, we get the very wanted green message we were waiting for passing the test.

> I encourage you to open SubtopicsTestHelper and follow and understand the 'proccessResponse' method (Reis\GraphQLTestRunner\Runner\Runner::processGraphQL). 
> There you will be able to see the GraphQL call happening and the json path component being wrapped in the returning funcion. 

### Refactor

#### Observe

Having our green lights on, it's time for some retrospective on what we've done so far. Going to a higher to a lower level, lets talk first about the proccess. 

One of the benefits I see using GraphQL is to be able to think functionality from an API perspective first. Please notice we just touched the db layer at the very end, when we were sure our app would attend the outter world expectations, with the safety validations net in place. 

When we wrote our resolver, we could be confident that we were receiving good data, passed through a typed validation system, and also that the data we returned was being checked with the same criteria. 

[This nice article about 'GraphQL first'](https://dev-blog.apollodata.com/graphql-first-a-better-way-to-build-modern-apps-b5a04f7121a0) from DeBergalis is a nice reference talking about this way of developing starting from the contract. 

Having used it now for a while, I can add to the voices saying it will save you unnecessary work and headaches. And give you code that fulfill client's expectations in a more precise way, since you already start from there.

So, what have we done? 

1. Understood our functionality
2. Defined the schema
3. Wrote our test
4. Wrote the resolver
5. Updated doctrine

It might seem as too many steps for some people, but to me they are very nice steps because they have very well defined purposes. 

1. Know what you are doing ( :-) ) 
2. Define the contract and put validation in place
3. Define how we want it to behaviour
4. Implement the logic
5. Implement the persistence

#### Improve 

So, to honor the refactor step, looking at the code, is there anything that can be improved? Well, there is allway something. But, we should allways start from the lower hanging ones, right? 

I was reading this article on [how to design mutations](https://dev-blog.apollodata.com/designing-graphql-mutations-e09de826ed97) and that approach seemed very nice to me. All the recommendations make sense, in our case, especially the one about allways having a specific return type exclusively baked for your mutation. 

Until now we have: 

```yml
#Mutation.types.yml
            informationRegisterForThread:
                type: "Thread"
                args:
                    threadId:
                        type: "ID!"
                    information: 
                        type: "InformationInput"
                resolve: "@=service('app.resolver.informations').registerForThread(args)"
```

The result type is the "Thread" type. This gives us almost none flexibility to add or remove informations that might be needed there. So let's bake a type for us: 

```yml
# Mutation.types.yml - at the first level, after Mutation root field

InformationRegisterForThreadResult:
    type: object
    config:
        fields:
            thread:
                type: "Thread"
```

From now on we nested our response inside threadField. Well, an extra level does not seem a good idea just for fun. But, that will give us a lot of flexibility. 

Imagine, for example, I want to allow this mutation to return extra information about an aggregated value. Let's say, a count of how many Threads have being tagged into that same subtopic. 

Adding those fields in a flat response would be very confusing. But now we can just add a informationStatus field on the InformationRegisterForThreadResult type and that will be available and well organized. 

Such a field could help us saving a request to grab this extra information and keeping the UI in synch with only one request that would do the mutation, return the thread with it's informations and also return those status. 

Long life to GraphQL!

## Big Components

So, now that we dived in the system adding this functionality, we can have a better view of the big components of our architecture: 

The GraphQL layer, implemented using the OverblogGraphQLBundle. 
    - We define the GraphQL schema and expose an endpoint.
    - All queries and mutations will enter through that endpoint. 
    - Validation will be run, using a strict type system. 
    - Execution will run through resolvers. 

The testing layer. 
    - Ok, I know tests are not strictly a layer on the architecture. But I want to leave them here to remind you that you can call those GraphQL queries using the tests and implement a safety net on your system. Also tests can serve as a very good documentation.  

The resolvers layer, implementing business logic. 
    - Those are simple PHP classes registered as services
    - We map them using the Expression Language in the schema definition. 
    - Resolvers receive validated args and return data. 
    - They alter data when running mutations. 
    - That return data is also validated by the GraphQL layer in it's way back. 
    - Usually resolvers will call a data layer like Doctrine to do their job. 
    - But they can also call a lot of different services, even a rest call can be done there. 

The data layer, specific to your application. 



## Extra pieaces. 

In the last Section, "The Development Cycle", we understood (I hope ;-)) how this architecture works in a dynamical way. 

In this section I'll tell some extra details about specific libs and configurations I fell are important to finalise our job here. What job? Transmitting to you this architecture and enabling you to know what each part is for, how it serves the whole and giving you a way to judge if you need it or not. 

### Overblog GraphQL 

To build the GraphQL server over symfony, we are using the Overblog GraphQL [Bundle](https://github.com/overblog/GraphQLBundle).

It's a very good lib that integrate the GraphQL lib into symfony adding nice features to it. 

One very special feature it has is the [Expression Language](https://github.com/overblog/GraphQLBundle/blob/master/Resources/doc/definitions/expression-language.md). BTW, that's a missing point on the upper section, the secret place where you map this: 

```yml
    # Mutation.types.yml
    resolve: "@=service('app.resolver.informations').registerForThread(args)"
```

To the call to registerForThread method into the service registered as 'app.resolver.informations'. It also improves error handling, implement endpoints and more...

As you can see on it's [requirements](https://github.com/overblog/GraphQLBundle/blob/master/composer.json), it is built over cool libs. I want to especially note these two: 

	1 - overblog/graphql-php-generator - This is responsible for reading a nice and clean yml and convert it into the GraphQL type objects. It will all happen under the hood and all you will need to touch is the yml and the resolvers. 

	2 - webonyx/graphql-php - This is the real engine of our car. A PHP port of GraphQL reference implementation. Very stable and ready to use. Please take a look at the docs and I call special attention to the 

### The Schema Declaration - The entrance to the backend

Our schema declaration is under src/AppBundle/Resources/config/graphql/
Queries are defined in the Query.types.yml and mutations on Mutations.types.yml. 

The resolvers are nicely defined there with the expession language we've talked about. You should know that, to fulfill a request, more than one resolver can be called down in the scema tree. 

This file is the entrance on our system. It defines the API interface with the outter word. 

### Cors

This app is not on a prod server yet, but I intend to keep the server in a different env, so I installed the [NelmioCorsBundle|https://github.com/nelmio/NelmioCorsBundle]

The configurations today are very open and will need to be a lot more string on a prod server. But, I just wanted you to note that it's running and will help you to avoid a lot of errors seen on the frontend client. 

It's also worth noticing that I had to add 'content-type' as an allowed header in it's config. 

If you don't know this bundle yet, it's worth taking a look at it. It will manage the headers sent and specially the OPTIONS pre-flight requests to enable cross origin resource sharing. In other words, will enable you to call your API from a different domain.

### Error Handling

Error handling is a very open topic on GraphQL world. The specs don't say a lot ([1](https://facebook.github.io/graphql/#sec-Errors),[2](https://facebook.github.io/graphql/#sec-Executing-Operations)) and are open to a lot of interpretations. And, as we can see by the community, there are a lot of different opinions on how to handle them ([1](https://voice.kadira.io/masking-graphql-errors-b1b9f15900c1),[2](https://medium.com/@tarkus/validation-and-user-errors-in-graphql-mutations-39ca79cd00bf))

Being GraphQL so recent, it's expected that you don't have well established best practices on it yet. And this is something nice to take into consideration when designing your system. Maybe you see a better way of doing things. So, please do it, test, and share with us. 

Overblog deals with errors in a manner that is good enough me. First, it will add normal validation errors when it encounter them on the data validation of input or output types. 

Second, it will handle the exceptions thrown in the resolvers. When it catches an exception, it will add an error to the response. Almost all exceptions are added as an "internal server error" generic message. The only two Exception types (and subtypes) that are not translated to this generic message are: 

- ErrorHandler::DEFAULT_USER_WARNING_CLASS
- ErrorHandler::DEFAULT_USER_ERROR_CLASS

These can be [configured](https://github.com/overblog/GraphQLBundle/blob/master/DependencyInjection/Configuration.php#L101) on your config.yml with your own exceptions: 

```yml
overblog_graphql:
    definitions:
        exceptions:
            types:
                errors: "\\AppBundle\\Exceptions\\UserErrorException"
```

To understand it a little deeper, please put your diving mask and look at the class that implements this handling: "Overblog\GraphQLBundle\Error\ErrorHandler"

The ErrorHandles is also responsible to put the stack trace on the response. But, it will only do it if the symfony debug is turned on. That is normally done on the "$kernel = new AppKernel('dev', true);" call. 

I encourage you to test sending a non existent id to that query with debug==true and with debug==false and seing the response. You can do that on GraphiQL. 

Exceptions are also logged on dev.log.  

### Final Considerations

I'm glad you made it. I know this article is larger than usual. I had so many insights and sgruggles to put this all together that, in fact, I had to cut a lot of paragraphs and content to make this. 

My first text version also included the frontend with Apollo Client and React Semantic and optimization as big class topics. I also know there are some missing important points to talk about, like pagination, connections, user authorization and authentication. 

So, please add a comment with your feedback, speacially saying where I can improve. In all aspects. Was it usefull to you? Do you want me to talk about something else? Is the line or tought clear? 

Thanks for reading this far. I hope I contibuted with your discoveries about GraphQL. 