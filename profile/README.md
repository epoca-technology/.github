# EPOCA

Epoca implements a modern variation of the "Value Averaging Strategy" that operates in the Bitcoin Futures Market. It makes use of the Hedge-Mode feature and is capable of managing a long and a short position simultaneously. 

The requirements for running an instance of Epoca are:

- NodeJS v16.14.0^
- Docker v24.0.6^
- 2 CPUs
- 4 GB Memory
- 50 GB Disk


![Dashboard](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2Fintro.png?alt=media&token=30559c14-f241-4ffe-8fb7-c9f62fd2590c)


Epoca manages the trading budget in an increase/decrease manner. When the price moves against a position side, it increases it. In contrast, when the price moves strongly in favor of a position, it reduces it. Additionally, the trading budget is split into 10 - 20 pieces and is used to constantly improve the entry price, regardless of the side.

To achieve this, Epoca makes use of a series of indicators built on publicly available data. The indicators used are:
- Price Window
- KeyZones (Support & Resistance Levels)
- Coins (Short-Term Price Window of the top cryptocurrencies)
- Liquidity in Bitcoin's Order Book
- Volume
- Reversal

#
## Price Window

The window module operates in a moving window of 128 15-minute-interval candlesticks (~32 hours) that is synced every ~3 seconds through Binance Spot's API.

To calculate the state of the window, a total of 8 splits are applied to the sequence of candlesticks and the state for each is derived based on the configuration.

The splits applied to the window are:

* 100%: 128 items (last ~32 hours)

* 75%: 96 items (last ~24 hours)

* 50%: 64 items (last ~16 hours)

* 25%: 32 items (last ~8 hours)

* 15%: 20 items (last ~5 hours)

* 10%: 13 items (last ~3.25 hours)

* 5%: 7 items (last ~1.75 hours)

* 2%: 3 items (last ~45 minutes)

The supported states are:

* 2: Increasing Strongly

* 1: Increasing

* 0: Sideways

* -1: Decreasing

* -2: Decreasing Strongly

![Price Window](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F02_window.png?alt=media&token=e66ed882-f738-4e87-8203-b5dcca6e9c7b)

![Price Window Dialog](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F03_window.png?alt=media&token=4772ce25-bbc5-48df-90ea-761d5b80a54c)

For more information regarding this module, please visit the **Core API's source** code or the following notebook: [Bitcoin Price State](https://www.kaggle.com/code/jesusgraterol/bitcoin-price-state).




#
## KeyZones

This module aims to identify support and resistance levels and manage the life cycle of the price contact events.

To achieve this, the Epoca performs a KeyZones Build every certain period of time and works hand in hand with the Window Module in order to generate events.

The "strength" of a KeyZone is measured by a score system that makes use of the liquidity within the KeyZone's range in real-time and the trading volume that took place when the KeyZone was first discovered.

When the price increases or decreases significantly and hits one of these levels, a KeyZone Event is created and remains active for a period of time. 
![KeyZones](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F04_keyzones.png?alt=media&token=d46f94d5-4749-4b77-a73d-5a7be2fd89f9) 

![KeyZones Events](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F05_keyzones.png?alt=media&token=3770e28b-51ba-44ea-b859-387ccd889935)

#### Example of a Support KeyZone Contact:

![Support KeyZone Contact](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F06_keyzones.png?alt=media&token=feee4659-64ea-4409-9d47-96c49f8eddb5)

#### Example of a Resistance KeyZone Contact:

![Resistance KeyZone Contact](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F07_keyzones.png?alt=media&token=ebcb2589-6438-4889-a1d3-c79fa5e76032)

For more information regarding this module, please visit the **Core API's source** code or the following notebook: [Bitcoin KeyZones (Support & Resistance Levels)](https://www.kaggle.com/code/jesusgraterol/bitcoin-keyzones-support-resistance-levels).






#
## Coins

This module aims to have a deep understanding of the short-term price direction for the top cryptocurrencies. 

Epoca establishes a WebSocket Connection to the "Mark Price" for all the installed cryptocurrencies and calculates their state on a real time basis for both rates, BTC and USDT. 

The state is calculated the same way as it is for the Window Module. 

![Coins/USDT](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F08_coins.png?alt=media&token=19cbcd48-1b84-45a9-91e9-d322258880e6)

![Coins/BTC](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F09_coins.png?alt=media&token=60ffa431-e708-46db-ae95-0abd70497092)

The state of the coins is analyzed as a group and individually:

![ETH/USDT](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F10_coins.png?alt=media&token=c53ed87a-ae24-4b8d-b426-c9ee961c1458)

![ARB/BTC](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F11_coins.png?alt=media&token=67802556-d943-414f-9307-97d30524594e)

Epoca automatically extracts all the supported coins from the exchange and scores them based on several factors. Such as: daily volume, maximum supported leverage, etc. It also supports the installation of any number of coins into the system.

![Install](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F12_coins.png?alt=media&token=a94230c2-b788-444e-be4a-a43c2baba160)





#
## Liquidity

This module aims to identify when an abnormal amount of buying/selling liquidity has been placed in Bitcoin's Order Book.

To achieve this, Epoca establishes a WebSocket Connection to the "Order Book" and processes the raw data on a real time basis. During this process, the system identifies "Liquidity Peaks" and groups them based on their intensity. 

Having identified all the peaks within the price range, the "Bid Liquidity Power" is calculated, whose only purpose is to indicate when there is an abnormal amount of buy or sell liquidity in the Order Book.


![Liquidity Summary](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F13_liquidity.png?alt=media&token=db42370b-ba2e-4b94-a83e-7ecc9e9eaff1)

![Liquidity Peaks](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F14_liquidity.png?alt=media&token=5f4f296e-0f2a-462a-952b-cc97fc90c781)

![Liquidity Asks](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F15_liquidity.png?alt=media&token=22e74d0c-a81a-4bcb-911f-ae321b6d1ec4)

![Liquidity bids](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F16_liquidity.png?alt=media&token=026e13ec-f5e2-406e-8e3a-cc165cd5a5c3)






#
## Volume

This module aims to have a deep understanding of the Bitcoin Spot Market's trading volume to identify times at which there is significant participation. 

To achieve this, the system establishes a real time connection with ~32 hours (same lookback as the window) worth of 1m candlesticks and calculates the volume state requirements (The amount of traded USDT required for it to mean that the volume is high).

The system evaluates the current volume against the requirements on a real time basis and broadcasts the state. The volume can have the following states:

* 0: Ultra Low

* 1: Low

* 2: Medium

* 3: High

Note that when a state greater than 0 is identified, it remains active for ~3 minutes. If the "high" volume is not sustained, it goes back to 0 after this period.

![Volume State](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F17_volume.png?alt=media&token=c01885b3-8f45-4983-9bbc-9ecfdd8694d7)






#
## Reversal

This module aims to identify when a KeyZone Contact Event (Support or Resistance) has the potential to cause a price reversal.

Whenever a KeyZone Event (support or resistance) comes into existence, the Reversal Module is activated and starts tracking the behavior of the market.

SUPPORT CONTACT: the Reversal Module analyzes Epoca's indicators to determine if there is enough buying power for the price to reverse (increase).

RESISTANCE CONTACT: the Reversal Module analyzes Epoca's indicators to determine if there is enough selling power for the price to reverse (decrease).

Each indicator has a weight and when the required points are reached, a reversal event is issued.

#### Example of a Support Contact followed by a Support Reversal Event:

![Support Contact](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F18_reversal.png?alt=media&token=aa521154-318d-4428-a919-3a28964469ea)

![Support Reversal Event](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F19_reversal.png?alt=media&token=7e3ab75e-5ce1-4cc8-b4ee-3d9caf939cfd)

#### Example of a Resistance Contact followed by a Resistance Reversal Event:

![Resistance Contact](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F20_reversal.png?alt=media&token=359fb9ab-07ab-4cd8-918f-ee295db40fd9)

![Resistance Reversal Event](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F21_reversal.png?alt=media&token=37601d24-9d15-4eeb-bf4f-0bca5347d7fb)




#
## Position Management


As described above, Epoca increases and decreases position sizes based on the state of the market. If the price moves against a position and the requirements are met, the position is increased. On the other hand, if the price moves in favor, the position is reduced.



#### Long Position Example

The image below shows a long position that was initially opened on the 14th of July 2023 and was closed (reduced to $0) on the 1st of October 2023. During this period, the position was increased ~10 times and experienced ~29 reductions. The reductions are represented by the black dots in the Gain% chart:

![Long Position](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F22_position.png?alt=media&token=200a20ce-839f-480b-8ceb-077354705f44)



#### Short Position Example

The image below shows a short position that was opened on the 14th of July 2023 and was closed on the 20th of July 2023. During this period, the position was increased ~3 times and experienced ~9 reductions:

![Short Position](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F23_position.png?alt=media&token=1ccec29c-9dbb-4e30-92fe-2e1a2a09153f)



#### Performance Analysis

Epoca provides an advanced interface to visualize the performance yielded for any period of time. It also allows you to keep a close eye on the fees charged by the exchange real time.

![Transactions](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F24_transactions.png?alt=media&token=eee6ea24-fb04-4b57-86a2-a7f7e563e103)



#
## Customizable

Even though the configuration that comes out-of-the-box has been tested for over a year, every module can be deeply fine-tuned to adapt to any trading strategy.



![Adjustments](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F25_adjustments.png?alt=media&token=2b6349ea-6a06-4e66-a037-6f1be1571767)






#
## Dataset Builder

In order to provide a high quality input to each of the modules, Epoca stores and permanently syncs the historical prices of Bitcoin in candlestick format (1 minute intervals). This allows the system to build and output datasets in any desired format (defaults to .csv) and time interval, so they can be used by any system external to Epoca (E.g. Jupyter Notebooks).

![Candlesticks Bundle](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F26_candlesticks.png?alt=media&token=3ac2356b-49a5-44cd-9486-3503ecaef7c4)




#
## Server Management

Epoca is a complete technological infrastructure that handles security, authentication, trading, server and database management so trading can be performed without any interruptions.

The hardware that runs the platform is constantly monitored and admins are notified if any irregularity or high temperature is detected.

![Server Monitoring](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F27_server.png?alt=media&token=fa15db97-2dff-4de2-81d8-f8c4feec3a4a)

The database is backed up on a daily basis, a copy is kept locally, and another one is uploaded to the cloud and can be easily restored in case of a system failure or natural disaster.

![Database](https://firebasestorage.googleapis.com/v0/b/projectplutus-prod.appspot.com/o/public%2Fwhitepaper%2F28_server.png?alt=media&token=abc356f9-a3c7-40e3-bcc7-3c3de7ce7d3e)
