## flash_loan

一個 func 內不見一有 transfer，防止 flash_loan

PTB -> 可以同時塞入很多個呼叫

sui 的設計模式，function 盡量簡單，然後包成一個 PTB 一起送

PTB -> 一個串一個，只要在最後所有東西都清空就好

回到前面範例

```move
public fun new(
    name: String,
    ctx: &mut TxContex,
) {
    let student = Student{
        id: object::new(ctx)
        name
    }

    transfer::publiic_transfer(student, tx_context::sender(ctx))
}
```

用 PTB 的方式去打包靈活性會更好

```move
public fun new(
    name: String,
    ctx: &mut TxContex,
): Student {
    let student = Student{
        id: object::new(ctx)
        name
    }

    student
}

public fun transfer_student(
    student: Student,
    ctx: &mut TxContex,
) {
    transfer::publiic_transfer(student, tx_context::sender(ctx))
}
```

可以想想如果 new 完就 transfer，就轉移 object 了，我的下一個交易就無法做操作

用 PTB 的方式，就可以做完一連傳操作後在最後再 transfer


### Hot Potato

回到 flash_loan 當你要去 Lending 借錢時，除了你拿到錢時，你還會拿到一個 receipt

```move
public struct receipt {}
```

不給任何能力，因此對方在做完 flash_loan 時需要去清空 receipt


### PTB 指令

```move
//SUI CLI PTB:
// 語法： sui client ptb 
// 地址需要加上 「 @ 」符號。
// --assign <NAME> <VALUE> :  將<VALUE> 綁定到 <NAME> 變數
// --transfer-objects "<[OBJECTS]>" <TO>： 轉移 Object ，注意 <[OBJECTS]> 是 Array
// --split-coins <COIN> "<[AMOUNT]>" : Coin 切割操作。
// --merge-coins <INTO_COIN> "<[COIN OBJECTS]>":  Coin 合併操作
// --move-call <PACKAGE::MODULE::FUNCTION> "<TYPE>" <FUNCTION_ARGS>: 執行合約 Function，注意： <TYPE> 需要用 「""」包起來， Ex: "0x2::sui::SUI"
// --dry-run: 試跑，但不會真的發生在鏈上。
```


用 module3 nft 合約來測試

```
sui client ptb \
--assign PACKAGE_ID @0x2451988b564b189a0f35d6b4a5dce127dba0e43eab5d570c4d5f4121050e3d32 \
--move-call PACKAGE_ID::rich_capy::mint @0xe61b3052fc2b307de70ecb3ca27b76e47866ce025335f03b0143aff7de2b4a9f '"kais3"' \
--move-call PACKAGE_ID::rich_capy::mint @0xe61b3052fc2b307de70ecb3ca27b76e47866ce025335f03b0143aff7de2b4a9f '"kais4"'
```


## 官方 Coin module

```move
public struct Coin<phantom T> has key, store {
    id: UID,
    balance: Balance<T>,
}
```

通常在處理錢時，不會想要轉來轉去

```move
public fun split<T>(self: &mut Coin<T>, split_amount: u64, ctx: &mut TxContext): Coin<T> {
    take(&mut self.balance, split_amount, ctx)
}
```

可以看到都在處理 Balance


因此通常在處理 Coin 時，都把它拆開

```move
/// Destruct a Coin wrapper and keep the balance.
public fun into_balance<T>(coin: Coin<T>): Balance<T> {
    let Coin { id, balance } = coin;
    id.delete();
    balance
}
```

當處理完後後要轉移時又會在封裝回 Coin

```move
public fun from_balance<T>(balance: Balance<T>, ctx: &mut TxContext): Coin<T> {
    Coin { id: object::new(ctx), balance }
}
```

那為什麼不用 Coin 而是直接使用 balance。以 split 來說，如果都使是以 Coin 來操作，在 PTB 中會一直產生 Coin 一直產生 id，因此會產生一些手續費
-> 少了產生 id 的行為，因此可以少些手續費



