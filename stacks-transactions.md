The `@stacks/transactions` package provides comprehensive functionality for creating, signing, and broadcasting transactions on the Stacks blockchain, including STX transfers, contract deployments, and contract calls.

## Installation

```package-install
@stacks/transactions
```

## Transaction functions

### makeSTXTokenTransfer [#makeSTXTokenTransfer]

`makeSTXTokenTransfer` creates a signed STX token transfer transaction.

### Signature

```typescript
function makeSTXTokenTransfer(options: SignedTokenTransferOptions): Promise<StacksTransaction>
```

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `recipient` | `string` | Yes | Recipient STX address |
| `amount` | `bigint` | Yes | Amount to transfer in microSTX |
| `senderKey` | `string` | Yes | Sender's private key |
| `network` | `StacksNetwork \| string` | Yes | Network configuration |
| `memo` | `string` | No | Optional memo field |
| `fee` | `bigint` | No | Transaction fee in microSTX |
| `nonce` | `bigint` | No | Account nonce |
| `anchorMode` | `AnchorMode` | No | Block anchoring mode |

### Examples

#### Basic transfer

```typescript

const transaction = await makeSTXTokenTransfer({
  recipient: 'SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159',
  amount: 1000000n, // 1 STX in microSTX
  senderKey: 'b244296d5907de9864c0b0d51f98a13c52890be0404e83f273144cd5b9960eed01',
  network: 'mainnet'
});

const response = await broadcastTransaction({ transaction, network: 'mainnet' });
console.log(response.txid);
```

#### Transfer with memo

```typescript
const transaction = await makeSTXTokenTransfer({
  recipient: 'SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159',
  amount: 2500000n,
  memo: 'Payment for services',
  senderKey: privateKey,
  network: 'testnet'
});
```

### makeUnsignedSTXTokenTransfer [#makeUnsignedSTXTokenTransfer]

`makeUnsignedSTXTokenTransfer` creates an unsigned STX token transfer transaction.

### Signature

```typescript
function makeUnsignedSTXTokenTransfer(options: UnsignedTokenTransferOptions): Promise<StacksTransaction>
```

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `recipient` | `string` | Yes | Recipient STX address |
| `amount` | `bigint` | Yes | Amount to transfer in microSTX |
| `publicKey` | `string` | Yes | Sender's public key |
| `network` | `StacksNetwork \| string` | Yes | Network configuration |
| `memo` | `string` | No | Optional memo field |
| `fee` | `bigint` | No | Transaction fee in microSTX |
| `nonce` | `bigint` | No | Account nonce |

### Example

```typescript

const unsignedTx = await makeUnsignedSTXTokenTransfer({
  recipient: 'SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159',
  amount: 1000000n,
  publicKey: publicKeyString,
  network: 'mainnet'
});
```

### makeContractDeploy [#makeContractDeploy]

`makeContractDeploy` creates a signed smart contract deployment transaction.

### Signature

```typescript
function makeContractDeploy(options: SignedContractDeployOptions): Promise<StacksTransaction>
```

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `contractName` | `string` | Yes | Name for the contract |
| `codeBody` | `string` | Yes | Clarity contract source code |
| `senderKey` | `string` | Yes | Deployer's private key |
| `network` | `StacksNetwork \| string` | Yes | Network configuration |
| `clarityVersion` | `ClarityVersion` | No | Clarity version (defaults to latest) |
| `fee` | `bigint` | No | Transaction fee in microSTX |
| `nonce` | `bigint` | No | Account nonce |

### Examples

#### Deploy a contract

```typescript

const codeBody = readFileSync('./contract.clar', 'utf-8');

const transaction = await makeContractDeploy({
  contractName: 'my-contract',
  codeBody,
  senderKey: privateKey,
  network: 'testnet'
});

const response = await broadcastTransaction({ transaction, network: 'testnet' });
```

#### Deploy with Clarity version

```typescript
const transaction = await makeContractDeploy({
  contractName: 'clarity-v3-contract',
  codeBody: contractCode,
  clarityVersion: 3,
  senderKey: privateKey,
  network: 'mainnet'
});
```

### makeContractCall [#makeContractCall]

`makeContractCall` creates a signed contract function call transaction.

### Signature

```typescript
function makeContractCall(options: SignedContractCallOptions): Promise<StacksTransaction>
```

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `contractAddress` | `string` | Yes | Contract address |
| `contractName` | `string` | Yes | Contract name |
| `functionName` | `string` | Yes | Function to call |
| `functionArgs` | `ClarityValue[]` | Yes | Function arguments |
| `senderKey` | `string` | Yes | Caller's private key |
| `network` | `StacksNetwork \| string` | Yes | Network configuration |
| `postConditions` | `PostCondition[]` | No | Post conditions |
| `validateWithAbi` | `boolean \| ClarityAbi` | No | Validate arguments against ABI |
| `fee` | `bigint` | No | Transaction fee in microSTX |
| `nonce` | `bigint` | No | Account nonce |

### Examples

#### Basic contract call

```typescript

const transaction = await makeContractCall({
  contractAddress: 'SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR',
  contractName: 'hello-world',
  functionName: 'say-hi',
  functionArgs: [Cl.stringUtf8('Hello!')],
  senderKey: privateKey,
  network: 'testnet'
});

const response = await broadcastTransaction({ transaction, network: 'testnet' });
```

#### Call with post conditions

```typescript

const postCondition = Pc.principal('SP2ZD731ANQZT6J4K3F5N8A40ZXWXC1XFXHVVQFKE')
  .willSendLte(1000000n)
  .ustx();

const transaction = await makeContractCall({
  contractAddress: 'SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR',
  contractName: 'token-contract',
  functionName: 'transfer',
  functionArgs: [
    Cl.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159'),
    Cl.uint(100n)
  ],
  postConditions: [postCondition],
  validateWithAbi: true,
  senderKey: privateKey,
  network: 'mainnet'
});
```

### sponsorTransaction [#sponsorTransaction]

`sponsorTransaction` adds a sponsor signature to a transaction, enabling sponsored transactions.

### Signature

```typescript
function sponsorTransaction(options: SponsorTransactionOptions): Promise<StacksTransaction>
```

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `transaction` | `StacksTransaction` | Yes | Transaction to sponsor |
| `sponsorPrivateKey` | `string` | Yes | Sponsor's private key |
| `fee` | `bigint` | Yes | Fee to be paid by sponsor |
| `sponsorNonce` | `bigint` | No | Sponsor account nonce |

### Example

```typescript

// Deserialize the transaction from the origin
const deserializedTx = deserializeTransaction(serializedTx);

// Sponsor the transaction
const sponsoredTx = await sponsorTransaction({
  transaction: deserializedTx,
  sponsorPrivateKey: sponsorKey,
  fee: 1000n,
  sponsorNonce: 0
});

const response = await broadcastTransaction({ transaction: sponsoredTx, network: 'testnet' });
```

### fetchCallReadOnlyFunction [#fetchCallReadOnlyFunction]

`fetchCallReadOnlyFunction` calls a read-only contract function without creating a transaction.

### Signature

```typescript
function fetchCallReadOnlyFunction(options: CallReadOnlyFunctionOptions): Promise<ClarityValue>
```

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `contractAddress` | `string` | Yes | Contract address |
| `contractName` | `string` | Yes | Contract name |
| `functionName` | `string` | Yes | Function to call |
| `functionArgs` | `ClarityValue[]` | Yes | Function arguments |
| `network` | `StacksNetwork` | Yes | Network configuration |
| `senderAddress` | `string` | Yes | Address of the caller |

### Example

```typescript

const result = await fetchCallReadOnlyFunction({
  contractAddress: 'SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR',
  contractName: 'data-store',
  functionName: 'get-value',
  functionArgs: [Cl.stringUtf8('key')],
  network: STACKS_MAINNET,
  senderAddress: 'SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159'
});

console.log(result);
```

### broadcastTransaction [#broadcastTransaction]

`broadcastTransaction` broadcasts a signed transaction to the network.

### Signature

```typescript
function broadcastTransaction(options: BroadcastTransactionOptions): Promise<TxBroadcastResult>
```

### Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `transaction` | `StacksTransaction` | Yes | Signed transaction to broadcast |
| `network` | `StacksNetwork \| string` | Yes | Network to broadcast to |

### Example

```typescript

const result = await broadcastTransaction({
  transaction: signedTx,
  network: 'mainnet'
});

if (result.error) {
  console.error('Broadcast failed:', result.reason);
} else {
  console.log('Transaction ID:', result.txid);
}
```

## Clarity value construction

### Primitive values

```typescript

// Boolean values
const isTrue = Cl.bool(true);
const isFalse = Cl.bool(false);

// Integer values (signed 128-bit)
const positiveInt = Cl.int(42n);
const negativeInt = Cl.int(-100n);

// Unsigned integer values (unsigned 128-bit)
const unsignedInt = Cl.uint(100n);

// Buffer values
const buffer = Cl.bufferFromUtf8('hello world');
const hexBuffer = Cl.bufferFromHex('0x1234');

// String values
const asciiStr = Cl.stringAscii('Hello ASCII');
const utf8Str = Cl.stringUtf8('Hello UTF-8! 👋');
```

### Complex values

```typescript
// Principal values
const standardPrincipal = Cl.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159');
const contractPrincipal = Cl.contractPrincipal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159', 'my-contract');

// Optional values
const none = Cl.none();
const some = Cl.some(Cl.uint(42n));

// Response values
const okResponse = Cl.ok(Cl.stringUtf8('Success'));
const errResponse = Cl.error(Cl.uint(404n));

// Tuple values
const tuple = Cl.tuple({
  name: Cl.stringUtf8('Alice'),
  age: Cl.uint(30n),
  active: Cl.bool(true)
});

// List values
const list = Cl.list([Cl.uint(1n), Cl.uint(2n), Cl.uint(3n)]);
```

## Post conditions

### STX post conditions

```typescript

// Standard principal STX post condition
const stxPostCondition = Pc.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159')
  .willSendGte(1000000n)
  .ustx();

// Contract principal STX post condition
const contractStxCondition = Pc.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159.my-contract')
  .willSendEq(500000n)
  .ustx();
```

### Fungible token post conditions

```typescript
// Standard principal fungible token post condition
const ftPostCondition = Pc.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159')
  .willSendLte(100n)
  .ft('SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR.token-contract', 'my-token');

// Contract principal fungible token post condition
const contractFtCondition = Pc.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159.vault')
  .willNotSend()
  .ft('SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR.token-contract', 'my-token');
```

### Non-fungible token post conditions

```typescript
// Standard principal NFT post condition
const nftPostCondition = Pc.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159')
  .willSend()
  .nft('SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR.nft-contract', 'my-nft', Cl.uint(1n));

// Contract principal NFT post condition
const contractNftCondition = Pc.principal('SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159.marketplace')
  .willNotSend()
  .nft('SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR.nft-contract', 'my-nft', Cl.uint(1n));
```

## Multi-signature transactions

Multi-signature transactions require multiple signatures before broadcasting.

```typescript
  makeUnsignedSTXTokenTransfer,
  createStacksPrivateKey,
  deserializeTransaction,
  pubKeyfromPrivKey,
  publicKeyToString,
  TransactionSigner,
  BytesReader
} from '@stacks/transactions';

// Create unsigned multi-sig transaction
const transaction = await makeUnsignedSTXTokenTransfer({
  recipient: 'SP3FGQ8Z7JY9BWYZ5WM53E0M9NK7WHJF0691NZ159',
  amount: 1000000n,
  fee: 200n,
  numSignatures: 2, // Require 2 of 3 signatures
  publicKeys: [publicKey1, publicKey2, publicKey3],
  network: 'mainnet'
});

// Sign with multiple parties
const deserializedTx = deserializeTransaction(transaction.serialize());
const signer = new TransactionSigner(deserializedTx);

// Add required signatures
signer.signOrigin(privateKey1);
signer.signOrigin(privateKey2);

// Append public key of non-signing party
signer.appendOrigin(publicKey3);

// Broadcast the multi-sig transaction
const signedTx = deserializedTx;
const response = await broadcastTransaction({ transaction: signedTx, network: 'mainnet' });
```

## Key generation utilities

### randomPrivateKey [#randomPrivateKey]

`randomPrivateKey` generates a new random private key.

```typescript

const privateKey = randomPrivateKey();
console.log(privateKey); // Random 32-byte hex string with optional compression flag
```

### privateKeyToPublicKey [#privateKeyToPublicKey]

`privateKeyToPublicKey` derives a public key from a private key.

```typescript

const privateKey = 'b244296d5907de9864c0b0d51f98a13c52890be0404e83f273144cd5b9960eed01';
const publicKey = privateKeyToPublicKey(privateKey);
console.log(publicKey); // Compressed public key
```

## Utility functions

### deserializeTransaction [#deserializeTransaction]

`deserializeTransaction` converts a serialized transaction back to a transaction object.

```typescript

const serializedTx = '0x00000000...'; // Hex string
const transaction = deserializeTransaction(serializedTx);
```

### cvToJSON [#cvToJSON]

`cvToJSON` converts Clarity values to JSON format for easier manipulation.

```typescript

const clarityValue = hexToCV('0x0100000000000000000000000000000001');
const json = cvToJSON(clarityValue);
console.log(json); // { type: 'uint', value: '1' }
```