# Ampleforth Explained

## Contracts


Ampleforth on Mainnet

| Name               |Address                          |
|----------------|-------------------------------|
| ERC-20 Token   | 0xD46bA6D942050d489DBd938a2C909A5d5039A161 |
| Supply Policy  | 0x1B228a749077b8e307C5856cE62Ef35d96Dca2ea |
| Orchestrator   | 0x6fb00a180781e75f87e2b690af0196baa77c7e7c |
| Market Oracle  | 0x99c9775e076fdf99388c029550155032ba2d8914 |
| CPI Oracle     | 0x2A18bfb505b49AED12F19F271cC1183F98ff4f71 |
| WAMPL          | 0xEDB171C18cE90B633DB442f2A6F72874093b49Ef | 


#### ERC-20 Contract
The AMPL Contract implements the basic ERC20 interface and has one additional function. If `supplyDelta` is positive, new tokens are added to existing holders. If negative, tokens are removed. Instead of triggering transactions, Ampleforth balances are internally represented by a hidden internal denomination. Fore more information, check the Whitepaper. Rebase function: `rebase(uint256 epoch, int256 supplyDelta)`

#### Supply Policy
The Supply Policy Contract has a single external function, also called `rebase()`, this is not to be confused with the rebase method in the Ampleforth ERC-20 Contract. This `rebase()` method is publicly callable by anyone, but will only execute at most once every 24 hours. Opening this method up, helps to remove us as a necessary central party in the system’s execution. If we fail to call `rebase()` for any reason, others are free to make that call in our place. The rebase() method first queries the Market Oracle to get the current price. If the price is within `priceThreshold` of the target price, no supply policy change is applied. Otherwise, the absolute `supplyDelta` is equal to `(price-target)*totalSupply/target`. For example, if Amples are trading for $1.15, the absolute `totalSupply` increase will be 15%. Next, it applies a “rebase reaction lag” to dampen the supply change. At launch, the reaction lag will be 30 days. Finally, the Ampleforth ERC-20 token is instructed to adjust its supply by the dampened value. Continuing with the example above, the dampened increase would be (15% / 30 days) = 0.5% per day. Due to the unpredictability of when transactions get mined into a block, and because at least 24 hours must pass before a rebase executes, there will always be slightly more than 24 hours between rebases. This means that, even though our rebase time is 24hrs, the rebase call will “drift” slightly over time. Based on our measurements on Ethereum’s Rinkeby testnet, we expect this drift to be about 1 hour per year. So if rebase calls execute at 0:00 UTC time on Jan 1st, they would execute 1:00 UTC a year later. A public countdown timer to the next allowable rebase operation each day is displayed on the Ampleforth Protocol dashboard.

#### Market Oracle
The Market Oracle Contract provides data from the outside world to be used by the Supply Policy Contract. Specifically, it returns the 24-hour volume-weighted Ample Price from exchanges. At launch, oracle will have a trusted whitelist of sources and the price is calculated as the median of the sources.
1. Only whitelisted addresses can provide market data.
2. A market report must exist on-chain publicly for at least 1 hour before it can be used by the supply policy.
3. A market report will expire on-chain if a new report is not provided before 6 hours elapse.


## Ampleforth Fork Deployment

| Component                 | Address                                           |
|---------------------------|---------------------------------------------------|
| Deployer                  | 0x08150bAB13edC834FD5b436C9416dC849f410C66       |
| UFragments deployed to    | 0x0Df6D221DbB5cabAc86777E9FF6B455a1b2cFAD4       |
| Implementation            | 0x86B1c1eD8E4161eAbE7c78D182ef16b9b49c9146       |
| Market oracle to          | 0x15a98688AB4274C578d19874D5Cf3B2991d08A31       |
| CPI oracle to             | 0x6b06Eb8f79457bC036d1a50e4bDcbeBD7AdD50c4       |
| UFragmentsPolicy to       | 0x75296Fae0b5e3dF0bBd8D29732555E883dFF84cA       |
| Implementation            | 0xe44b1ab41c56140B195E66DC9F0F2a264885F015       |
| Orchestrator deployed to  | 0xAF6e88fF6FFF8E18822ecA4422Cf0013CA241940       |
| Admin/Provider wallet     | 0x054cf8e459bE39D29436c77B094DD62b73968DA4       |
| Transferring ownership to | 0x054cf8e459bE39D29436c77B094DD62b73968DA4       |




### How Ampleforth works

- **Equilibrium 1:** Alice has 1 Ample worth $1.
- **Demand Increases:** Alice has 1 Ample worth $2.
- **Equilibrium 2:** Alice has 2 Amples each worth $1.

Whether Alice holds 1 Ample worth $2 or 2 Amples each worth $1, makes no difference in terms of net balance. The difference is the Ampleforth protocol directly propagates price information to each token owner through the count in their token balances. By expanding to and contracting from coin holders directly, a given user’s percent ownership remains fixed unless the user chooses to sell or buy more.

