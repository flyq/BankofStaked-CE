<p align="center">
  <img width="200" height="200" src="./BankofStaked-logo.png">
</p>

<h1 align="center">
BankofStaked: CPU&NET leasing Smart Contract
</h1>


### About
Bank of Staked is an EOS smart contract aiming to provide cheap CPU and NET lease to both EOS users and EOS developers. This contract is build by `EOSLaoMao Team`.

### Design

The UX Bank of Staked wants to achieve is that user can simply transfer certain amount of EOS, then they can get delegetad resource automatically. And the undelegate process is also trigged automatically.

The main logic of Bank of Staked are formed by these three tables:

#### 1. Plan Table

Plan table holds all avialible plans user can choose, fields are:

```
price			asset; //plan price
cpu			asset; // delegated cpu this plan offers.
net			asset; // delegated net this plan offers.
duration			unit64; //the period of time service gonna last, in minutes.
is_free			uint64; //free plan or not, if is_free is 1, order will be auto refunded.
...
```

#### 2. Creditor Table

`Creditors` are the real ones do delegate and undelegate. When a valid transfer happens, the contract will try to find `is_active` creditor to do auto delegation.

```
account			account_name;
is_active			uint64;
...
```

`is_active` indicates if this creditor is ready to delegate or not.

in production, you should always have creditors shifting like 3 days in a roll, so that non-active creditors have time to get their undelegated token back.

#### 3. Order Table

Order table consists of records of active orders, by active, we mean these orders are not expired.

```
buyer			account_name;
creditor			account_name;
beneficiary			account_name;
cpu_staked			asset;
net_staked			asset;
expire_at			uint64;
...
```

`buyer` is the account who did the actual transfer in the first place, if any refund happens, token will be refunded to `buyer` account also.

`beneficiary` is the actual account who get cpu and net delegated. `buyer` can specify this account in transfer memo if he/she like.

`creditor` is the account who did delegation.

`cpu_staked` and `net_staked` is cpu&net delegated in this order.

`expire_at` is when this order will expire. After order expired, order record will be deleted from Order Table.


There are also several other tables facilitating this contract. such as,

`freelock` table is used to lock free plan for each account for 24 hours.

`history` table is used to store meta data of deleted expired order.

`blacklist` is used to blacklist certain account from using `bankofstaked` contract.


![Process](./Order-Process-of-BankofStaked.svg)


# Bank of Staked on Mainnet

### How to use it

We have deployed it on Mainnet: 

https://www.myeoskit.com/#/tx/bankofstaked


#### 1.Check available price plan

Query `plan` table to get available plan:


```
cleos -u https://api1.eosasia.one get table bankofstaked bankofstaked plan

{
  "rows": [{
      "id": 0,
      "price": "0.1000 EOS",
      "is_free": "1",
      "cpu": "1.0000 EOS",
      "net": "0.0000 EOS",
      "duration": 60,
      "created_at": 1535965927,
      "updated_at": 1535965927
    },
  ],
  "more": false
}
```

`price` indicates the EOS you should transfer to `bankofstaked` to get certain `cpu` and `net` for `duration` minutes.

`is_free` means if you can get a refund immediately!!

Currently we only provide 1 FREE plan: `0.1 EOS` for `60 min of 1 EOS CPU` staked.

Will add more plans ASAP.


#### 2.Transfer EOS and get delegated!


Currently, you can transfer `0.1 EOS` to `bankofstaked` to get `1 EOS` staked for `cpu` for `60 minutes`. 

NOTE: `bankofstaked` will automatically return the token you transfered back to you immediately.



```
cleos -u https://api1.eosasia.one transfer YOUR_ACCOUNT bankofstaked "0.1 EOS" -p YOUR_ACCOUNT@active
```

After transfering token, you will be delegated `1 EOS of CPU` to you , after 60 minutes, it will undelegate automatically using deferred transaction.



---

if you want to contribute your EOS to creditor fund(only delegate, no transfer needed), please contact us at: contact@eoslaomao.com

built with love by EOSLaoMao Team. :)
