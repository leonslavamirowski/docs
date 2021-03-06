# Scripts

As already mentioned, **dfinance** supports transaction scripting. It means users can compile and execute scripts. Different between modules here is that you can't deploy script and use it again in the future, each script executing by new transaction every time.

The Move Book also has a section about [scripts](https://move-book.com/chapters/function.html) in Move language.

## Write a script

Let's write a basic script, accepts two arguments, a and b values, and then using module math make a sum from these two numbers and then fire events.

```rust
script {
   use 0x0::Event;
   use {{sender}}::Math;

   fun main(a: u64, b: u64) {
      let sum = Math::add(a, b);

      let event_handle = Event::new_event_handle<u64>();
        Event::emit_event(&mut event_handle, sum);
        Event::destroy_handle(event_handle);
   }
}
```

Replace `{{sender}}` with the address you used during deploy of the module in the previous part of current documentation.

The script accepts two arguments in function **"main"**, then calculate sum with provided arguments, and fire event with this sum. Both arguments are **u64** integers.

Compile the script using **dncli**:

```text
dncli query vm compile-script <script file> <address> --to-file <output file>
```

And then execute with arguments:

```text
dncli tx vm execute-script <output file> 15 20 --from <my address> --fees 1dfi
```

You can verify execution with querying transaction by id.

There will be even fired event, that will contain **"keep"** status and the resulting sum, like:

```javascript
[
   {
      "type":"contract_events",
      "attributes":[
         {
            "key":"guid",
            "value":"0x030000000000000077616c6c657400000000000095abf6bf9cd39a391567e4508becb25d0f1b98de"
         },
         {
            "key":"sequence_number",
            "value":"0"
         },
         {
            "key":"type",
            "value":"U64"
         },
         {
            "key":"data",
            "value":"0x2300000000000000"
         }
      ]
   },
   {
      "type":"contract_status",
      "attributes":[
         {
            "key":"status",
            "value":"keep"
         }
      ]
   }
]
```

