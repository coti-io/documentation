# Glossary

## Account

An Ethereum account is a digital identity on the Ethereum blockchain, allowing users to send, receive Ether, and interact with smart contracts.

**Technical:** Its an object containing an address, balance, nonce, and optional storage and code. An account can be a contract account or an externally owned account (EOA).

## Authenticated Memory

Protection mechanism which tranports data between the storage and the execution environment. The function first loads the ciphertext from storage into the execution environment memory, from which it may turn into a garbledtext.

## Authenticated Storage

Protection mechanism which protects ciphertexts ‘at rest’; each ciphertext is associated with one or more contract addresses in a way that permits onboarding of the ciphertext only in the execution of the relevant contracts.

## ciphertext (CT)

This data type represents the result of a CPA-secure encryption scheme and used for securing data at rest. It is the actual datatype visible in the system’s state.

## Garbled Circuits

A cryptographic protocol that enables two-party secure computation in which two mistrusting parties can jointly evaluate a function over their private inputs without the presence of a trusted third party.

## garbledtext (GT)&#x20;

This datatype is used to securely handle data while in use. Unlike inputtext and ciphertext, garbledtext is in a form that is readily available for manipulation (e.g., making arithmetics over the plaintext it hides) which is made inside the garbled execution environment.

## gcEVM

A propietary extension of the Ethereum EVM. It introduces new data types and operations that can perform manipulation on secret data types without disclosing the contents.

## inputtext (IT)

This data type is a wrapper of CT only used to infiltrated data to the gcEVM from the outside world. The role of IT is to make sure that the wrapped CT is used only for the purpose it is intended to by the user who sent it.

## MPC (multi-party computation)

A cryptographic protocol that allows multiple parties to share data and compute a function without revealing their private data. MPC is also known as secure computation or privacy-preserving computation.

