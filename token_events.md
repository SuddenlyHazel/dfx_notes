# Proposal for Token Event Design

# Introduction

**Summary**

One of TIC’s most potent features is its [reliable](https://sdk.dfinity.org/docs/language-guide/motoko-introduction.html#_asynchronous_messaging_and_type_sound_execution), [cheap](https://github.com/dfinity/ic/blob/779549eccfcf61ac702dfc2ee6d76ffdc2db1f7f/rs/config/src/subnet_config.rs#L152), [easy to utilize](https://sdk.dfinity.org/docs/language-guide/actors-async.html) asynchronous messaging system. TIC makes it incredibly simple for developers to build event driven systems. Token implementations on TIC should should be capable of publishing an event stream either to an intermediary message broker, or directly to subscribers. This document outlines some use cases and a minimum framework for consideration.
 
**Why?**

Exposing an event stream from IC Tokens will allow IC services to immediately react to important token events. For instance, let us pretend a wrapped cycles token becomes popular. A developer building a digital-goods service might subscribe to a `TransferEvent` on their services token address. As users transfer their tokens to this services address, the service would automatically be able to respond to the events, and grant the users whatever they’re purchasing. 


**Considerations**

Given emitting message [has a cost](https://github.com/dfinity/ic/blob/779549eccfcf61ac702dfc2ee6d76ffdc2db1f7f/rs/config/src/subnet_config.rs#L152), albeit a small one, and there is an [upper bound on cycles per message](https://github.com/dfinity/ic/blob/779549eccfcf61ac702dfc2ee6d76ffdc2db1f7f/rs/config/src/subnet_config.rs#L11). Implementations of this event system should:

1. Make sure a reasonable transaction fee is in place to prevent cycle-draining attacks.
2. Consider offloading subscription mapping logic to another canister.
# Types

The following are the absolute minimum needed events I believe we need.


## Event Broker

May or may not be the contract principal. Flexibility included to allow for horizontal scaling / offloading to a message broker. For example, imagine some binning function `bin`;  `bin(to) => Principal`.


    public type EventBroker = Principal;


## TokenEvent (Supertype)


    public type TokenEvent = {
        #transferEvent : TransferEvent;
        #accountEvent : AccountEvent;
    };

**TransferEvent**

Emitted once per transaction. Sent to the subscribers of the `to` and `from` props.
Metadata should be limited to a reasonable size - maybe len 1000, and not persisted. 
Metadata should be relayed from the transfer call, maybe a `transferWithMetadata` method.

    public type TransferEvent = {
        to : Principal;
        from : Principal;
        amount : Nat;
        metadata : ?Blob;
    };


## AccountEvent (Supertype)

****
    public type AccountEvent = {
        #approvalEvent : ApprovalEvent;
    };

**ApprovalEvent**

Emitted once per approval operation. Sent to the subscribers of the `to` and `from` props.


    public type ApprovalEvent = {
        owner: Principal;
        spender : Principal;
        amount : Nat;
    };

**SubscribeMessage**


    public type SubscribeMessage = {
        callback: shared (event : TokenEvent) -> async ();
        account: Principal;
    };
# Methods

**addSubscription**

Adds a subscription. Caller should be authorized to act on `account`.


      addSubscription: shared (account: SubscribeMessage) -> async ();

**removeSubscription**

Removes a subscription. Caller should be authorized to act on `account`.


      removeSubscription: shared (account: Principal, subscriber: Principal) -> async ();
# Thoughts
1. Token Implementers might consider charging a token fee for registering subscriptions.


