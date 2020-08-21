# Distributed Map-Reduce with Python

_created by Austin Poor_

* `Client` connects to `master` node
* `Master` node passes instructions from `client` to `worker` nodes
* `Worker` nodes execute instructions on their data
* `Client` sends async instructions to `master`
* `Master` acts as a _server_, waiting for instructions from `client`, then acts as a _client_ passing instructions along to the `client` nodes (which only act as servers).
* When the cluster is starting...
  * `master` spins up at a specified address
  * Then, as `worker` nodes spin up, they pass `master` their address (as _clients_) and then move into _server-mode_

---

## Python AsyncIO Streams Reference

**TCP Echo Client**

```python
import asyncio

async def tcp_echo_client(message):
    reader, writer = await asyncio.open_connection(
        '127.0.0.1', 8888)

    print(f'Send: {message!r}')
    writer.write(message.encode())

    data = await reader.read(100)
    print(f'Received: {data.decode()!r}')

    print('Close the connection')
    writer.close()

asyncio.run(tcp_echo_client('Hello World!'))
```

**TCP Echo Server**

```python
import asyncio

async def handle_echo(reader, writer):
    data = await reader.read(100)
    message = data.decode()
    addr = writer.get_extra_info('peername')

    print(f"Received {message!r} from {addr!r}")

    print(f"Send: {message!r}")
    writer.write(data)
    await writer.drain()

    print("Close the connection")
    writer.close()

async def main():
    server = await asyncio.start_server(
        handle_echo, '127.0.0.1', 8888)

    addr = server.sockets[0].getsockname()
    print(f'Serving on {addr}')

    async with server:
        await server.serve_forever()

asyncio.run(main())

```


