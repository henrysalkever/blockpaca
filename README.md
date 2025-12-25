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
        frequency = 10, #run the strategy every 10 seconds
        run_seconds = 60 #total seconds to run the trading loop for
        eos_behavior = "hold", #end of loop behavior
        warm_up_period = 10, #time to wait for quotes before starting the trading loop
        live_display = True, #toggle on/off for terminal rich-based display of positions and trade status,
        display_refresh_rate = 0.05, #time between each refresh of values in live display
        max_trade_updates = 20, #controls how many trade updates are shown in the live display
    )
```


    


