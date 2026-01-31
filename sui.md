# 2026 sui workshop

## Sui

- hight speed and scalability
- Object centric model

### Object centric model

區塊鏈上一個實體 - uid

與 account-base 剛好反過來

## Object

任一東西都是 Object 且都會有一個 uid
- coin
- NFT
- Pool
- Package(Contract)

### 轉移

最小單位為 Object

因此 Alice 手上如果有 10 sui(Object)，當他要轉錢給 Bob 時就會拆分出個 1 sui 的 Object 後轉出
而切分出的 1 sui 會是新的 Object id，原本的 Object id 上的 id 還是一樣

### 擁有權

- Owned Object
    - 有一個錢包擁有
- Share Object
    - 任何人都能『讀』『寫』
- Immutable Object
    - 任何人都能『讀』

對照回剛剛的 Object:

- Coin - Owned Object
- NFT - Owned Object
- Pool - Share Object
- Package(Contract) Immutable Object



## Directed Acyclic Graph(DAG)

共識機制

- Owned Object 本身的轉移 -> 因為 Obect 本身是獨立的，因此本人同意就好，不需要全網共識
- Share Object -> 需要走大共識

## Sui cli

- sui keytool import - 匯入
- sui client addresses - 顯示所有地址
- sui client gas - 顯示有多少 gas
- sui client envs - 網路環境
- sui client new-env - 新增節點
- sui client new-address ed25519 - 新增錢包
- sui client switch --address - 切換地址
- sui client switch --env - 切換網路

cli 開頭分類：
- sui keytool - 地址相關
- sui client - 節點相關
- sui move - 跟合約相關

### faucet

```shell
curl --location --request POST 'https://faucet.testnet.sui.io/v2/gas' \
--header 'Content-Type: application/json' \
--data-raw '{
    "FixedAmountRequest": {
        "recipient": "<< Address >>"
    }
}'
```

### 網路

- testnet - 測試鏈
- devnet - 開發鏈定期會清除

### pay-sui - 轉帳

sui decimal 10^8

```
sui client pay-sui \
  --input-coins <COIN_ID1> <COIN_ID2> \
  --recipients <RECIPIENT_ADDR1> <RECIPIENT_ADDR2> \
  --amounts <AMOUNT1> <AMOUNT2> \
  --gas-budget <GAS_BUDGET>
```

GAS_BUDGET - 預算 可以用 0.1 - 10000000

### split-coin 切分

切分本身的 Object 還是需要另外一個 Object 來支付手續費，因此當你錢包只有一個 object 的 coin 時是無法做切分的

```
sui client split-coin --coin-id <COIN_ID> --amounts <AMOUNTS>
```

### merge-sui 合併

需指定要合併回去的 Object

```
sui client merge-coin --primary-coin <COIN_ID> --coin-to-merge <COIN_ID>
```

