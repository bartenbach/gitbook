---
description: How to operate a validator with no keys on the server
icon: key
---

# Keyless Operation

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

First, the startup arguments need to be changed in a way that does not depend on your keys, and you need to migrate your keys to a local machine somewhere.

For our main identity key, we will replace that with a junk identity. You can generate one of those like so:

```
solana-keygen new -s --no-bip39-passphrase -o junk.json
```

For the vote account, the key is actually not required at all. You can simply replace this with your vote pubkey. That looks like this:

```bash
exec ~/bin/agave-validator \
    --identity ~/junk.json \
    --vote-account CATvBs6b5xMebtLvieEdy9z4fcS3V8n3UjMuxhCnCGMS \
```

At this point, the validator needs to be restarted to start using the junk identity. Restart the validator in your usual way.

Once the validator comes back up and is running with the junk identity, we can simply set the identity remotely using the `set-identity` subcommand.\
\
Additionally, we need to set the authorized voter using the `authorized-voter` subcommand. If you fail to do this, your validator will not be able to vote.

```bash
ssh $USER@$IP "agave-validator --ledger $LEDGER set-identity" <identity.json
ssh $USER@$IP "agave-validator --ledger $LEDGER authorized-voter add" <identity.json
```

You should see confirmation that the change has taken place, and that's it. You have now set the identity and authorized-voter completely remotely - with none of your actual keys on the server.

### Note

If the validator does restart for whatever reason while using this model, it will restart using the junk identity and **not** your main, staked identity. So, you will need robust monitoring. If this happens, you will need to set your identity and authorized voter again remotely (and any time the validator is restarted).

