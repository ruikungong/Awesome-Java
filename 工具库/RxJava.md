

Base classes
RxJava 2 features several base classes you can discover operators on:

* io.reactivex.Flowable: 0..N flows, supporting Reactive-Streams and backpressure
* io.reactivex.Observable: 0..N flows, no backpressure,
* io.reactivex.Single: a flow of exactly 1 item or an error,
* io.reactivex.Completable: a flow without items but only a completion or error signal,
* io.reactivex.Maybe: a flow with no items, exactly one item or an error.

