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

To calculate the new prices of each position in a prediction market after a trade, you can use the **Logarithmic Market Scoring Rule (LMSR)**, a popular automated market-making algorithm proposed by Robin Hanson. This model adjusts prices based on the total number of shares (or the total investment) in each outcome.

### **Key Components:**

- **Cost Function \( C(q) \):**

```math
  C(q) = b \cdot \ln\left(\sum_{i} e^{\frac{q_i}{b}}\right)
```

  - \( q_i \): Total number of shares in outcome \( i \).
  - \( b \): Liquidity parameter (controls the market's sensitivity to trades).

- **Price Function \( p_i \):**

```math
  p_i = \frac{e^{\frac{q_i}{b}}}{\sum_{j} e^{\frac{q_j}{b}}}
```

  - \( p_i \): Price (probability) of outcome \( i \).
  - The denominator sums over all possible outcomes \( j \).

### **Calculating New Prices:**

1. **Initial State:**

   - Assume initial quantities \( q_{\text{yes}} = q_{\text{no}} = 0 \).
   - Initial prices are:
```math
     p_{\text{yes}} = p_{\text{no}} = \frac{1}{2}
```

2. **After Purchasing Shares:**

   - Suppose a trader buys \( \Delta q_{\text{yes}} \) shares of "Yes".
   - Update the total quantity:
```math
     q_{\text{yes}}' = q_{\text{yes}} + \Delta q_{\text{yes}}
```
   - New prices are:
```math
     p_{\text{yes}}' = \frac{e^{\frac{q_{\text{yes}}'}{b}}}{e^{\frac{q_{\text{yes}}'}{b}} + e^{\frac{q_{\text{no}}}{b}}}
```
```math
     p_{\text{no}}' = \frac{e^{\frac{q_{\text{no}}}{b}}}{e^{\frac{q_{\text{yes}}'}{b}} + e^{\frac{q_{\text{no}}}{b}}}
```
   - Since \( q_{\text{no}} \) remains the same, only \( q_{\text{yes}} \) changes.

### **Example Calculation:**

- **Given:**
  - Trader buys \( \Delta q_{\text{yes}} = 100 \) shares.
  - Choose \( b = 50 \) (you can adjust \( b \) based on desired liquidity).
- **Compute New Prices:**
  - Update quantity:
```math
    q_{\text{yes}}' = 0 + 100 = 100
```
  - Calculate exponentials:
```math
    e^{\frac{100}{50}} = e^{2} \approx 7.389
```
```math
    e^{\frac{0}{50}} = e^{0} = 1
```
  - New prices:
```math
    p_{\text{yes}}' = \frac{7.389}{7.389 + 1} \approx 0.881
```
```math
    p_{\text{no}}' = \frac{1}{7.389 + 1} \approx 0.119
```

### **General Formula:**

For any number of outcomes, after updating the quantities \( q_i \), the new price for each outcome \( i \) is:

```math
p_i = \frac{e^{\frac{q_i}{b}}}{\sum_{j} e^{\frac{q_j}{b}}}
```

### **Notes:**

- **Liquidity Parameter \( b \):** A larger \( b \) makes the market less sensitive to trades (prices change less with each trade).
- **Cost of Shares:** The cost to purchase \( \Delta q_i \) shares is the increase in the cost function:
```math
  \text{Cost} = C(q_i + \Delta q_i, q_{-i}) - C(q_i, q_{-i})
```
  - \( q_{-i} \): Quantities of all other outcomes.


More info on **Logarithmic Market Scoring Rule (LMSR)**
https://www.cultivatelabs.com/crowdsourced-forecasting-guide/how-does-logarithmic-market-scoring-rule-lmsr-work <br>
https://learn.microsoft.com/en-us/archive/msdn-magazine/2016/june/test-run-introduction-to-prediction-markets  <br>
https://docs.gnosis.io/conditionaltokens/docs/introduction3/  <br>
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
