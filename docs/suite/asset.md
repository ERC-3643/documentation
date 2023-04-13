# Asset

The asset smart contract is usually a token (the default implementation is an ERC20 token).

Whenever a transfer is initiated, the asset contract must, usually in the transfer method:
- Call the Identity Registry `.isVerified(recipientAddress)` method to verify that the recipient is associated with a
compliant identity and revert if this method returns `false`.
- Call the Compliance `.canTransfer(sourceAddress, recipientAddress, amount)` method to verify that the transfer is
compliant with distribution rules (for NFT, the canTransfer method could include the token ID) and revert if this
method returns `false`.

Whenever a transfer is executed (or a mint or a burn), the asset contract must signify the compliance some assets have
changed ownership, so that the compliance modules can keep track of them according to their rules.
- Call the Compliance `.transferred(from, to, amount)` when a transfer is executed.
- Call the Compliance `.created(to, amount)` when a mint is executed.
- Call the Compliance `.destroyed(from, amount)` when a burn is executed.

It is also encouraged to keep in the smart contract a `version` of the ERC3643 protocol used.
