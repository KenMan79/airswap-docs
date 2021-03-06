Servers are processes that implement the [Quote](../system/apis.md#quotes) and [Order](../system/apis.md#orders) APIs using [JSON-RPC 2.0](http://www.jsonrpc.org/specification) over HTTPS.

# Server Client

Add the `@airswap/protocols` package to your application.

```bash
$ yarn add @airswap/protocols
```

Import the Server client.

```javascript
import { Server } from '@airswap/protocols'
```

### `constructor`

Create a new `Server` client.

```javascript
public constructor(locator: string)
```

| Param     | Type     | Optionality | Description                                                  |
| :-------- | :------- | :---------- | :----------------------------------------------------------- |
| `locator` | `string` | `required`  | URL of the server, If no scheme provided, `https` is implied |

**Example**
Create a client to connect to `https://maker.example.com/`.

```javascript
const server = new Server('maker.example.com')
```

**Example**
Create a client to connect to a local development server.

```javascript
const server = new Server('http://localhost:3000')
```

### `Quotes`

Servers implement the [`Quote`](../apis/quote.md) API.

**Example**
Call `getMaxQuote` on a local development Server.

```javascript
const server = new Server('http://localhost:3000')
const quote = await server.getMaxQuote(senderToken, signerToken)
```

### `Orders`

Servers implement the [`Order`](../apis/order.md) API.

**Example**
Call `getSenderSideOrder` on a local development Server.

```javascript
const wallet = new ethers.Wallet('...')
const server = new Server('http://localhost:3000')
const quote = await server.getSenderSideOrder(
  signerAmount,
  signerToken,
  senderToken,
  wallet.address,
)
```
