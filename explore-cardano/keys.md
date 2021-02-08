# Keys

Keys are asymmetric cryptography key pairs that are used for:

- Signing and validating payments and staking certificates
- Identifying and defining addresses on the Cardano blockchain.

## Types of keys

There are two main type of keys in Shelley:
* Node keys
* Address keys

### Node keys

Node keys represent the security of the blockchain and consist of the following keys:
* Operator/operational key, operator's offline key pair with cert counter for new certificates.
* Hot KES key, operator's hot KES key pair.
* Block signing key, operational VRF key pair, it participates in the right to create and sign the block for the specific slot.

### Address keys

Address keys represent the functions of the addresses derived from the keys for identifying funds on the blockchain that consist of the following keys:

Payment key, single address key pair usually used for generating UtxO addresses.
Staking key, stake/reward address key pair usually used for generating account/reward addresses.
