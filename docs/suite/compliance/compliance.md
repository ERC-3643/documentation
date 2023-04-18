import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Compliance

The `Compliance` contract is the contract that will verify the regulatory compliance of a transaction following pre-set
rules defined by the legal paperwork related to the security token. Apart from investor identity eligibility, the
Compliance will validate more general restrictions e.g. maintaining a max investor cap or a max tokens cap (as it might
be needed for certain securities in certain specific countries of distribution).

The contract is modular to support the addition of multiple general compliance rules as per the requirement of the token
issuer or the regulatory framework under which the token is operated. Modularity is assured by separate Module Contracts
that are bound to the Compliance Contract. Any interaction with the modules must be made using the Compliance contract.

## Methods

### canTransfer

This READ-ONLY function checks that a transfer is compliant with the rules defined by the modules bound to the contract.

An asset contract should call this method prior to executing a transfer.

| Parameter | Type    | Description                               |
|-----------|---------|-------------------------------------------|
| _from     | address | Address of the source of the transfer.    |
| _to       | address | Address of the recipient of the transfer. |
| _amount   | uint256 | Quantity of tokens transferred.           |

```solidity
function canTransfer(
    address _from,
    address _to,
    uint256 _amount
) view returns (bool);
```

### transferred

Function called whenever tokens are transferred from an address to another.
This function can update state variables in the modules bound to the compliance, these state variables being used by the
module checks to decide if a transfer is compliant or not depending on the values stored in these state variables and on
the parameters of the modules. In practice, it calls `moduleTransferAction()` on each module bound to the compliance
contract.

> :::info
> This function can be called ONLY by the token contract bound to the compliance

| Parameter | Type    | Description                               |
|-----------|---------|-------------------------------------------|
| _from     | address | Address of the source of the transfer.    |
| _to       | address | Address of the recipient of the transfer. |
| _amount   | uint256 | Quantity of tokens transferred.           |

> :::info
> The function parameter `amount` can be replaced by a `tokenId` in case of NFTs.

```solidity
function transferred(
    address _from,
    address _to,
    uint256 _amount
);
```

### created

Function called whenever tokens are given to an address. This function can update state variables in the modules bound
to the compliance these state variables being used by the module checks to decide if a transfer is compliant or not
depending on the values stored in these state variables and on the parameters of the modules. In practice, it calls
`moduleMintAction()` on each module bound to the compliance contract.

> :::info
> This function can be called ONLY by the token contract bound to the compliance

> :::caution
> Modules handle transfers differently than mints and burn, make sure to call `created()` when you mint tokens and not
> the `transferred()` method.

| Parameter | Type    | Description                                     |
|-----------|---------|-------------------------------------------------|
| _to       | address | Address of the recipient of the created tokens. |
| _amount   | uint256 | Quantity of tokens created.                     |

> :::info
> The function parameter `amount` can be replaced by a `tokenId` in case of NFTs.

```solidity
function created(
    address _to,
    uint256 _amount
);
```

### destroyed

Function called whenever tokens are removed from an address. This function can update state variables in the modules
bound to the compliance these state variables being used by the module checks to decide if a transfer is compliant or
not depending on the values stored in these state variables and on the parameters of the modules. In practice, it calls
`moduleBurnAction()` on each module bound to the compliance contract.

> :::info
> This function can be called ONLY by the token contract bound to the compliance

> :::caution
> Modules handle transfers differently than mints and burn, make sure to call `destroyed()` when you burn tokens and not
> the `transferred()` method.

| Parameter | Type    | Description                                   |
|-----------|---------|-----------------------------------------------|
| _from     | address | Address of the owner of the destroyed tokens. |
| _amount   | uint256 | Quantity of tokens destroyed.                 |

> :::info
> The function parameter `amount` can be replaced by a `tokenId` in case of NFTs.

```solidity
function destroyed(
    address _from,
    uint256 _amount
);
```

## Configuring modules

Modules are contracts that implements a set of rules used to validate transfer compliance. Modules contains storage and
expose their own configuration methods, while also implementing standard methods. Add and remove active modules via the
`addModule` and `removeModule` methods on the modular Compliance contract. Configuring modules shoukd be done via the
compliance `callModuleFunction` method.

### addModule

Used to add a module to the compliance contract. Only the owner of the compliance can add a module.

Emits a `ModuleAdded(address indexed module)` event.

```solidity
addModule(address _module);
```

### removeModule

Used to remove a module from the compliance contract. Only the owner of the compliance can add a module.

Emits a `ModuleRemoved(address indexed module)` event.

```solidity
removeModule(address _module);
```

### callModuleFunction

Used to call a configuration method on a compliance module contract. Direct interaction with module is discouraged, and
should instead go through the Compliance Contract that will proxy the call to the bound module.

Emits a `ModuleInteraction(address indexed module, bytes4 functionSelector)` event.

```solidity
function callModuleFunction(bytes calldata callData, address _module);
```

> :::tip
> Use helper librairies to encode the `callData` parameters.

<Tabs>
<TabItem value="js" label="JavaScript">

```javascript
const complianceContract = new ethers.Contract(complianceAddress, ['function callModuleFunction(bytes calldata callData, address module)'], signer);
const allowedCountryModule = new ethers.Interface(['function addAllowedCountry(uint256 country)']);
const callData = allowedCountryModule.encodeFunctionData('addAllowedCountry', [42]);
await complianceContract.callModuleFunction(callData, moduleAddress);
```

</TabItem>
</Tabs>

## Additional methods

### bindToken

Used to bind a token for the compliance contract. Only bound tokens can inform the compliance and its module of asset
transfers.

Only the owner of the compliance can bind a token to an already configured compliance contract.

It emits a `TokenBound(address token)` event

```solidity
function bindToken(address _token);
```

### bindToken

Used to unbind a token for the compliance contract.

Only the owner of the compliance can unbind a token, or the token itself.

It emits a `TokenUnound(address token)` event

> :::caution
> Because compliance modules build up their state when transfers occur, it is important to bind modules before starting
> minting. Some modules may not be impacted (eg. country restrictions), but modules that restrict transfers based on
> time or balances won't see transfers that happened before they were bound.

```solidity
function unbindToken(address _token);
```
