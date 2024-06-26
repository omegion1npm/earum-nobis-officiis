# @omegion1npm/earum-nobis-officiis&nbsp;&nbsp;&nbsp;[![Build Status](https://github.com/omegion1npm/earum-nobis-officiis/actions/workflows/ci.yml/badge.svg)](https://github.com/omegion1npm/earum-nobis-officiis/actions/workflows/ci.yml) [![Coverage Status](https://coveralls.io/repos/davedoesdev/@omegion1npm/earum-nobis-officiis/badge.png?branch=master)](https://coveralls.io/r/davedoesdev/@omegion1npm/earum-nobis-officiis?branch=master) [![NPM version](https://badge.fury.io/js/@omegion1npm/earum-nobis-officiis.png)](http://badge.fury.io/js/@omegion1npm/earum-nobis-officiis)

Node.js [`Writable`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_class_stream_writable) which goes at the speed of its fastest peer `Writable` and ends peers which can't keep up.

- Alternative to [`readable.pipe`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_readable_pipe_destination_options) which goes at the rate of the slowest peer.
- Peers which aren't drained when a write occurs are ended.
- With no peers added, `@omegion1npm/earum-nobis-officiis` consumes data as fast as it is written and throws it away.
- Full set of unit tests with 100% coverage.

Example:

```javascript
var stream = require('stream'),
    assert = require('assert'),
    FastestWritable = require('@omegion1npm/earum-nobis-officiis').FastestWritable,
    source = new stream.PassThrough(),
    fw = new FastestWritable(),
    dest1 = new stream.PassThrough({ highWaterMark: 1 }),
    dest2 = new stream.PassThrough({ highWaterMark: 1 });

source.pipe(fw);
fw.add_peer(dest1);
fw.add_peer(dest2);

source.write('foo');
assert.equal(dest1.read().toString(), 'foo');
source.write('bar');
// drain emitted next tick
process.nextTick(function ()
{
    assert(dest2._writableState.ended);
});
```

The API is described [here](#tableofcontents).

## Installation

```shell
npm install @omegion1npm/earum-nobis-officiis
```

## Licence

[MIT](LICENCE)

## Test

```shell
grunt test
```

## Code Coverage

```shell
grunt coverage
```

[c8](https://github.com/bcoe/c8) results are available [here](http://rawgit.davedoesdev.com/davedoesdev/@omegion1npm/earum-nobis-officiis/master/coverage/lcov-report/index.html).

Coveralls page is [here](https://coveralls.io/r/davedoesdev/@omegion1npm/earum-nobis-officiis).

## Lint

```shell
grunt lint
```

# API

<a name="tableofcontents"></a>

- <a name="toc_fastestwritableoptions"></a>[FastestWritable](#fastestwritableoptions)
- <a name="toc_fastestwritableprototypeadd_peerpeer"></a><a name="toc_fastestwritableprototype"></a>[FastestWritable.prototype.add_peer](#fastestwritableprototypeadd_peerpeer)
- <a name="toc_fastestwritableprototyperemove_peerpeer-end"></a>[FastestWritable.prototype.remove_peer](#fastestwritableprototyperemove_peerpeer-end)
- <a name="toc_fastestwritableeventsempty"></a><a name="toc_fastestwritableevents"></a>[FastestWritable.events.empty](#fastestwritableeventsempty)
- <a name="toc_fastestwritableeventswaitingstop_waiting"></a>[FastestWritable.events.waiting](#fastestwritableeventswaitingstop_waiting)
- <a name="toc_fastestwritableeventsreadynum_waiting-total-drain"></a>[FastestWritable.events.ready](#fastestwritableeventsreadynum_waiting-total-drain)
- <a name="toc_fastestwritableeventslaggardpeer"></a>[FastestWritable.events.laggard](#fastestwritableeventslaggardpeer)
- <a name="toc_fastestwritableeventspeer_addedpeer"></a>[FastestWritable.events.peer_added](#fastestwritableeventspeer_addedpeer)
- <a name="toc_fastestwritableeventspeer_removedpeer"></a>[FastestWritable.events.peer_removed](#fastestwritableeventspeer_removedpeer)

## FastestWritable([options])

> Creates a new `FastestWritable` object which can write to multiple peer [`stream.Writable`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_class_stream_writable) objects.

Inherits from [`stream.Writable`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_class_stream_writable) so you can use any `Writable` method or event in addition to those described here.

**Parameters:**

- `{Object} [options]` Configuration options. This is passed onto `Writable`'s constructor and can contain the following extra property:
  - `{Boolean} [end_peers_on_finish]` Whether to call [`writable.end`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_writable_end_chunk_encoding_callback) on all peers when this `FastestWritable` object emits a [`finish`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_event_finish) event. Defaults to `true`.

  - `{Boolean} [destroy_peers_on_destroy]` Whether to call [`writable.destroy`](https://nodejs.org/dist/latest-v14.x/docs/api/stream.html#stream_writable_destroy_error) on all peers when this `FastestWritable` is destroyed. Defaults to `true`.

  - `{Boolean} [emit_laggard]` Whether to emit an event named `laggard` on any peers which can't keep up _instead of_ ending them. Defaults to `false`.

<sub>Go: [TOC](#tableofcontents)</sub>

<a name="fastestwritableprototype"></a>

## FastestWritable.prototype.add_peer(peer)

> Add a peer [`Writable`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_class_stream_writable) to the list of peers to which data will be written.

When [`writable.write`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_writable_write_chunk_encoding_callback) is called on this `FastestWritable` object, the data is written to every peer. `FastestWritable` drains when _at least one_ of its peers drains. When `writable.write` is called again, [`writable.end`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_writable_end_chunk_encoding_callback) is called on any peer which hasn't drained from the previous `writable.write` call.

If this `FastestWritable` object has no peer `Writable`s then it drains immediately.

**Parameters:**

- `{stream.Writable} peer` Peer `Writable` to add.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.prototype](#toc_fastestwritableprototype)</sub>

## FastestWritable.prototype.remove_peer(peer, end)

> Remove a peer [`Writable`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_class_stream_writable) from the list of peers to which data will be written.

**Parameters:**

- `{stream.Writable} peer` Peer `Writable` to remove.
- `{Boolean} end` Whether to call [`writable.end`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_writable_end_chunk_encoding_callback) on the peer once it's been removed from the list. Defaults to `true`.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.prototype](#toc_fastestwritableprototype)</sub>

<a name="fastestwritableevents"></a>

## FastestWritable.events.empty()

> `empty` event

A `FastestWritable` object emits an `empty` event when it has no more `Writable` objects in its list of peers. 

Note that when a `FastestWritable` object is empty, it is always drained and throws any data it receives away.

You could, for example, [`end`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_writable_end_chunk_encoding_callback) the `FastestWritable` object when `empty` is emitted.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.events](#toc_fastestwritableevents)</sub>

## FastestWritable.events.waiting(stop_waiting)

> `waiting` event

A `FastestWritable` object emits a `waiting` event when it's waiting for any of its peers to drain.

When a peer drains, the `FastestWritable` object will emit a [`ready`](#fastestwritableeventsreadynum_waiting-total-drain) event. If there are no listeners for the `ready` event then it will emit a [`drain`](http://nodejs.org/docs/v0.11.13/api/stream.html#stream_event_drain) event.

**Parameters:**

- `{Function} stop_waiting` Call this function to force the `FastestWritable` object to drain without waiting for any of its peers to drain. You could use this to implement a timeout, for example. It's safe to call `stop_waiting` more than once or even after a peer has drained of its own accord.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.events](#toc_fastestwritableevents)</sub>

## FastestWritable.events.ready(num_waiting, total, drain)

> `ready` event

A `FastestWritable` object emits a `ready` event when one of its peers drains. It gives you the ability to control when the `FastestWritable` object emits `drain`.

**Parameters:**

- `{Integer} num_waiting` Number of peers which still haven't drained for the latest data written to the `FastestWritable` object.
- `{Integer} total` Number of peers which received the latest data written to the `FastestWritable` object.
- `{Function} drain` Call this function to let the `FastestWritble` object drain without waiting for any more of its peers to drain. It's safe to call `drain` more than once.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.events](#toc_fastestwritableevents)</sub>

## FastestWritable.events.laggard(peer)

> `laggard` event

A `FastestWritable` object emits a `laggard` event when one of its peers can't keep up, if `emit_laggard` was passed to the constructor. Note a `laggard` event is also emitted on the peer.

**Parameters:**

- `{stream.Writable} peer` Peer `Writable` which can't keep up.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.events](#toc_fastestwritableevents)</sub>

## FastestWritable.events.peer_added(peer)

> `peer_added` event

A `FastestWritable` object emits a `peer_added` event when a peer has been added to the list of peers to which data will be written.

**Parameters:**

- `{stream.Writable} peer` Peer added.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.events](#toc_fastestwritableevents)</sub>

## FastestWritable.events.peer_removed(peer)

> `peer_removed` event

A `FastestWritable` object emits a `peer_removed` event when a peer has been removed from the list of peers to which data will be written.

**Parameters:**

- `{stream.Writable} peer` Peer removed.

<sub>Go: [TOC](#tableofcontents) | [FastestWritable.events](#toc_fastestwritableevents)</sub>

_&mdash;generated by [apidox](https://github.com/codeactual/apidox)&mdash;_
