# Standard Library

Standard **Move VM** library is default modules that already developed and developers can use in developing new modules, scripts.

They all placed on the address **0x0**. So when you import something from **0x0**, you import standard modules, like:

```rust
use 0x0::Account;
use 0x0::Events;
use 0x0::DFI;
use 0x0::Coins;
...
```

You can look for actual standard modules in [dvm](https://github.com/dfinance/dvm/tree/master/lang) repository.

## Time

[Time](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/time.move#L3) module allows getting current UNIX timestamp of latest block.

Example:

```rust
script {
    use 0x0::Time;

    fun main() {
        let _ = Time::now();
    }
}
```

The method will return u64 value as UNIX timestamp of the latest block.

## Block

[Block](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/block.move#L3) module allows getting current blockchain height.

```rust
script {
    use 0x0::Block;

    fun main() {
        let _ = Block::get_current_block_height();
    }
}
```

The method will return u64 value as the height of the latest block.

## Transaction

[Transaction](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/transaction.move#L3) module contains functions to work with transaction data, currently supports two functions: `sender()`, `assert(bool, u64)`. 

Getting sender address of transaction:

```rust
script {
    use 0x0::Transaction;

    fun main() {
        let _ = Transaction::sender();
    }
}
```

Assert:

```rust
script {
    use 0x0::Transaction;

    fun main() {
        let a = 10;
        let b = 11;
        Transaction::assert(a == b, 101);
    }
}
```

In case you pass `false` as the first argument of `assert(bool, u64)` or the result of your expression, the transaction will fail and return "sub_status" in event of transaction that will equal your code provided as the second argument.

## Compare

[Compare](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/compare.move#L10) module allows comparing two vectors of u8 values (bytes).

Comparing two-byte vectors:

```rust
script {
    use 0x0::Compare;
    use 0x0::Transaction;

    fun main() {
        let a = x"00";
        let b = x"01";
        Transaction::assert(Compare::cmp_lcs_bytes(&a, &b) == 0, 101);
    }
}
```

## DFI && Coins

[DFI](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/dfi.move#L7) and [Coins](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/coins.move#L6) modules allow to get a type of currency that you going to use in your code.

```rust
script {
	use 0x0::Account;
	use 0x0::DFI;
	use 0x0::Coins;

	fun main(payee: address, dfi_amount: u128, eth_amount: u128, btc_amount: u128, usdt_amount: u128) {
        Account::pay_from_sender<DFI::T>(payee, dfi_amount);
        Account::pay_from_sender<Coins::ETH>(payee, eth_amount);
        Account::pay_from_sender<Coins::BTC>(payee, btc_amount);
        Account::pay_from_sender<Coins::USDT>(payee, usdt_amount);
	}
}
```

## Oracle

[Oracle](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/oracle.move#L3) module allows to get the current price of an asset pair.

```rust
script {
    use 0x0::Oracle;

    fun main(ticker: u64) {
        let _ = Oracle::get_price(ticker);
    }
}
```

More about work with oracles can see in our [oracles documentation](/oracles/README.md).

## Event

Event module allows us to work with events: generate new event handlers and fire events.

Example with firing event contains provided number:

```rust
script {
    use 0x0::Event;

    fun main(a: u64) {
        let event_handle = Event::new_event_handle<u64>();
        Event::emit_event(&mut event_handle, a);
        Event::destroy_handle(event_handle);
    }
}
```

Or you can store event handler in resource and fire events when you need:

```rust
module MyEvent {
    use 0x0::Event;
    use 0x0::Transaction;

    resource struct T {
        eh: Event::EventHandle<u64>,
    }

    public fun init() {
        move_to_sender<T>(T {
            eh: Event::new_event_handle(),
        });
    }

    public fun fire_event(a: u64) acquires T {
        let i = borrow_global_mut<T>(Transaction::sender());

        Event::emit_event(
            &mut i.eh,
            a
        );
    }
}
```

## Account

[Account](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/account.move#L6) module allows to work with user balances: get balances, deposit coins/tokens to balances, withdraw them to deposit in another module, etc.

Also, it creates an account, if the account doesn't exist yet, and related data, like event handlers for sending/receiving payments.

A lot of different methods can be used to send tokens from account A to account B, as these one-line methods:

```rust
script {
    use 0x0::Account;
    use 0x0::DFI;
    
    fun main(payee: address, amount: u128, metadata: vector<u8>) {
        // Move DFI from sender account to payee.
        Account::pay_from_sender<DFI::T>(payee, amount);

        // Again move DFI, but with metadata.
        Account::pay_from_sender_with_metadata<DFI::T>(payee, amount, metadata);
    }
}
```

Also, you can just withdraw from sender balance and deposit to payee:

```rust
script {
    use 0x0::Account;
    use 0x0::DFI;

    fun main(payee: address, amount: u128) {
        // Move DFI from sender account to payee.
        let dfi = Account::withdraw_from_sender<DFI::T>(amount);

        // Again move DFI, but with metadata.
        Account::deposit(payee, dfi);
    }
}
```

Or deposit to another module:

```rust
script {
    use {{address}}::Swap;
    use 0x0::DFI;
    use 0x0::Coins;
    use 0x0::Account;

    fun main(seller:address, price: u128) {
        let dfi = Account::withdraw_from_sender(price);

        // Deposit USDT to swap coins.
        Swap::swap<Coins::USDT, DFI::T>(seller, dfi);
    }
}
```

Also, get a balance:

```rust
script {
    use 0x0::Coins;
    use 0x0::Account;
    use 0x0::Transaction;

    fun main(addr: address) {
        // My balance.
        let my_balance = Account::balance<Coins::ETH>();

        // Someone balance.
        let someone_balance = Account::balance_for<Coins::ETH>(addr);

        Transaction::assert(my_balance > 0, 101);
        Transaction::assert(someone_balance > 0, 102);
    }
}
```

The rest of the features of Account module look at [account.move](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/account.move#L6). 

## Dfinance

[Dfinance](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/dfinance.move#L6) module allows you to work with coins balances, get coins info, also register new tokens, etc.

First of all, Dfinance module presents type for all balances in the system, it's `Dfinance::T`:

```rust
resource struct T<Coin> {
    value: u128
}
```

The value field contains information about actual balance for specific coin/token, e.g.:

```rust
script {
    use 0x0::Account;
    use 0x0::DFI;

    fun main(amount: u128) {
        // Use DFI::T to get Dfinance::T<DFI::T> contains balance.
        let dfi : Dfinance::T<DFI::T> = Account::withdraw_from_sender<DFI::T>(amount);
        Account::deposit_to_sender(dfi);
    }
}
```

Also, you can create an empty coin:

```rust
module BankDFI {
    use 0x0::Dfinance;
    use 0x0::DFI;

    resource struct T {
        balance: Dfinance::T<DFI::T>,
    }

    public fun create()  {
        move_to_sender<T>(T {
            balance: Dfinance::zero<DFI::T>()
        })
    }
}
```

Get denom, decimals, and actual value:

```rust
script {
    use 0x0::Dfinance;
    use 0x0::Account;
    use 0x0::DFI;
    use 0x0::Transaction;

    fun main(amount: u128) {
        let dfi = Account::withdraw_from_sender<DFI::T>(amount);

        // Get denom vector<8>.
        let _ = Dfinance::denom<DFI::T>();

        // Get value of withdrawed dfi.
        let value = Dfinance::value(&dfi);

        Transaction::assert(amount == value, 101);

        Account::deposit_to_sender(dfi);
    }
}
```

And check if it's user token or system coin:

```rust
script {
    use {{address}}::MyToken;
    use 0x0::Dfinance;
    use 0x0::DFI;

    fun main() {
        Transaction::assert(Dfinance::is_token<DFI::T>() == false, 101);
        Transaction::assert(Dfinance::is_token<MyToken::T>(), 102);
    }
}
```

Also, you can create your resource and make it token too!

```rust
module MyToken {
    use 0x0::Dfinance;

    resource struct Token {
    }

    public fun create(): Dfinance::T<Token>  {
        // Create new token with denom "wow" (hex == 776f77).
        Dfinance::tokenize<Token>(10, 0, x"776f77")
    }
}
```

And also deposit it to your balance:

```rust
script {
    use wallet16ehqyls5mn73kk54vvh6kyjlsrkwh37tjtzesu::MyToken;
    use 0x0::Account;

    fun main() {
        let new_tokens = MyToken::create();
        Account::deposit_to_sender(new_tokens);
    }
}
```

More documentation about the feature provided by Dfinance module see in [dfinance.move](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/dfinance.move#L6). 

## Vector

[Vector](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/vector.move#L4) module contains functions to work with `vector`
 type.

For example:

```rust
script {
    use 0x0::Vector;

    fun main() {
        let v = Vector::empty<u64>();
        let i = 0;

        loop {
            if (i == 10) {
                break
            };

            Vector::push_back(&mut v, i);
            i = i + 1;
        };
    }
}
```

Vector module great describe in [Move Book](https://move-book.com/chapters/vector.html).

## Signature

[Signature](https://github.com/dfinance/dvm/blob/bf457b3145c5e448ece3258bbf67c22326559a12/lang/stdlib/signature.move#L3) module allows to verify ed25519 signature:

```rust
script {
    use 0x0::Signature;
    use 0x0::Transaction;

    fun main(signature: vector<u8>, pub_key: vector<u8>, message: vector<u8>) {
        let is_verified = Signature::ed25519_verify(signature, pub_key, message);
        Transaction::assert(is_verified, 101);
    }
}
```
