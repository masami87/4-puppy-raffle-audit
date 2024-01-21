### [M-#] Looping through players array to check for duplicates in `PuppyRaffle:enterRaffle` is a potential denail of service (DoS) attack, incrementing gas costs for future entrants

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
