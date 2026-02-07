## nft_and_nft_markeplace

### markeplace 上的所有 NFT

每個 Object 上限 256 kb

因次會使用 Dynamic Fields 來存放 NFT 

```move
public struct MarktetPlace has key {
    id: UID,
    whitelist: VecSet<ID>,
    // list: VecSet<Rookie>,    (X)
}

public struct Listing<Item> has key, store {
    id: UID,
    owner: address,
    price: u64,
    item: Item,
}
```

所以要 list 所有 MarktetPlace 的 NFT 會需要

MarktetPlace id + Dynamic Fields

並且使用 `sui client dynamic-field` 來取得所有的 NFT

可以想像成關聯式資料庫的查詢

但是回傳的不可能回給你所有 metadata，因此需要另外再撈

ts-sdk: 

- `getDynamicField()`
- `multiGetObject()` - batch call

### 撈出自己錢包內所有 NFT

ts-sdk: 
- `getOwnObject()` - 回傳 struct type

取回該 NFT type 後就可以在自己的錢包內做 filter

### dependencies function

現階段 json rpc 情況下

1. `getDynamicField()`
2. `multiGetObject()` - batch call

下一個版本 SDK 會改成 grpc 因此可以去將這兩個接口 merge

### 打交易

dapp-kit

`useSignAndExecuteTransaction`

#### transaction block

```move
// transaction logic
const tx = new Transaction();

// tx_1
const nft = tx.moveCall({
    target: `${PUBLISHED_AT}::lesson_one::${isDynamicField ? "delist_rookie_with_df" : "delist_rookie_with_dof"}`,
    arguments: [tx.object(NFT_MARKET_PLACE_ID), tx.object(rookieNFTID)],
});

// tx_2
tx.transferObjects([nft], recipient);

// finish tx
await signAndExecuteTransaction({ transaction: tx });
```

tx_2 如果沒做，前端互動會噴錯 `UnusedValueWithoutDrop` 因為 delist_rookie_with_df 會回傳 Rookie Object

這也是 Move 有趣的地方，因為其他鏈如 EVM 無法回傳東西，因此要做到上面操作則需要封裝成新的 function 後部署才能一次用

### 撈出所有 SUI

以前需要去錢包老出所有 SUI Object 在做 merge

但現在 SDK 有方法可以做

```move
coinWithBalance({
    type: '0x2::sui::Sui',
    useGasStation: true,
})
```

前面由提到，假設你錢包只有一個  SUI Object 是無法做交易的，因為一個要轉錢一個要當手續費

useGasStation - 只有在使用 sui 時才能使用

他會自動先切好可以當手續費的 SUI Object

### 課堂作業

UnusedValueWithoutDrop { result_idx: 0, secondary_idx: 0 }

result_idx 第幾個 tx 出錯
secondary_idx 第幾個回傳


```move
public fun buy(
    market: &mut MarktetPlace,
    id: ID,
    coin: &mut Coin<SUI>,
    ctx: &mut TxContext,
): Rookie {}


// source code
public fun transfer<T: key>(obj: T, recipient: address) {}
public fun public_transfer<T: key + store>(obj: T, recipient: address) {}
```

可以看到這些 <T> 都沒有 & 或 &mute 因此你無法將傳入的 coin 無法通過上面兩個 function 來處理

```javascript
const tx = new Transaction();

// tx_1
const coin = tx.add(
coinWithBalance({
    type: "0x2::sui::SUI",
    balance: payment,
    useGasCoin: true,
}),
);

const nft = tx.moveCall({
target: `${PUBLISHED_AT}::lesson_one::buy`,
arguments: [
    tx.object(NFT_MARKET_PLACE_ID),
    tx.object(rookieNFTID),
    coin,
],
});

// tx_2
tx.transferObjects([nft, coin], recipient); // 因此需要將 coin 轉回

// finish tx
await signAndExecuteTransaction({ transaction: tx });

return;
```



