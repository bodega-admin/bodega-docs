# Prediction Market Smart Contract

## Overview

This project implements a prediction market using smart contracts. Users can place bets on real-world events, with positions that can be continuously bought and sold until the day of payout. On payout day, the winning side shares the profit pro-rata.

## High-Level Example

Consider a prediction market for the 2024 US election:

1. Initial state:
   - Trump: 50% ($0)
   - Kamala: 50% ($0)

2. Bob buys $100 of Trump:
   - Trump: 99% ($100)
   - Kamala: 1% ($0)

3. Alice buys $50 of Kamala:
   - Trump: 66% ($100)
   - Kamala: 33% ($50)

4. John buys $100 of Kamala:
   - Trump: 40% ($100)
   - Kamala: 60% ($150)

### Selling Positions

To calculate the selling price of a position:
```
Original investment * (Current probability / Buy-in probability)
```

Example:
- Alice selling Kamala shares: $50 * (60% / 33%) ≈ $90.91
- Bob selling Trump shares: $100 * (40% / 99%) ≈ $40.4

### Payout Calculation

If Kamala wins the 2024 election:

```
Original investment + ((original investment / total winning pool) * losing amount)
```

- Alice's winnings: $50 + ((50/150) * 100) = $83.33
- Bob's winnings: $100 + ((100/150) * 100) = $166.66

Note: Winnings are rounded down to the nearest whole number. In this case:
- Alice receives $83
- Bob receives $166
- The remaining $0.99 goes to the house

## Smart Contract Design

The system consists of three main scripts:

1. Minting Script
2. Predictions Script
3. Positions Script

### Key Features:

All positions are in USDM (USD Mockup). A single UTXO contains the datum of each position and all USDM. A batcher processes transactions and updates the datum in real-time. Assets are minted 1:1 to the amount of USDM deposited. Minted assets are sent to the positions script. Position UTXOs contain data on the wallet that minted the assets and the proportion after entering the position. The token name represents their position. True/False of a statement. 


### Selling a Position:

1. The positions script checks if the owner signed the transaction
2. The predictions validator reads the datum of the position UTXO
3. It compares the proportion at entry with the current proportion
4. Assets in the positions UTXOs equal the initial deposit
5. Assets must be burnt to consume from the predictions script

### Getting Rewards a Position:

1. Read from an oracle UTxO that determines the outcome of the trade. The oracle will simply have a true statement corresponding to the UTxO in the predictions Script. In the example of the 2024 election, the oracle will publish a true or false based on a statement. For example, Trump wins the 2024 election and in the datum field of the oracle it will have the value true. The UTxO in the predictions script will have the following datum. 
```
{
true_position_amount: Int, 
false_poistion_amount: Int
...
}
```
The logic in the predictions validator will simply check the field in the oracle UTxO and determine the winners. 
2. The rewards can only be distributed after the statement has been published by the oracle
3. To get their rewards they also need to pass in the positions UTxO. The assets in the positions UTxO must be burnt. Validate the rewards they are getting is valid by looking at the datum in the predictions utxo and the assets burnt in the transaction. 


## Implementation Notes

- Use USDM for all bets and transactions
- Implement a batcher for efficient transaction processing
- Implement accurate rounding mechanisms for payouts, all remainder decimals will be ours to keep
