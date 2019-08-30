# Peers

Manages delegated trading rules for use in the Swap Protocol. [View on GitHub](https://github.com/airswap/airswap-protocols/tree/master/protocols/peer).

The authorization feature enables traders to deploy smart contracts that trade on their behalf. These contracts can include arbitrary logic and connect to other liquidity sources. This contract lets one set and unset specific trading rules.

## Features

- **Trustless Trading** to configure a smart contract to trade on your behalf.
- **Limit Orders** to set rules to only take trades at specific prices.
- **Partial Fills** to send up to a maximum amount of a token.

## Price Calculations

All amounts are in the smallest unit (e.g. wei), so all calculations based on price result in a whole number. For calculations that would result in a decimal, the amount is automatically floored by dropping the decimal. For example, a price of `5.25` and `takerParam` of `2` results in `makerParam` of `10` rather than `10.5`. Tokens have many decimal places so these differences are very small.

### Rule Examples

Set a rule to send up to 100,000 DAI for WETH at 0.0032 WETH/DAI

```Solidity
setRule(<WETHAddress>, <DAIAddress>, 100000, 32, 4)
```

Set a rule to send up to 100,000 DAI for WETH at 312.50 WETH/DAI

```Solidity
setRule(<WETHAddress>, <DAIAddress>, 100000, 31250, 2)
```

Set a rule to send up to 100,000 DAI for WETH at 312 WETH/DAI

```Solidity
setRule(<WETHAddress>, <DAIAddress>, 100000, 312, 0)
```

## Constructor

Create a new `Peer` contract.

```Solidity
constructor(
  address _swapContract,
  address _peerContractOwner
) public
```

| Param                | Type      | Description                                           |
| :------------------- | :-------- | :---------------------------------------------------- |
| `_swapContract`      | `address` | Address of the swap contract used to settle trades.   |
| `_peerContractOwner` | `address` | Address of the owner of the peer for rule management. |

## Set a Rule

Set a trading rule on the peer.

```Solidity
function setRule(
  address _takerToken,
  address _makerToken,
  uint256 _maxTakerAmount,
  uint256 _priceCoef,
  uint256 _priceExp
) external onlyOwner
```

| Param             | Type      | Description                                                    |
| :---------------- | :-------- | :------------------------------------------------------------- |
| `_takerToken`     | `address` | The token the peer would send.                                 |
| `_makerToken`     | `address` | The token the consumer would send.                             |
| `_maxTakerAmount` | `uint256` | The maximum amount of token the peer would send.               |
| `_priceCoef`      | `uint256` | The coefficient of the price to indicate the whole number.     |
| `_priceExp`       | `uint256` | The exponent of the price to indicate location of the decimal. |

## Unset a Rule

Unset a trading rule for the peer.

```Solidity
function unsetRule(
  address _takerToken,
  address _makerToken
) external onlyOwner
```

| Param         | Type      | Description                        |
| :------------ | :-------- | :--------------------------------- |
| `_takerToken` | `address` | The token the peer would send.     |
| `_makerToken` | `address` | The token the consumer would send. |

## Get a Maker-Side Quote

Get a quote for the maker (consumer) side. Often used to get a buy price for \_quoteTakerToken.

```Solidity
function getMakerSideQuote(
  uint256 _quoteTakerParam,
  address _quoteTakerToken,
  address _quoteMakerToken
) external view returns (
  uint256 quoteMakerParam
)
```

| Param              | Type      | Description                                             |
| :----------------- | :-------- | :------------------------------------------------------ |
| `_quoteTakerParam` | `uint256` | The amount of ERC-20 token the peer would send.         |
| `_quoteTakerToken` | `address` | The address of an ERC-20 token the peer would send.     |
| `_quoteMakerToken` | `address` | The address of an ERC-20 token the consumer would send. |

| Revert Reason         | Scenario                                         |
| :-------------------- | :----------------------------------------------- |
| `TOKEN_PAIR_INACTIVE` | There is no rule set for this token pair.        |
| `AMOUNT_EXCEEDS_MAX`  | The quote would exceed the maximum for the rule. |

## Get a Taker-Side Quote

Get a quote for the taker (peer) side. Often used to get a sell price for \_quoteMakerToken.

```Solidity
function getTakerSideQuote(
  uint256 _quoteMakerParam,
  address _quoteMakerToken,
  address _quoteTakerToken
) external view returns (
  uint256 quoteTakerParam
)
```

| Param              | Type      | Description                                             |
| :----------------- | :-------- | :------------------------------------------------------ |
| `_quoteMakerParam` | `uint256` | The amount of ERC-20 token the consumer would send.     |
| `_quoteMakerToken` | `address` | The address of an ERC-20 token the consumer would send. |
| `_quoteTakerToken` | `address` | The address of an ERC-20 token the peer would send.     |

| Revert Reason         | Scenario                                         |
| :-------------------- | :----------------------------------------------- |
| `TOKEN_PAIR_INACTIVE` | There is no rule set for this token pair.        |
| `AMOUNT_EXCEEDS_MAX`  | The quote would exceed the maximum for the rule. |

## Get a Max Quote

Get the maximum quote from the peer.

```Solidity
function getMaxQuote(
  address _quoteTakerToken,
  address _quoteMakerToken
) external view returns (
  uint256 quoteTakerParam,
  uint256 quoteMakerParam
)
```

| Param              | Type      | Description                                             |
| :----------------- | :-------- | :------------------------------------------------------ |
| `_quoteTakerToken` | `address` | The address of an ERC-20 token the peer would send.     |
| `_quoteMakerToken` | `address` | The address of an ERC-20 token the consumer would send. |

| Revert Reason         | Scenario                                  |
| :-------------------- | :---------------------------------------- |
| `TOKEN_PAIR_INACTIVE` | There is no rule set for this token pair. |

## Provide an Order

Provide an order to the peer for taking.

```Solidity
function provideOrder(
  Types.Order memory _order
) public
```

| Param   | Type    | Description                                                |
| :------ | :------ | :--------------------------------------------------------- |
| `order` | `Order` | Order struct as specified in the `@airswap/types` package. |

| Revert Reason         | Scenario                                                       |
| :-------------------- | :------------------------------------------------------------- |
| `TOKEN_PAIR_INACTIVE` | There is no rule set for this token pair.                      |
| `AMOUNT_EXCEEDS_MAX`  | The amount of the trade would exceed the maximum for the rule. |
| `PRICE_INCORRECT`     | The order is priced incorrectly for the rule.                  |

# PeerFactory

Deploys Peer contracts for use in the Swap Protocol. [View on GitHub](https://github.com/airswap/airswap-protocols/tree/master/protocols/peer-factory).

## Create a Peer

Create a new Peer contract. Implements `IPeerFactory.createPeer`.

```Solidity
function createPeer(
  address _swapContract,
  address _peerContractOwner
) external returns (address peerContractAddress)
```

| Param                | Type      | Description                          |
| :------------------- | :-------- | :----------------------------------- |
| `_swapContract`      | `address` | The swap contract the peer will use. |
| `_peerContractOwner` | `address` | The owner of the new peer contract.  |

## Has

Check to see whether the factory has deployed a peer by locator. Implements `ILocatorWhitelist.has`.

```Solidity
function has(
  bytes32 _locator
) external view returns (bool)
```

| Param      | Type      | Description                                                        |
| :--------- | :-------- | :----------------------------------------------------------------- |
| `_locator` | `bytes32` | The locator in question, a contract address in the first 20 bytes. |