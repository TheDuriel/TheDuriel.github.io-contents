This page aims to collect common pitfalls and trivia, which may be useful in debugging more the esoteric issues that can arise.

### Nylon is Asynchronous

Nylon processes nodes asynchronously from Godots process/physics process. This means messages should be treated like InputEvents, as they can be emitted at any point during a frame. Furthermore, new messages will continue to be emitted even while you may be awaiting or executing deferred calls in response to a message.

As an example, if you were to await in a function triggered by message A, message B will still be emitted and may trigger a new function call before message A has been processed. You may wish to queue messages if this becomes an issue for you.