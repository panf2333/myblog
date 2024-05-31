# Promise

## State
https://thenewtoys.dev/blog/2021/02/08/lets-talk-about-how-to-talk-about-promises/

1. pending: 
   >the promise has been created, and the asynchronous function it's associated with has not succeeded or failed yet.<font color="red"> This is the state your promise is in when it's returned from a call to fetch()========asynchronous</font>, and the request is still being made.

2. fulfilled: 
   >the asynchronous function has succeeded. When a promise is fulfilled, its then() handler is called.
   
3. rejected: 
   >the asynchronous function has failed. When a promise is rejected, its catch() handler is called.

>Sometimes, we use the term settled to cover both fulfilled and rejected.

Note that what "succeeded" or "failed" means here is up to the API in question. For example, fetch() rejects the returned promise if (among other reasons) a network error prevented the request being sent, but fulfills the promise if the server sent a response, even if the response was an error like 404 Not Found.


A promise is resolved if it is settled, or if it has been "locked in" to follow the state of another promise.

https://thenewtoys.dev/blog/2021/02/08/lets-talk-about-how-to-talk-about-promises/