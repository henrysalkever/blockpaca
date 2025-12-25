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
    quotes, positions = conext['get_current_data]()
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

    


