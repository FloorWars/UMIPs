
## Headers

| UMIP                |                                                               |
| ------------------- | ------------------------------------------------------------- |
| UMIP Title          | Add ETHfw10PUNK, fw10PUNKETH, USDfw10PUNK, and fw10PUNKUSD as supported price identifiers      |
| Authors             | tomosaigon                                                    |
| Status              | Draft                                                         |
| Created             | October 7, 2021                                               |
| Discourse Link      | [Link](https://discourse.umaproject.org/t/add-ethfw10punk-fw10punketh-usdfw10punk-and-fw10punkusd-as-price-identifiers/1352)            |

# Summary 

The DVM should support price requests for `ETHfw10PUNK`, `fw10PUNKETH`, `USDfw10PUNK`, and `fw10PUNKUSD`.  `fw10PUNKETH` reflects the FloorWars Decile Floor price of CryptoPunks in ETH. `ETHfw10PUNK` is the inverse of `fw10PUNKETH`. `USDfw10PUNK` is `ETHfw10PUNK` converted from ETH to USD with the DVM-supported price ID from UMIP-6. `fw10PUNKUSD` is the inverse of `USDfw10PUNK`.


# Motivation

This price identifier would allow for the creation of a synthetic covered (covered long ETH, short index) call option using UMA's Long Short Pair financial contract, e.g. `ETHfw10PUNKc200-1221`. The option would allow taking a position on the relative price movement of ETH versus an index price representative of the minimum value of a collection of NFTs, in this case CryptoPunks. The price representing the CryptoPunks Decile Floor is calculated by taking the median offered price of the lowest 10% of listing prices (in ETH) from the primary market. This value represents a more stable "floor" price for the collection and a price at which one can reasonably expect to buy into the collection.


# Data Specifications

-----------------------------------------
- Price identifier name: fw10PUNKETH
- Base Currency: fw10PUNK
- Quote Currency: ETH
- Markets & Pairs: derived from Ethereum contract 0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb
- Example data providers: Ethereum blockchain via Infura
- Cost to use: Free up to 100,000 requests/day - https://infura.io/pricing ; Alchemy
- Real-time data update frequency: Not applicable for non-liquidable contracts like LSP
- Historical data update frequency: Updated every Ethereum block (e.g. ~15 seconds)

-----------------------------------------
- Price identifier name: ETHfw10PUNK
- Base Currency: ETH
- Quote Currency: fw10PUNK
- Markets & Pairs: derived from Ethereum contract 0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb
- Example data providers: Ethereum blockchain via Infura
- Cost to use: Free up to 100,000 requests/day - https://infura.io/pricing
- Real-time data update frequency: Not applicable for non-liquidable contracts like LSP
- Historical data update frequency: Updated every Ethereum block (e.g. ~15 seconds)
-----------------------------------------
- Price identifier name: fw10PUNKUSD
- Base Currency: fw10PUNK
- Quote Currency: USD
- Markets & Pairs: derived from ETHfw10PUNK from this UMIP and ETHUSD from UMIP-6
- Example data providers: Ethereum blockchain via Infura; Alchemy; Kraken
- Cost to use: Free up to 100,000 requests/day - https://infura.io/pricing
- Real-time data update frequency: Not applicable for non-liquidable contracts like LSP
- Historical data update frequency: Updated every Ethereum block (e.g. ~15 seconds)

-----------------------------------------
- Price identifier name: USDfw10PUNK
- Base Currency: USD
- Quote Currency: fw10PUNK
- Markets & Pairs: derived from ETHfw10PUNK from this UMIP and ETHUSD from UMIP-6
- Example data providers: Ethereum blockchain via Infura; Alchemy; Kraken
- Cost to use: Free up to 100,000 requests/day - https://infura.io/pricing
- Real-time data update frequency: Not applicable for non-liquidable contracts like LSP
- Historical data update frequency: Updated every Ethereum block (e.g. ~15 seconds)


# Price Feed Implementation

Existing price feeds include:
- [CryptoWatch](https://github.com/UMAprotocol/protocol/blob/master/packages/financial-templates-lib/src/price-feed/CryptoWatchPriceFeed.js)

A sample dapp to calculate Decile FloorWars CryptoPunks price from CryptoPunksMarket will be included in the FloorWars GitHub repository.


# Rationale

The commonly used "floor" price of an NFT collection is easy to manipulate by the owner of the current "floor" or lowest offer so should not be used directly.

At any given time, only a portion of all available NFTs are offered for sale. Ones which were recently traded may represent a price which one could have bought into the collection at but does not represent a price the public can buy in now. Trades will include wash trades where the published trading price is much higher or much lower than the public could or would have bought for. Trades should not be considered in an index price representing what the public can currently buy.

Public bids in an order book also represent a view on liquidity. However, there are very few bidders on CryptoPunks and the bids are far apart from the asks. Decile Floor CrytoPunks are offered but seldom have standing bids on.

The offers tend to cluster more at the bottom price range and have widely ranging prices in the higher deciles. The "bottom" or lowest decile has stable clustering with high offer liquidity around the median which is why that was chosen. The prices around the median tend to be close or the same thus limiting the impact of delisting the current median or relisting it at a different price whether within our above the decile.

The mean of the bottom decile is another useful metric but the median tends to be rather stable over time compared to the mean.

CrytoPunks offers are priced in ETH. A USD price for them must be converted from ETH to USD. We use the already defined and used UMIP-6 standard for that conversion when necessary.

It's possible to sell CryptoPunks outside of the primary contract. However, the contract and what's displayed on the Larva Labs website is the primary market and considered canonical for floor prices.

It's possible to trade CryptoPunks privately but these sales should not be considered publicly available in the same way as the contract and may include wash trades.

It's possible to use OpenSea for market data but unnecessary given the CryptoPunksMarket is the primary market.

# Implementation

Both bids and asks (listings) for all CryptoPunks is available in the CryptoPunksMarket contract on Ethereum mainnet and can be queried by connecting to the blockchain via any RPC such as Infura or Alchemy (which provides free archival data requests).

The CryptoPunksMarket contract address is [0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb](https://etherscan.io/address/0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb)

Listing prices can be found by calling the function punksOfferedForSale for each `punkIndex` value from `0` to `9999`. If the returned value has `isForSale` set `true`, and `onlySellTo` set to `0x0` then consider `minValue` as a selling price. Count and sort all offers and take 10% of the number of offers, rounded down to a whole number, as the decile count. From the decile count of the lowest sorted offers, find the median value of the lowest decile. This is the decile floor priced in ETH to be returned. Historical values can be returned by querying archival data from a specific block height.

This is `fw10PUNKETH` and is the exact amount from the contract for the median decile floor CryptoPunk offer. Calculate the inverse for `ETHfw10PUNK` and round to the nearest 6 decimal places.

An ETHUSD price should be calculated according to UMIP-6. `fw10PUNKUSD` at a given timestamp is `fw10PUNKETH` multiplied by `ETHUSD` at that time. `USDfw10PUNK` is the inverse of `fw10PUNKUSD` rounded to the nearest 6 decimal places.

To calculate historical prices at a specific timestamp, first convert the timestamp to the nearest earlier Ethereum block height. Query the CryptoPunksMarket contract at that block height using an archival node RPC such as Alchemy (free). Query ETHUSD according to UMIP-6 with the timestamp of the block used and not the timestamp of the original request because we will return the relevant CryptoPunksMarket offers in ETH converted to USD at the ETHUSD value applicable at that block.

# Security Considerations

- How could pricing data manipulation occur?

    Manipulating a floor price for an NFT collection can be as trivial as buying the cheapest listed item then relisting at a much lower or higher price or delisting it. By considering the lower decile of the collection instead of a single item, the required funds to manipulate price this way is much more than can be gained by manipulating the above-defined index can earn.

- How could this price ID be exploited?

    One could purchase minted tokens at a fair price and hope to manipulate the index price at expiry but doing so requires funds and flash loans can't be used to manipulate the price. The USD-denominated prices are protected from on-chain flash ETHUSD price manipulation because they come from centralized exchanges (UMIP-6).

- Do the instructions for determining the price provide people with enough certainty?

    The values in the CryptoPunksMarket contract are canonical at a given block height. There are many ways to query the data and an example implementation will be provided. 

- What are current or future concern possibilities with the way the price identifier is defined?

    The market may shift away from CryptoPunksMarket to alternate or centralized NFT exchanges.

- Are there any concerns around if the price identifier implementation is deterministic?
    
    Fetching data from the contract may take longer than Ethereum block time. Block height should be used then.
