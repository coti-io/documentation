# Security

The COTI Wallet Plugin is designed with a zero-persistence security model. All sensitive cryptographic material exists only in memory during the active browser session.

## Memory-Only Keys

AES keys live exclusively in React state and a module-level singleton cache. They are never written to localStorage, sessionStorage, IndexedDB, or cookies.

## Ephemeral by Design

Keys are lost on page refresh. Users must re-authenticate, eliminating persistent attack surface.

## Singleton Cache

The internal AES key cache (`globalAESKeyCache`) is a module-scoped variable shared across all hook instances within a single browser tab. This is intentional for performance (avoids re-prompting the user on every component mount) but means the plugin is designed exclusively for **browser SPA environments**.

{% hint style="warning" %}
The singleton cache is not compatible with SSR or React Server Components.
{% endhint %}

## Sanity Guards

Decryption includes threshold checks to detect AES key mismatches before displaying garbage values.

## No Network Transmission

AES keys are never sent over the network. All decryption is client-side.

## Connector Identity

Wallet type detection uses wagmi's stable `connector.id`, not spoofable `window.ethereum.isMetaMask`.

## Session Isolation

`sessionAesKey` is automatically cleared on account change, disconnect, or manual lock. The singleton cache is also cleared on these events via `clearSnapCache()`.
