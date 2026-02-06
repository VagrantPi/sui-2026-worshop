# move 進階

sui move 是以 Object 開發的

所以重點是 Object 的 ownership

所以今天我如果要做一個 set_math(student, math)

此時只有兩種方法可以呼叫

- student 為 share object
- 呼叫者要有 student

## cap

因此如果要控制某些 func 只能由項目方控制

```move
public struct AdminCap has key, store{
    id: UID,
}

public set_hp(
    _: &AdminCap, // 引入不使用所以用 _ 去接，另外也不需要把實體傳入，所以使用 &
    role: &Role
    hp: u64,
) {}
```

## 泛型


概念有兩個作用：
- 當作標籤識別
- 跟欄位有關

ex1 struct:

- Color 識別不同 struct
- Content 為帶入的不同欄位

```move
public struct Box<phantom Color, phantom Content> has key, store{
    id: UID,
    content: Content, 
}
```

!! 當外層有 has 時，帶入的參數都會有一樣的能力

如果不需要的話則在參數前面加上 `phantom`

此外如果那個參數可以有而外的能力 `<Content: store>`

但這用法會出現一些問題，舉例 `<Content: key, store>` 這樣會無法帶入 u8 這些基本型別，因為 u8 本身沒有 key

另外的是如果 `<Content: store>` 這樣傳入的參數可以傳入 u8，原因在於 `: store` 為最低標準


ex2 fun:

```move
public fun new<Color, Content>(
    content: Content,
    ctx: &mut TxContext,
){
    let box = Box<Color, Content>{
        id: object::new(ctx),
        content: content
    };

    transfer::transfer(box, ctx.sender());
}
```

cli 使用時

```
--args --type-args package::module::struct u8 --args 8
```

> content 不用帶，因此後面參數只需要帶入 8


彈性 -> 風險

因此很多項目方在定義 struct 時會用泛型

但在 function 定義時會特別設計限制

## Dynamic Fields

Dynamic Fields
has copy, store, drop

要綁定的實體
has store
> 不過這是最低要求因此也可以宣告 has store, copy

來看 官方 source code
```move
public fun add<Name: copy + drop + store, Value: store>(
```

可以看到 Value 最低要求為 store


### Dynamic Object Fields

要綁定的實體
has store, key

### 差異

Dynamic Object Fields 因為有 id，因此要綁定的東西一樣可以在 explore 上找到該 Object

Dynamic Fields 要綁的東西，即便有 key，因此在 explore 上找不到
> 不過目前在 suivision 可以查得到

但還是可以從 suivision 可以看到

- df 的 type objectId::XXXX
    - 可以直接定位到綁定的 Object
- dof 的 type sui::dynamic_field::...::XXXX
    - 需要從 dynamic_field 回找才找得到


### 常用 function

```move
module sui::dynamic_field;

// 綁標籤
public fun add<Name: copy + drop + store, Value: store>(
    object: &mut UID,
    name: Name, // 標籤
    value: Value, // 綁定實體
)

// 拆掉標籤
public fun remove<Name: copy + drop + store, Value: store>(object: &mut UID, name: Name): Value
// 只會設下 value 綁定實體，這時出現一些問題，假設 value 不具備 drop 能力就需要特別處理


// 檢查有沒有這個標籤
public fun exists_<Name: copy + drop + store>(object: &UID, name: Name): bool
```

### 使用的約定熟成

```move
// 太長了，會縮寫
use sui::{
    dynamic_field:: { Self as df },
    dynamic_object_field:: { Self as dof },
};
```

```shell
# 建立 student
sui client call --package 0xc4a4ff61f902edad7f6157b5cf3deb35f2b03bd746d5ab595735f80423217cae --module df_and_dof --function new --args kais

# 帶帽子
sui client call --package 0xc4a4ff61f902edad7f6157b5cf3deb35f2b03bd746d5ab595735f80423217cae --module df_and_dof --function wear_hat --args 0x1e79ab7cd5ddc09b237351c11794917c6237d0908c53a072efa4766185fded87 SuiHat

# 穿衣服
sui client call --package 0xc4a4ff61f902edad7f6157b5cf3deb35f2b03bd746d5ab595735f80423217cae --module df_and_dof --function wear_clothes --args 0x1e79ab7cd5ddc09b237351c11794917c6237d0908c53a072efa4766185fded87 SuiClothes
```

## Witness 見證 

是一個 struct has drop

所以想像它見證了一個 fun 的使用

大多使用場景：限制合約行為，以 coin 來說，可以限制這個幣只能被發一次 init 跑完後就被 drop 了

```move
module module_3::witness;

// witness
// 最低標一定要有 drop
public struct Aaa has drop{}
public struct Bbb has drop, store{}

// one-time witness 只能有 drop
// 且命名與 module 一樣且全大寫
// 只能在 init function 被長出來
public struct WITNESS has drop{}
```

```move
// 普通 witness 可以無線產生
public fun new_aaa() {}


// init
fun init(ctx: &mut TxContext) {}
fun init(otw: WITNESS, ctx: &mut TxContext) {}
// 而如 init 離開後，WITNESS 實體會消失，如果想讓他的 type 還跑留在鏈上可以使用泛型
// 所以可以看到 sui coin
// package::module::struct<WITNESS>
// 0x2::coin_registry::Currency<0x2::sui::SUI>
```

```move
// packageId::witness::Asset<packageId::witness::WITNESS>
// 0x2::coin::Coin<0x2::sui::SUI>
public struct Asset<phantom T: drop> has store, key{
    id: UID,
    amount: u64,
}
```

## 發幣

官方文件

https://github.com/MystenLabs/sui/blob/main/crates/sui-framework/packages/sui-framework/sources/registries/coin_registry.move

```move
public fun new_currency_with_otw<T: drop>(
    otw: T,
    decimals: u8,
    symbol: String,
    name: String,
    description: String,
    icon_url: String,
    ctx: &mut TxContext,
): (CurrencyInitializer<T>, TreasuryCap<T>) {
    assert!(sui::types::is_one_time_witness(&otw), ENotOneTimeWitness);
    assert!(is_ascii_printable!(&symbol), EInvalidSymbol);

    let treasury_cap = coin::new_treasury_cap(ctx);
    let currency = Currency<T> {
        id: object::new(ctx),
        decimals,
        name,
        symbol,
        description,
        icon_url,
        supply: option::some(SupplyState::Unknown),
        regulated: RegulatedState::Unregulated,
        treasury_cap_id: option::some(object::id(&treasury_cap)),
        metadata_cap_id: MetadataCapState::Unclaimed,
        extra_fields: vec_map::empty(),
    };

    (CurrencyInitializer { currency, is_otw: true, extra_fields: bag::new(ctx) }, treasury_cap)
}
```

所以基本上就是直接呼叫

```move
fun init(
    otw: DEMO_COIN,
    ctx: &mut TxContext,
){
    let (initializer, treasury_cap) = coin_registry::new_currency_with_otw(otw, 9, b"DC".to_string(), b"Demo Coin".to_string(), b"Demo Coin".to_string(), b"https://demo.coin.xyz".to_string(), ctx);
    let metadata_cap = initializer.finalize(ctx);

    transfer::public_transfer(metadata_cap, ctx.sender());
    transfer::public_transfer(treasury_cap, ctx.sender());
}
```



呼叫完 new_currency_with_otw 會得到兩個東西
- initializer: 回傳的 metadata 包含了 coin 的所有參數集合的 Object
- treasury_cap: 有 treasury_cap 才可以把 coin mint, burn 掉

下一步我們要看怎麼讓他們消失，因為離開 init 後所有變數都需要移除

所以我要想辦法移除的 function 則需要去同一 module 尋找

```move
// CurrencyInitializer 不具備 drop
public struct CurrencyInitializer<phantom T> {
    currency: Currency<T>,
    extra_fields: Bag,
    is_otw: bool,
}

// 但又回傳了 MetadataCap
public fun finalize<T>(builder: CurrencyInitializer<T>, ctx: &mut TxContext): MetadataCap<T> {}


// 有 key，因此可以 transfer
public struct MetadataCap<phantom T> has key, store { id: UID }


// let metadata_cap = coin_registry::finalize(initializer, ctx);
// 當同一包 module 下，可以轉換成下面
// 第一個參數放到前面 `.` function(第二個開始的參數)
let metadata_cap = initializer.finalize(ctx);

// 舉例
// tx_context::sender(ctx);
// -> ctx.sender();
```


```move
use sui::coin::{Self, TreasuryCap, DenyCapV2, CoinMetadata, RegulatedCoinMetadata, Coin};
```

往上找

```move
// 有 key，因此可以 transfer
public struct TreasuryCap<phantom T> has key, store {
    id: UID,
    total_supply: Supply<T>,
}
```

### mint

官方文件，基本跟前面單元講解到 OTW 的用法一樣，參照 `05_witness.move`

```move
public fun mint<T>(cap: &mut TreasuryCap<T>, value: u64, ctx: &mut TxContext): Coin<T> {
    Coin {
        id: object::new(ctx),
        balance: cap.total_supply.increase_supply(value),
    }
}
```

### burn

```move
// coin::burn(cap, demo_coin)
cap.burn(demo_coin);
```

### register

回家研究

new_currency_with_otw


currency 建立時會註冊進去官方的 0xc Share Object 做統一管理

```move
public fun register(
    registry: &mut CoinRegistry,
    currency: Receiving<Currency<DEMO_COIN>>,
    ctx: &mut TxContext,
){
    registry.finalize_registration<DEMO_COIN>(currency, ctx);
}
```

## NFT

有這兩個就可以發有圖片的 nft

```move
use sui::{
    package::{ Self },
    display::{ Self },
};
```

翻文件

```move
public fun new_with_fields<T: key>(
    pub: &Publisher,
    fields: vector<String>,
    values: vector<String>,
    ctx: &mut TxContext,
): Display<T> {
    let len = fields.length();
    assert!(len == values.length(), EVecLengthMismatch);

    let mut display = new<T>(pub, ctx);
    fields.zip_do!(values, |field, value| display.add_internal(field, value));
    display
}
```

第一個參數 Publisher

sui::package

```move
public fun claim<OTW: drop>(otw: OTW, ctx: &mut TxContext): Publisher {
    assert!(types::is_one_time_witness(&otw), ENotOneTimeWitness);

    let type_name = type_name::with_original_ids<OTW>();

    Publisher {
        id: object::new(ctx),
        package: type_name.address_string(),
        module_name: type_name.module_string(),
    }
}
```

OTW - init
ctx - TxContext

OK，到這邊就可以知道為什麼需要 sui::display 的同時也需要 sui::package

```move
use sui::{
    package::{ Self },
    display::{ Self },
};
```





