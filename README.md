# blockpaca
Parallel execution system for Alpaca broker API 

## Installation 

```pip install blockpaca```

Pip installation requires python version >= 3.10 

Alternatively, you can clone the repository in a new or existing environment. 

In terminal: 
```git clone https://github.com/henrysalkever/blockpaca.git``` 

## Repo structure 
```
blockpaca/
├── LICENSE
├── pyproject.toml
├── README.md
└── src
    └── blockpaca
        ├── __init__.py
        ├── core.py
        ├── live_display.py
        └── tools.py
```

## Running blockpaca

```python
from blockpaca import run_trading

log_dir = "/Users/username/log_dir"
API_KEY = "custom_api_key" #API key obtained from your Alpaca account
SECRET_KEY = "custom_secret_key #secret key obtained from your Alpaca account
TICKERS = ["AAPL", "TSLA", "NVDA"] #tickers to receive data for, up to 30 max for free accounts

def custom_strategy(context):
    quotes, positions = context['get_current_data]()
    #some custom trading logic
    return order_dict 

if __name__ == "__main__": 


    run_trading(
        strategy_callable = custom_strategy,
        trade_log_name = "custom_strat_name",
        trade_log_path = log_dir,
        frequency = 10, 
        run_seconds = 60 
        eos_behavior = "hold", 
        warm_up_period = 10, 
        live_display = True, 
        display_refresh_rate = 0.05, 
        max_trade_updates = 20, 
    )
```

### Parameters 

`strategy_callable` : custom trading logic 

`trade_log_name` : name for trade log file (make this something identifiable to the strategy you are running) 

`trade_log_path` : path for trade logs 

`frequency` : frequency at which the trading logic executes. Accepts integers, floats as well as strings of types `"1m", "15m", "1h", "2h"` *

`run_seconds` : total seconds to run the trading loop for. Accepts integers, floats, as well as strings of types `"1m", "15m", "1h", "2h"` * and `"EOD"` which will input the seconds until the market closes for the day minus 30 seconds.

`eos_behavior` : dictates the behavior at the end of the trading session. Options are `"liquidate", "hold"` and `"custom"`. See the section on custom eos behavior for further details. 

`warm_up_period` : time to wait for quotes before starting the trading loop (5 seconds recommended)

`live_display` : Boolean value to toggle on/off for the rich terminal display of positions and trade status 

`display_refresh_rate` : time between each refresh of values in live display 

`max_trade_updates` : controls how many trade updastes are shown in the live display

*(numbers are adjustable, though will throw an error if you input a value for hours that will make your trading session terminate after 30 seconds before the market closes, for example `"9h"`) 


## Creating Custom Trading Logic 

Custom trading functions must be set up as follows: 

```python
def custom_function(context):
    #trading logic
    return orders
```

Your function must be set up to accept the context block as the only argument. The context block is a dictionary that is passed in that contains the methods for getting current quotes and positions, and submitting orders. The context block also contains the addresses for the shared memory, the result queue, and the command queue that are necessary for sharing information across python processes. 

```python
context = {
    "get_current_data": get_current_data, #callable
    "submit_order": submit order, #callable
    "shm_meta": shm_meta, #shared memory
    "result_queue": result_q, #result queue
    "command_queue": cmd_q, #command queue
}
```

### Receiving Current Quotes and Positions 

Within your trading function, run the following to get current quotes and positions: 

```python
quotes, positions = context['get_current_data']()
```

The schema for the quote data you recieve will be 
```python
{
   "AAPL": {"bid": 150.25, "ask": 150.30},
   "MSFT": {"bid": 299.50, "ask": 299.75},
   "GOOG": {"bid": 2800.00, "ask": 2801.50}
}
```

and the schema for position data received will be 
```python
[
   {"ticker": "AAPL", "quantity": 10, "bought_at": 145.00, "cost_basis": 1450.00},
   {"ticker": "MSFT", "quantity": -5, "bought_at": 300.00, "cost_basis": -1500.00},
   {"ticker": "GOOG", "quantity": 20, "bought_at": 2750.00, "cost_basis": 55000.00}
]
```

### Persistent and Non-Persistent Data Between Trading Loops 

If you would like to run strategies that depend on data collected over the course of the trading session (moving averages etc.), you should collect data in a structure outside of your custom trading function. For example, if you wanted to collect the quotes passed to the trading algorithm each time it was called, you would set up your code in the following way: 
```python

quote_collection = []

def trading_strat(context):
    quotes, positions = context['get_current_data']()
    quote_collection.append(quote)

    #trading logic...

    return orders

if __name__ == "__main__":

    run_trading(
        strategy_callable = trading_strat,
        ...
    )
```

If you were to instead initialize your list inside the trading function, it would be wiped every time your trading logic runs. You can also read in csv files or other past data and have it persist from execution to execution, it just needs to be read in outside of the strategy function itself.  

### Composing and Submitting Orders 

Sides: `"buy", "sell"`
Actions: `"buy_to_cover", "sell_long", "short_sell", "liquidate", "cover_short"` 

| Side  | Action         | Result                    | 
|-------|--------------  |---------------------------|
| buy   | none           | open long                 | 
| buy   | buy_to_cover   | close short               | 
| buy   | cover_short    | close short (used in eos) |
| sell  | none/sell_long | close long                | 
| sell  | short_sell     | open short                | 
| sell  | liquidate      | close long (used in eos)  |  

the schema for individual orders is 

```python
{
    "ticker": str,         # e.g. "AAPL"
    "quantity": float,     # e.g. 10
    "side": str,           # "buy" or "sell"
    "order_type": str,     # "market" or "limit"
    "action": str,         # optional: "buy_to_cover", "short_sell", "sell_long", "liquidate", "cover_short"
    "limit_price": float,  # required if order_type is "limit"
}
```

to submit a block of orders, simply combine them into a list 
```python
orders = [
    {"ticker": "AAPL", "quantity": 10, "side": "buy", "order_type": "market"},
    {"ticker": "MSFT", "quantity": 5, "side": "buy", "order_type": "market"},
    {"ticker": "GOOGL", "quantity": 3, "side": "sell", "order_type": "market"},
]
```

there are two options for submitting orders: 
```python
#Option 1

def trading_strat(context):
    #retrieve quotes, positions, run trading logic to yield orders 

    context['submit_order'](orders)
    return None

#Option 2

def trading_strat(context):
    #retrieve quotes, positions, run trading logic to yield orders 

    return orders
```
Both of these options are different ways to access the same execution function. 

## Calling Protocol 

It is important to call `run_trading` as follows 
```python
#global variables for initialization

def trading_function(context):
    return orders

if __name__ == "__main__":
    run_trading(
        strategy_callable = trading_function,
        #.....
    )
``` 
because `run_trading` spawns child processes, and calling run_trading outside the main block may spawn extra child processes during import. 


## Custom EOS Behvaior  
















    


