+++
title = "Senders/Receivers"
date = 2025-05-05
+++

C++26 introduces senders/receivers as a generic framework for organizing (asynchronous) execution.
Let's look into them.

- `sender`: A description of work that produces values which are sent to receivers.
- `receiver`: A consumer of values sent by senders.

In order to produce actual work, a sender must be connected to a receiver, and the result of such a `connect` operation is the task to be executed, which we refer to as `operation_state`.

```cpp
struct my_sender {
  template <typename Receiver>
  auto connect(Receiver&& recv) -> my_operation_state { /* ... */ }
};
```

Receivers receive values via `set_value`:

```cpp
struct my_receiver {
  auto set_value(auto&& values) -> void {}
};
```

Operation states represent the actual work that need to be executed.
So they need to have a `start` method.

```cpp
struct my_operation_state {
  auto start() -> void {}
};
```

Now we can put together these three pieces and start the execution:

```cpp
my_sender{}.connect(my_receiver{}).start()
```

## The `just` sender

Our first sender is the `just` sender.
It simply sends a given value to receivers.

Let's first write the operation state of `just`.
When a `just` sender operates, it should give a value to a receiver.

```cpp
template <typename R, typename V>
struct just_operation {
  R r;
  V v;
  auto start() -> void {
    std::move(r).set_value(std::move(v));
  }
};
```

It's then trivial to write down the `just` sender.

```cpp
template <typename T>
struct just_sender {
  T t;
  template <typename R>
  auto connect(R&& r) -> just_operation<R, T> {
    return {std::forward<R>(r), std::move(t)};
  }
};

template <typename T>
auto just(T&& t) -> just_sender<T> {
  return {std::forward<T>(t)};
}
```

## `then`: a sender adapter
