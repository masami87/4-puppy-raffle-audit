### [M-1] Looping through players array to check for duplicates in `PuppyRaffle:enterRaffle` is a potential denail of service (DoS) attack, incrementing gas costs for future entrants

**Description:** Gas will grow.

**Imapact:** The gas costs for raggle entrants will greatly increase as more players enter the raffle. Discouraging layer users from entering, and causing a rush at the start of a raffle to be one of the first entrants in the queue.

An attacker might make the `PuppyRaffle:entrants` array so big, that no one else enters, guarenteeing themselves the win.

**Proof of Concepy:**

If we have 2 sets of 100 players enter, the gas costs will be such:

- 1st 100 players: ~6252039
- 2nd 100 players: ~18068126

<details>
<summary>PoC</summary>

```javascript
    function testDos() public {
        // 100 payers
        uint256 playersNum = 100;
        address[] memory players = new address[](playersNum);

        for (uint256 i = 0; i < playersNum; i++) {
            players[i] = address(i);
        }

        uint256 gasStart = gasleft();

        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);

        uint256 gasEnd = gasleft();

        uint256 gasUsedFirst = gasStart - gasEnd;

        console.log("gasUsedFirst", gasUsedFirst);

        // 2nd 100 payers

        address[] memory playersTwo = new address[](playersNum);

        for (uint256 i = 0; i < playersNum; i++) {
            playersTwo[i] = address(i + playersNum);
        }

        gasStart = gasleft();

        puppyRaffle.enterRaffle{value: entranceFee * playersNum}(playersTwo);

        gasEnd = gasleft();

        uint256 gasUsedSecond = gasStart - gasEnd;

        console.log("gasUsedSecond", gasUsedSecond);

        assert(gasUsedSecond > gasUsedFirst);
    }
```

</details>

**Recommended Mitigation:** There are a few recomendations.

1. Consider allowing duplicates.

2. Consider using a mapping to check for duplicates.

### [I-1]: Solidity pragma should be specific, not wide

Consider using a specific version of Solidity in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.0;`, use `pragma solidity 0.8.0;`

### [I-2] Magic Numbers

**Description:** All number literals should be replaced with constants. This makes the code more readable and easier to maintain. Numbers without context are called "magic numbers".

**Recommended Mitigation:** Replace all magic numbers with constants.

```diff
+       uint256 public constant PRIZE_POOL_PERCENTAGE = 80;
+       uint256 public constant FEE_PERCENTAGE = 20;
+       uint256 public constant TOTAL_PERCENTAGE = 100;
.
.
.
-        uint256 prizePool = (totalAmountCollected * 80) / 100;
-        uint256 fee = (totalAmountCollected * 20) / 100;
         uint256 prizePool = (totalAmountCollected * PRIZE_POOL_PERCENTAGE) / TOTAL_PERCENTAGE;
         uint256 fee = (totalAmountCollected * FEE_PERCENTAGE) / TOTAL_PERCENTAGE;
```

### [I-4] Zero address validation

**Description:** The `PuppyRaffle` contract does not validate that the `feeAddress` is not the zero address. This means that the `feeAddress` could be set to the zero address, and fees would be lost.

```
PuppyRaffle.constructor(uint256,address,uint256)._feeAddress (src/PuppyRaffle.sol#57) lacks a zero-check on :
                - feeAddress = _feeAddress (src/PuppyRaffle.sol#59)
PuppyRaffle.changeFeeAddress(address).newFeeAddress (src/PuppyRaffle.sol#165) lacks a zero-check on :
                - feeAddress = newFeeAddress (src/PuppyRaffle.sol#166)
```

**Recommended Mitigation:** Add a zero address check whenever the `feeAddress` is updated.
