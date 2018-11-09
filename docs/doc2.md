---
id: serverapi1
title: Problem Statement
---
Build a trader desktop *prototype* for fund managers to buy and sell stocks. The application should also show real-time graphical representation of status of trades

## Context and Business Domain 

Fund managers request the buying or selling of a stock by placing an *Order* into the system. The order is assigned to a trader who is responsible for placing the trade with a broker. If the order is too big and has the potential of inefficient execution (e.g. moving the market in an unfavorable direction), the trader may decide to split the order into multiple *Placements* and place each one with a different broker.

Each placement may be fulfilled in the market in one or more chunks, known as *Executions*. For example, a purchase of 1,000,000 shares may be fulfilled using ten executions of 100,000 shares.

Some more terminology:
- The total quantity that is placed with one or more brokers is also called the quantity *committed*.
- The total quantity that is executed is also called the quantity *done*. 

The system receives market prices for all equities of interest. This information is vital for traders to trade efficiently.

Finally, the system is able to send alerts to the fund manager, e.g. when a new order has been entered for a trader.

## Use cases
For the prototype, following (highly simplified) set of use-cases need to be fulfilled.

As a Fund Manager:
* I should be able to place trades from a list of stocks - 

     Maintain a hard code list of stocks (Symbol, Direction, Quantity) in your app.
     You can either create UI to enter specific share and quantity or randomize to submit orders.
     
* I should be able to see real-time status of all my orders

    An order consists of a stock symbol, a buy/sell flag and an integer quantity. 
    The status of an order is defined as the total quantity of the order, the quantity placed with brokers and the quantity executed.
* I should be able to filter the status of orders by Open, Done/Executed and All 
* At any give time for a stock, I should be able to see total quantity, commited and executed quantity
* I should be able to clear all the trades
* I should be able to place a random set of orders with buy/sell side

## API

Download/Clone the `trader-desktop-server` application from (https://github.com/cssmiles/hackathon) and run 
```bash
$ yarn install 
$ yarn build 
$ yarn start
# if you prefer you can use npm instead of yarn
```
To verify that the application is working correctly, point your browser to http://localhost:8080/orders - you should see a response with a list of orders in JSON format. 
Since **the persistence layer is in memory**, the list will be empty.
### Load all orders 

```bash
GET https://localhost:8080/orders
```
Response
If successful, this method returns a response body with the following structure:
```bash
[
    {
        id: '1fda514f-1f21-4dff-966f-5cc8fc296ad8',
        side: 'BUY',
        symbol: 'AAPL',
        quantity: 10000
    },
    {
        id: '1def99c1-5ec0-e4r5-a136-918d21afc345',
        side: 'SELL',
        symbol: 'MSFT',
        quantity: 10000
    }
]
```

### Send an order
```bash
POST https://localhost:8080/orders
```

Request body
```bash
{
    "id": "1fda514f-1f21-4dff-966f-5cc8fc296ad8",
    "side": "BUY",
    "symbol": "AAPL",
    "quantity": 100000
}
```
| Name               | Description                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id` | Unique Id                                                  |
| `side`      | Valid values 'BUY' or 'SELL'  |
| `symbol`      | Stock symbol  |
| `quantity`      | Quantity to buy/sell  |

### Listen status updates 
Any additions/changes to orders on the server will be pushed down the client using WebSockets. 
```bash
#sample code
const ws = new WebSocket("ws://localhost:8080");
ws.onmessage = (event : any) => {
    const { type, payload } = JSON.parse(event.data);
    # see table below for response structure
}
```
```bash
{
    "type":"orderChanged",
    "payload":{
        "committed": "305"
        "executed": "153"
        "id": "1fda514f-1f21-4dff-966f-5cc8fc296ad8"
        "quantity": 10000
        "side": "BUY"
        "symbol": "AAPL"
        }
    }
}
```
| Name               | Description                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type` | Possible values are <ul><li>**orderChanged** - When an order changes</li><li>**allOrdersDeleted** - When all orders deleted successfully</li><li>**orderCreated** - When a new order is created</li></ul>                                                |
| `committed`      | Quantity that is placed with one or more brokers |
| `executed`      | Successfully executed quantity  |
| `id` | Unique Id                                                  |
| `side`      | Valid values 'BUY' or 'SELL'  |
| `symbol`      | Stock symbol  |
| `quantity`      | Quantity bought/sold  |
| `nextPlacementTime`      | Next trade placement time in milliseconds, `null` if all trades placed |
| `nextExecutionTime`      | Next trade execution time in milliseconds, `null` if all trades executed |


### To reset orders 
Clears orders stored in the server
```bash
POST https://localhost:8080/reset
```
```bash
{
    "type": "allOrdersDeleted"
}
```

