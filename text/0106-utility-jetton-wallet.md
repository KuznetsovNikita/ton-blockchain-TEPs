- **TEP**: [0](https://github.com/ton-blockchain/TEPs/pull/0) *(don't change)*
- **title**: Utility Jetton Wallet
- **status**: Draft
- **type**: Contract Interface
- **authors**: [Nikita Kuznetsov](https://github.com/KuznetsovNikita)
- **created**: 12.01.2023
- **replaces**: -
- **replaced by**: -

# Summary

This proposal suggests extending the standard Jetton Wallet by adding the operation `proxy` to proxy messages with jetton information.

# Motivation

The application developers may want to be able implement an utility jetton to provide additional value for jetton holders.

# Guide

Upon jetton wallet receiving `proxy` message with target address, it should include jetton balance and jetton wallet owner to message and send it to target address.

# Specification

## New Jetton Wallet contracts
Example of utility jetton wallet code can be found [here](https://github.com/OpenProduct/openmask-token/blob/main/contracts/jetton-wallet.fc)

Jetton Wallet should handle message

`proxy#1ee6e170 query_id:uint64 response_destination:MsgAddress forward_payload:Either Cell ^Cell = InternalMsgBody;`

**Should be rejected if:**

1. message is not from the owner.
2. there is empty jettons balance on the sender wallet
3. After processing the request, the receiver's jetton-wallet **must** send at least  `2 * max_tx_gas_price + fwd_fee` ~ 0.026 TON for current basechain settings to the `response_destination` address.
    If the sender jetton-wallet cannot guarantee this, it must immediately stop executing the request and throw error.
    `max_tx_gas_price` is the price in Toncoins of maximum transaction gas limit of FT habitat workchain. For the basechain it can be obtained from [`ConfigParam 21`](https://github.com/ton-blockchain/ton/blob/78e72d3ef8f31706f30debaf97b0d9a2dfa35475/crypto/block/block.tlb#L660) from `gas_limit` field.  `fwd_fee` is forward fee for transfer request, it can be obtained from parsing transfer request message.

**Otherwise should do:**

Jetton wallet send message to `response_destination` address with `forward_amount` nanotons attached and with the following layout:
   TL-B schema:

```
proxy_notification#5df38bce query_id:uint64 amount:(VarUInteger 16)
                              owner:MsgAddress forward_payload:(Either Cell ^Cell)
                              = InternalMsgBody;
```

`query_id` should be equal with request's `query_id`.

`amount` is jetton wallet balance.

`owner` is jetton wallet owner,

`forward_payload` should be equal with request's `forward_payload`.


## Scheme:

```
crc32('proxy query_id:uint64 response_destination:MsgAddress forward_payload:Either Cell ^Cell = InternalMsgBody') = 1ee6e170
crc32('proxy_notification query_id:uint64 amount:VarUInteger 16 owner:MsgAddress forward_payload:Either Cell ^Cell = InternalMsgBody') = 5df38bce
```

# Drawbacks

The solution required a custom implementation for the proxy handler contract to process the `proxy_notification` message. The handler contract should include a jetton wallet code and will use gas for storage.

# Rationale and alternatives

-

# Prior art

-

# Unresolved questions

-

# Future possibilities

Implement an application to provide an addition feature for jetton owners. The simple jetton handler implementation can be found [here](https://github.com/OpenProduct/openmask-token/blob/b8bc49056d7065b52ecb7b414b29ac95e965a764/contracts/proxy-handler-example.fc#L60)  
