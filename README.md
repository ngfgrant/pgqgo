# PGQGO

`pgqgo` is a Go API that utilises native Postgres (>9.5) functionality and the
Go standard library to provide a convenient, concurrent-safe message queue. 

# Motivation

It is not uncommon to require a robust messaging queue in an application. It is
also not uncommon to reach for powerful tooling such as RabbitMQ or Kafka.
However, in most cases we don't need to use such powerful tools, at least not
to start with and in many cases we are already using a database. If you've made
Good Decisions, you've probably ended up using Postgres to store your data, so
why not use it as your queuing solution too; at least it will be one less tool
to configure and maintain.


# Approach

This concurrent-safe approach relies on Postgres `FOR UPDATE SKIP LOCKED` and
`DELETE` statements. A good explanation of the approach can be found at:
[here](https://www.2ndquadrant.com/en/blog/what-is-select-skip-locked-for-in-postgresql-9-5/)
[and here](https://www.crunchydata.com/blog/message-queuing-using-native-postgresql).

Enqueuing messages is a reasonably straightforward atomic process (`INSERT`).
Dequeuing relies on the approach outlined above but returns a set of messages
and a `db.Tx` which can be used to execute further SQL statements on the result
or `ROLLBACK` the transaction if processing the result fails. If a Dequeued
Message fails to process then it will remain in a processable state for future
workers to pick up.


# Proposed API

## `Connect(ctx context.Context, conn db *sql.DB, queue string) Queue`


## `NewQueue(ctx context.Context, conn *sql.DB, queueName string) Queue`


## `Enqueue(ctx context.Context, message Message) EnqueueResult`


## `Dequeue(ctx context.Context, batchSize Batch) DequeueResult`



# Proposed Types

## `Queue`

```golang
type Queue struct {}
```


## `Message`

```golang
type Message struct {
	Message		string
	Processed 	bool
}
```

## `Batch`

```golang
type BatchSize struct {
	Batch int
}
```

## `EnqueueResult`

```golang
type EnqueueResult struct {
	ID 	string
	Error	error
}
```

## `DequeueResult`

```golang
type DequeueResult struct {
	Transaction	sql.Tx
	Message 	Message
	Errors		[]error
}
```
