---
published: true
title: My divorce contract is smart
layout: post
---
Few months into solidity, I'd walk you through my divorce smart contract. The contract simply needs both parties to agree on how the money in the contract would be shared when both parties have signed on the divorce.

Firstly, to initialize the contract we'd need two addresses, the husband's and wife's address and the percentage the wife would take whey they agree to split.

```solidity

constructor (
    address payable _husbandAddress, 
    address payable _wifeAddress, 
    uint _wifeCut
) payable 		
{
    require((_husbandAddress != address(0)) && (_wifeAddress != address(0)));
    require(_wifeCut < 100);

    emit InitiatedContract(_husbandAddress, _wifeAddress, _wifeCut);

    husbandAddress = _husbandAddress;
    wifeAddress = _wifeAddress;
    wifeCut = _wifeCut;
    balance = msg.value;
    state = State.Initiated;
}

```

Spliting the money, when they decide to divorce based in the agreed percentages. 

Firstly, we'd need to divide the percentages using `_getPercentages`.

```solidity
function _getPercentages() private view returns (uint, uint) {
    uint si = address(this).balance.mul(wifeCut).div(100);
    return (si, address(this).balance - si);
}
```

...and then sending their value if percentages respectively. Note this funtion would throw an error if the divorce contract has be settled. Hence the modifier `onlyState(State.Initiated)`

```solidity
function splitBalance() external onlyOwner onlyState(State.Initiated) {
    emit Split(msg.sender);
    (uint wifeAmount, uint husbandAmount) = _getPercentages();

    wifeAddress.transfer(wifeAmount);
    husbandAddress.transfer(husbandAmount);
    state = State.Inactive;
    emit ContractFinalized();
}
```

[full code here](https://github.com/keosariel/lil-web3/blob/master/contracts/LilDivorce.sol)
