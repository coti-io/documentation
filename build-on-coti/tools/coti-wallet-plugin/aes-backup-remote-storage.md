# Secure remote AES backup storage

{% hint style="warning" %}
**Deprecated.** Remote AES key backup is **not** a supported product path and is expected to be removed. Do **not** build new integrations against remote AES backup APIs.
{% endhint %}

The **only supported** encrypted-backup persistence path is browser **`localStorage`** via host `onboardingServices` callbacks (`mode: 'custom'`). See [AES backup security model](aes-backup-security.md) and [Configuration](configuration.md).

This page is retained only so existing links do not break. Ignore remote storage auth helpers and challenge-based API designs for new work.

## Related docs

* [AES backup security model](aes-backup-security.md)
* [AES Key Onboarding](aes-key-onboarding.md)
* [Configuration](configuration.md)
