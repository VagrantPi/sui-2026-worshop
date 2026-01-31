# Move

init project

`sui move new << name >>`

## 專案架構

所以任何合約的進入點就是看 Move.toml 在哪，將來在需要 import 時就是這樣找

```shell
.
├── Move.toml            - 設定
├── sources              - 合約，每一個檔案都是一個合約
│   └── demo.move
└── tests                - 測試
    └── demo_tests.move
```

### Move.toml

```move
[package]
name = "demo"
edition = "2024" - move 版本

// implicit-dependencies = true      如果想要完全的客製化 dependencies

[dependencies]
// 底層其實會偷偷 import sui 官方合約 
sui = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "main" }
// 解釋 {{ Move.toml package name }}  = { git=repo, subdir=位置, rev=git版本 }
// 解釋 {{ rename }} = { git=repo, subdir=位置, rev=git版本, rename-from={{ Move.toml package name }} }
```

sui 部署時會是整包合約吃同一個

### 部署

`sui client publish`

```
...
╭──────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                   │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0xcbb2232509e94c89cff2e1ce4e3a04a6b227efa8feaef5736f2a70a6d8a83d1d                  │
│  │ Sender: 0x12c1e3db35e33196cf1c64331100b6534d94a1c3b96d20bbc9986f82f3e6ed32                    │
│  │ Owner: Account Address ( 0x12c1e3db35e33196cf1c64331100b6534d94a1c3b96d20bbc9986f82f3e6ed32 ) │
│  │ ObjectType: 0x2::package::UpgradeCap                                                          │
│  │ Version: 349181274                                                                            │
│  │ Digest: 7GBxKYURKfyW3KKC4RPtE6cZj8dfkCPNBq8CHAzPkrBf                                          │
│  └──                                                                                             │
│ Mutated Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x222b3cacbab545deac75753870269b933e12a16cd1a550ff11ebf6b7158ae0cd                  │
│  │ Sender: 0x12c1e3db35e33196cf1c64331100b6534d94a1c3b96d20bbc9986f82f3e6ed32                    │
│  │ Owner: Account Address ( 0x12c1e3db35e33196cf1c64331100b6534d94a1c3b96d20bbc9986f82f3e6ed32 ) │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                    │
│  │ Version: 349181274                                                                            │
│  │ Digest: 2SPPvn2acMtAnvWxHUbhv6qN3Yfci5o1kUruwtbe5BP1                                          │
│  └──                                                                                             │
│ Published Objects:                                                                               │
│  ┌──                                                                                             │
│  │ PackageID: 0xb8209ba5d73c4dd4de6472f30e4da5610fd192926b53331314cc597bc09e6155                 │
│  │ Version: 1                                                                                    │
│  │ Digest: H4UeW8D1AFaDxoK7ggvPVSrBZfbSe1ieNdxAC52utCUY                                          │
│  │ Modules: demo                                                                                 │
│  └──                                                                                             │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance Changes                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──                                                                                              │
│  │ Owner: Account Address ( 0x12c1e3db35e33196cf1c64331100b6534d94a1c3b96d20bbc9986f82f3e6ed32 )  │
│  │ CoinType: 0x2::sui::SUI                                                                        │
│  │ Amount: -3783880                                                                               │
│  └──                                                                                              │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

可以看到合約地址

```
│ Published Objects:                                                                               │
│  ┌──                                                                                             │
│  │ PackageID: 0xb8209ba5d73c4dd4de6472f30e4da5610fd192926b53331314cc597bc09e6155                 │
│  │ Version: 1                                                                                    │
│  │ Digest: H4UeW8D1AFaDxoK7ggvPVSrBZfbSe1ieNdxAC52utCUY                                          │
│  │ Modules: demo                                                                                 │
│  └──                                                                                             │
```

## 基礎知識

部署時會將 Move.toml 根目錄(Package)整包打包上傳

source 內的每一個檔案都是合約 Module

因此我要呼叫時

`package::module::func`


合約更新時都會是新的 Object id

因此假設你有一個 Dex 內的小功能修改，也會是重新上傳一個新的

因此 dependencies 越多層，最裡面如果異動，這會是風險




## 合約起手式

```move
// 兩種寫法
// module <<Move.toml name>>::<<module name>> {}
// module <<Move.toml name>>::<<module name>> ;

module module_2::template;
```

## import

同上面基礎知識

```move
// import 單一
// use << dependencies 中的 package name >>::<< module >>::<< func name >>
// import 多個
// use << dependencies 中的 package name >>::<< module >>::{ << func name >>, ... }
use sui::bag::Bag;
```

但是 func 一定會超多，不可能一個一個 import
```move
// use << dependencies 中的 package name >>::<< module >>::{Self}
use sui::bag::{Self}; // 使用時 Bag
// 等同
use sui::bag; // 使用時 bag::Bag
```

## 語法

參考講師 Module2 內的 03_basic_syntax.move

下面為補充

```move
// 數字
let num1: u64 = 1;
let num2: u8 = 2;
let num3 = 3u32;
let num4 = 4; // 沒指定預設會是 u64
let sum = num1 + (num2 as u64)

// 地址需加 @
let xxAddress = @0x2
```

變數宣告皆為不可變，如果是可以變得變數需要加上 mut

```move
let mut num1 = 1;
```

if 特別寫法

```move
// let k = 0
// if (num1 == 1) {
//     k = num1 + 1;
// } else {
//     k = 1;
// };

let k = if (num1 == 1) {
    return num1 + 1
} else {
    return 1
};
```

需注意 return 後不能有 `;`

而沒有 `;` 會視為回傳，因此上面寫法可以改成

```
let k = if (num1 == 1) {
    num1 + 1
} else {
    1
};
```


定義 Error

```move
// abort, assert! 回傳的 error code 一定要是 u64
const ENumTooLarge: 64 = 2; 
```

## Function

有五種

### Init Function: 部署合約會直接執行的 Function，只執行一次

```move
// use sui::tx_context::TxContext // 鏈上狀態，預設帶入，所以可以不用特別寫
// ctx - current transaction contex
func init(ctx: &mut TxContext) {
    
}
```

### Prive function

!!! scope 限制在單一 move 合約檔案內


```move
fun add (
    num1: u64,
    num2: u64,
): u64 {
    let sum = num1 + num2;
    sum
}

// 回傳複數資料
fun add (
    num1: u64,
    num2: u64,
): (u64, u64) {
    let sum = num1 + num2;
    (sum, num1)
}
```

### Package function

!!! scope 限制在同一個 package 內

```move
public(package) add_package(
    num1: u64,
    num2: u64,
): u64 {
    add(num1, num2)
}
```

在另外一個 package 底下的合約要使用一樣要 import

```move
// use << package >>::<< module >>::<< func >>
module module_2::functino_sample1;


add_package()
```

### Public function

```move
public fun ...
```

### Entry function

唯一限制，不能有回傳值，如果要有回傳值一調要有 drop 能力

基本用不太到，public 就能做到他的方法

```move
entry fun ...
```


## Struct

大駝峰命名

```move
// 
// use std::string::String;
// use std::object::UID;

public struct Student has key {
    id: UID,
    name: String,
    score: Score,
}

public struct Score has store {
    math: u8,
    eng: u8,
}
```

### Move 中的 struct 有 4 個能力

- key
    基本上有 key 就是 object -> 資產
    因此宣告時如果沒有給 id 參數，編譯會報錯

    並且因為是 Object 所以會存在 sui 區塊鏈上，還記得
- store
    可以存在鏈上，但需要依附在有 key 的 struct 裡面
- copy
    ```move
    // B 複製一份並且 assign 給 A 變數，因此如果沒有 copy 能力時會噴錯
    let Score2 = Score;  // 會噴錯
    ```


    基本的資料型態都包含了三個能力 `store`, `copy`, `drop`
    ```move
    fun add (
        num1: u64,
        num2: u64,
    ): u64 {
        let sum = num1 + num2;
        sum
    }
    ```

- drop

    ```move
    fun add (
        num1: u64,
    ): u64 {
        let test1 = num1;
    }
    ```


    ```move
    fun add (
        student: Student, // 只有 key 能力
        receiver: address, // 有 drop 能力
    ): u64 {
        let test1 = student; // Student 只有 key 所以這邊會噴錯，因為不能 copy
        
        // 此外這邊有個重點，如果 student 傳入後沒有做釋放，會噴錯
        // 因此沒有 drop 能力的變數被傳入後要能正常離開有三種方法
        // - 指定 ownership
        //     - transfer::transfer(student, address);
        //     - transfer::share_object(student);
        //     - transfer::freeze_object(student)'
        // - burn objcet
        // 1. 先 destruct
        /*
            let Sutdent{
                id: std_id, // UID has store
                name: std_name, // String 
                score: std_score, // Score
            } = student;

           2. 分別判斷變數有沒有 drop
            後續使用時就只能使用
            std_id
            std_name
            std_score

            所以應該要分別去看底層是否有 drop 能力
            - UID has store
            - String 有 drop -> 因此不用處理
            - score 我這邊定義的

            所以先拆 Score
            let Score{
                math: std_math, // u8 - 基本類型都有 drop 能力
                eng: std_eng, // u8
            } = Score;


            // UID 不是這個 module 的，因此只有那一包 package 存在可以解開的 fun
            object::delete(std_id)
        */

    }
    ```


四個能力無法同時存在
-> key + store => O 可以將資產放進 pool object
-> key + copy => X 你的資產串如後不能任意複製
-> key + drop => X 你的資產傳入後，不能因為沒轉移，離開後自動 drop


key + store 官方文件有寫
- transfer::public_transfer(student, address);
- transfer::public_share_object(student);
- transfer::public_freeze_object(student)'


範例

```move
public fun set_math(
    student: Student,
    math: u8,
) {
    student.score.math = math;  // 這邊會噴錯，原因是因為 Student 沒有 drop
}


public fun set_math(
    student: Student,
    math: u8,
    ctx: &mut TxContex,
) {
    student.score.math = math;  // 這邊會噴錯，原因是因為 Student 沒有 drop
    transfer::pub_transfer(student, tx_context::sender(ctx))
}


// & Student - 引用但不可改
// &mut Student - 引用且可改
public fun set_math(
    student: &mut Student,
    math: u8,
) {
    student.score.math = math;
}

```


struct 產出方式

> 只要牽涉到 id 都要帶入 TxContex
```move
public fun new(
    name: String,
    math: u8,
    eng: u8,
    ctx: &mut TxContex,
) {
    let score = Score{
        math, // math: math 可以簡寫
        eng,
    }
    
    let student = Student{
        id: object::new(ctx), // 產生 id 的方式永遠都是這樣
        name
        score
    }

    transfer::publiic_transfer(student, tx_context::sender(ctx))
}
```

## 呼叫合約

package - module - func


ctx 一率不用帶

```
sui client call --package 0x... --module xxxx --function new --args Paul 100 100
```
