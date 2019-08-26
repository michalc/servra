# servra

Lightweight and low-level asyncio Python HTTP server.

> Work-in-progress. These docs are a rough design spec


## Installation

```bash
pip install servra
```


## Usage

The API is streaming-first: for both request and response bodies, asynchronous iterators are used.

```python
import asyncio
from servra import Server

async def handler(method, path, request_headers, request_body):
    # Read request body
    async for chunk in request_body:
        print(chunk)

    # Yield response code and headers
    yield b'200', ((b'content-length', b'3'),)

    # Yield remaining bytes
    yield b'a'
    yield b'bc'

async def main():
    start, stop = Server(('0.0.0.0', 8080), handler)

    # Start listening on specified address, and in the background on each
    # incoming connection call the handler in a new task
    await start()

    # Stop listening, finish any incoming requests, close all connections
    await stop()

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```


## Scope

The scope of servra is restricted to:

- creating a task for each incoming connection;
- maintaining this task and connection for chains of keep-alive requests;
- passing and receiving HTTP headers and streaming bodies;
- decoding chunked requests.

This is to make the core behaviour useful to a reasonable range of uses, but to _not_ include what can be added by the handler or separate infrastructure. Specifically not included:

- HTTPS: servra would typically be used behind a load balancer or other server that performs TLS termination;
- path-based routing to different handler functions;
- returning a 500 on error: exceptions in the handler will simply close the connection;
- adding any HTTP header, including `content-length` or `transfer-encoding`, although servra will examine incoming and outgoing headers, as well as the HTTP version, and determine if the connection can remain open.
