---
description: Transition identity from one voting Solana validator to another
icon: fingerprint
---

# Identity Transition

[Based on Identity Transition Demo by mvines](https://github.com/mvines/validator-Identity-transition-demo)

{% embed url="https://youtu.be/WPgE2L0AQJo?t=437" %}
Live identity transition demo (video starts at actual transition timestamp)
{% endembed %}

## Hot Swap Configuration Requirements

* Two non-delinquent validator nodes
* Junk identities on both validators to use when not actively voting
* Validator startup scripts both modified to use symbolic link as the identity
* Validator startup scripts both modified to include staked identity as authorized voter

### Generating Junk Identities

Both validators need to have secondary identities to assume when not actively voting. You can generate these "junk" identities on each of your validators like so:

```bash
solana-keygen new -s --no-bip39-passphrase -o unstaked-identity.json
```

### Validator Startup Script Modifications

The identity flag and authorized voter flags should be modified on **both** validators.

Note that `identity.json` is not a real file but a symbolic link we will create shortly - so don't worry about whether or not that file exists. However, the authorized voter flag does need to point to your staked identity file (your main identity). I renamed mine to `staked-identity.json` for clarity and simplicity. You can certainly name yours whatever you'd like, just make sure they are specified as authorized voters as shown below.

```bash
exec /home/sol/bin/agave-validator \
    --identity /home/sol/identity.json \
    --vote-account /home/sol/vote.json \
    --authorized-voter /home/sol/staked-identity.json \
```

Summary:

* Identity is a symbolic link we will create in the next section. It may point to your staked identity or your inactive "junk" identity.
* Vote account is exactly what you think it is. No changes. Just shown for context.
* Authorized voter points to your main, staked identity.

### Creating Identity Symlinks

An important part of how this system functions is the `identity.json` symbolic link. This link allows us to soft link the desired identity so that the validator can restart or stop/start with the same identity we last intended it to have.

On your actively voting validator, link this to your staked identity

```bash
ln -sf /home/sol/staked-identity.json /home/sol/identity.json
```

On your inactive, non-voting validator, link this to your unstaked "junk" identity

```bash
ln -sf /home/sol/unstaked-identity.json /home/sol/identity.json
```

### Transition Preparation Checklist

At this point on both validators you should have:

* Generated unstaked "junk" identities
* Updated your validator startup scripts
* Created symbolic links to point to respective identities

If you have done this - great! You're ready to transition!

### Transition Process

#### Active Validator

* Wait for a restart window
* Set identity to unstaked "junk" identity
* Correct symbolic link to reflect this change
* Copy the tower file to the inactive validator

```bash
#!/bin/bash

# example script of the above steps - change IP obviously
agave-validator -l /mnt/ledger wait-for-restart-window --min-idle-time 2 --skip-new-snapshot-check
agave-validator -l /mnt/ledger set-identity /home/sol/unstaked-identity.json
ln -sf /home/sol/unstaked-identity.json /home/sol/identity.json
scp /mnt/ledger/tower-1_9-$(solana-keygen pubkey /home/sol/staked-identity.json).bin sol@68.100.100.10:/mnt/ledger
```

(At this point your primary identity is no longer voting)

#### Inactive Validator

* Set identity to your staked identity (requiring the tower)
* Rewrite symbolic link to reflect this

```bash
#!/bin/bash
agave-validator -l /mnt/ledger set-identity --require-tower /home/sol/staked-identity.json
ln -sf /home/sol/staked-identity.json /home/sol/identity.json
```

#### Verification

Verify identities transitioned successfully using either monitor or `solana catchup --our-localhost 8899`
