# Multiprocessing Crypto Recorder (order book snapshot)
As of September 7th, 2018.

## 1. Purpose
The purpose of this application is to record full limit order book and trade tick data from **GDAX** and **Bitfinex** 
into an Arctic Tickstore database (i.e., MongoDB) to perform reinforcement learning research.

There are multiple branches of this project, each with a different implementation pattern for persisting data:
 - **FULL** branch is intended to be the foundation of a fully automated trading system (i.e., process-thread /
 consumer-producer design patterns are ideal for a trading system that requires parallel processing) and 
 persists streaming tick data into an **Arctic tick store**
 - **LIGHT WEIGHT** branch is intended to record streaming data more efficiently than the __full__ branch (i.e., 
 all websocket connections are made from a single process __and__ the limit order book is not maintained) and
 persists streaming tick data into an **Arctic tick store**
 - **ORDER BOOK SNAPSHOT** branch has the same design pattern as the __full__ branch, but instead of recording streaming 
 ticks, snapshots of the limit order book are taken every **N** seconds and persisted 
 into an **Arctic tick store**

## 2. Scope
Application is intended to be used to record limit order book data for 
reinforcement learning modeling. Currently, there is no functionality 
developed to place order and actually trade.

## 3. Dependencies
- abc
- [arctic](https://github.com/manahl/arctic)
- asyncio
- datetime
- json
- multiprocessing
- os
- pandas
- requests
- SortedDict
- threading
- time
- websockets

## 4. Design Pattern
### 4.1 Architecture
- Each crypto pair (e.g., Bitcoin-USD) run on its own `Process`
  - Each exchange data feed is processed in its own `Thread` within the 
  parent crypto pair `Process`
  - A timer for periodic polling (or order book snapshots--see `feature-integration` branch) runs on a separate thread

![Design Pattern](assets/design-pattern.png)

### 4.2 Arctic Schema
**Arctic tick store** is the database implementation for this project for the 
following reasons:
 - Open sourced reliability
 - Superior performance metrics (e.g., 10x data compression)

The **arctic tick store** data model is essentially a `list` of `dict`s, where 
each `dict` is an incoming order book snapshot
- Each `list` consists of `./common_components/configs.CHUNK_SIZE` ticks (e.g., 100,000)
- All currency pairs are stored in the **same** MongoDB collection

### 4.3 Limit Order Book
**SortedDict** python class is used for the limit order book
for the following reasons:
- Sorted Price **Insertions** within the limit order book
 can be performed with **O(log n)**
- Price **Deletions** within the limit order book can be performed with **O(1)**
- **Getting / setting** values are performed with **O(1)**
- **SortedDict** interface is intuitive, thus making implementation easier

## 5. Appendix
### 5.1 Assumptions
- You know how to start up a Mongo database and have mongoDb installed already
- You know how to use Git tools
- You know how to use a CLI

### 5.2 To-dos:
1. Create a back testing simulation environment using GYM
2. Integrate Tensorflow into trading model
