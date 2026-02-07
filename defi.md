# Defi

## Account-Based vs. Object-Centric

只碰到 Own boject 情況下不需要過共識

所以如果是用 Account-Based 的思考

```move
#[allow(unused_field)]
public struct GlobalBalances {
    balances: Table<address, u64>,
}
```

這樣他會是 share object

好處：
- 可以取得所有金流以及他總有多少錢（indexer）

壞處：
- 無法平行化

因此在設計時，能用 Own boject 盡量用 Own boject（不過共識）
訂定好詳細的 ownership rule

## Generic Types

```move
// 底層封裝的 u64
public struct Balance<phantom T> has store {
    // 只有 store，因為可以塞到其他 struct 內
    value: u64,
}

public struct Coin<phantom T> has key, store {
    // 多一個 key，因為要成為一個 Object
    id: UID,
    balance: Balance<T>,
}

// 所以 Coin 的角色是用來讓 u64 的金額變成一個 Object
```

default 都是 private function

當下的 Object 如果是 Balance，可以呼叫任一 Balance function

但一但包進 Coin 後，是無法再去呼叫 Balance 的 function 的，因此通常會再做一層 helper function

```move
/// Get immutable reference to the balance of a coin.
public fun balance<T>(coin: &Coin<T>): &Balance<T> {
    &coin.balance
}

/// Get a mutable reference to the balance of a coin.
public fun balance_mut<T>(coin: &mut Coin<T>): &mut Balance<T> {
    &mut coin.balance
}
```

## transfer

其侄只是改變 owner address 而已，因為是 UTXO

```move
public struct OwnedCoin has key {
    id: UID,
    owner: Owner,
    balance: SimpleBalance,
}


public fun transfer<T: key>(obj: T, recipient: address) {}
public fun public_transfer<T: key + store>(obj: T, recipient: address) {}
```

cability 也可以用來卡控，如上加上 key 限定只能使用 transfer

## witness

不會存在在鏈上帳本，只有在 runtime 時才會產生

作用通常用於 witelist 用，例如限制某些 func 一定要帶 witness 才能使用

otw - 又更限制，因為只能產生一次
舉例： create_currency 時需夾帶，而 create_currency 內又需要檢查 otw，因此可以嚴格限制一次性的 fun

## 設計哲學

能不過共識就不過共識

