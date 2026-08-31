# Forex Trading Bot - Step-by-Step Implementation Guide

## Phase 1: Setup & Configuration

### Step 1: Create OANDA Account
1. Go to [OANDA.com](https://www.oanda.com/)
2. Click **"Sign Up"** → Select **"Practice Account"**
3. Fill in your details and verify email
4. Log in to your practice account
5. Go to **Account Settings** → **API Access**
6. Generate a **Personal Access Token**
7. Copy your:
   - **API Key** (token)
   - **Account ID** (format: 000-000-0000000-001)

### Step 2: Clone & Setup Repository
```bash
# Clone the repository
git clone https://github.com/tumzamogale18-eng/forex-trading-bot.git
cd forex-trading-bot

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\\Scripts\\activate
# On macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Step 3: Configure Environment Variables
```bash
# Copy the example environment file
cp .env.example .env

# Open .env and add your OANDA credentials
# Edit with your text editor (VS Code, Notepad, etc.)
```

**Your .env file should look like:**
```
OANDA_API_KEY=your_actual_api_token_here
OANDA_ACCOUNT_ID=your_actual_account_id_here
OANDA_ENVIRONMENT=practice
TRADE_CURRENCY_PAIR=EUR_USD
TIMEFRAME=H1
RISK_PER_TRADE=0.02
```

---

## Phase 2: Test Connection to OANDA

### Step 4: Create Test Script
Create a new file `test_connection.py`:

```python
import os
from dotenv import load_dotenv
from bot.data_handler import DataHandler

# Load environment variables
load_dotenv()

# Initialize data handler
api_key = os.getenv('OANDA_API_KEY')
account_id = os.getenv('OANDA_ACCOUNT_ID')
environment = os.getenv('OANDA_ENVIRONMENT', 'practice')

if not api_key or not account_id:
    print("❌ Error: API credentials not configured!")
    exit(1)

print("✅ Credentials loaded successfully")

# Test connection
data_handler = DataHandler(api_key, account_id, environment)
account_info = data_handler.get_account_info()

if account_info:
    print("✅ Successfully connected to OANDA!")
    print(f"Account ID: {account_info['account']['id']}")
    print(f"Balance: {account_info['account']['balance']}")
else:
    print("❌ Failed to connect to OANDA")
```

### Step 5: Run Test Script
```bash
python test_connection.py
```

**Expected Output:**
```
✅ Credentials loaded successfully
✅ Successfully connected to OANDA!
Account ID: 000-000-0000000-001
Balance: 100000.00
```

---

## Phase 3: Test Data Retrieval

### Step 6: Fetch Market Data
Create `test_data.py`:

```python
import os
from dotenv import load_dotenv
from bot.data_handler import DataHandler

load_dotenv()

api_key = os.getenv('OANDA_API_KEY')
account_id = os.getenv('OANDA_ACCOUNT_ID')
environment = os.getenv('OANDA_ENVIRONMENT', 'practice')

data_handler = DataHandler(api_key, account_id, environment)

# Get last 100 hourly candles for EUR/USD
price_data = data_handler.get_candlestick_data(
    currency_pair='EUR_USD',
    granularity='H1',
    count=100
)

if price_data:
    print("✅ Successfully retrieved price data!")
    print(f"Number of candles: {len(price_data['close'])}")
    print(f"Latest close price: {price_data['close'][-1]}")
    print(f"Highest price: {max(price_data['high'])}")
    print(f"Lowest price: {min(price_data['low'])}")
else:
    print("❌ Failed to retrieve price data")
```

### Step 7: Run Data Test
```bash
python test_data.py
```

**Expected Output:**
```
✅ Successfully retrieved price data!
Number of candles: 100
Latest close price: 1.0856
Highest price: 1.0890
Lowest price: 1.0820
```

---

## Phase 4: Test Technical Indicators

### Step 8: Calculate Indicators
Create `test_indicators.py`:

```python
import os
from dotenv import load_dotenv
import numpy as np
from bot.data_handler import DataHandler
from bot.indicators import TechnicalIndicators

load_dotenv()

api_key = os.getenv('OANDA_API_KEY')
account_id = os.getenv('OANDA_ACCOUNT_ID')
environment = os.getenv('OANDA_ENVIRONMENT', 'practice')

# Get market data
data_handler = DataHandler(api_key, account_id, environment)
price_data = data_handler.get_candlestick_data(
    currency_pair='EUR_USD',
    granularity='H1',
    count=100
)

if not price_data:
    print("❌ Failed to get price data")
    exit(1)

# Calculate indicators
indicators = TechnicalIndicators()

# RSI
rsi = indicators.calculate_rsi(price_data['close'], period=14)
print(f"✅ RSI (last 5): {rsi[-5:]}")

# MACD
macd = indicators.calculate_macd(price_data['close'])
print(f"✅ MACD (latest): {macd['macd'][-1]:.6f}")
print(f"✅ MACD Signal (latest): {macd['signal'][-1]:.6f}")

# SMA
sma = indicators.calculate_sma(price_data['close'], fast=20, slow=50)
print(f"✅ Fast SMA (latest): {sma['fast'][-1]:.6f}")
print(f"✅ Slow SMA (latest): {sma['slow'][-1]:.6f}")

# Check for signals
if rsi[-1] < 30:
    print("🟢 OVERSOLD - Potential BUY signal!")
elif rsi[-1] > 70:
    print("🔴 OVERBOUGHT - Potential SELL signal!")
else:
    print("⚪ NEUTRAL - No clear signal")
```

### Step 9: Run Indicators Test
```bash
python test_indicators.py
```

---

## Phase 5: Test Trading Logic

### Step 10: Generate Trading Signals
Create `test_signals.py`:

```python
import os
from dotenv import load_dotenv
from bot.data_handler import DataHandler
from bot.trader import ForexTrader

load_dotenv()

api_key = os.getenv('OANDA_API_KEY')
account_id = os.getenv('OANDA_ACCOUNT_ID')
environment = os.getenv('OANDA_ENVIRONMENT', 'practice')

# Initialize trader
trader = ForexTrader(api_key, account_id, environment)

# Get price data
price_data = trader.data_handler.get_candlestick_data(
    currency_pair='EUR_USD',
    granularity='H1',
    count=100
)

if not price_data:
    print("❌ Failed to get price data")
    exit(1)

# Generate signal
signal = trader.get_signal(price_data)

print("📊 Trading Signal Analysis")
print(f"Direction: {signal['direction']}")
print(f"Strength: {signal['strength']:.2%}")
print(f"RSI: {signal['indicators']['rsi']:.2f}")
print(f"Fast SMA: {signal['indicators']['sma_fast']:.6f}")
print(f"Slow SMA: {signal['indicators']['sma_slow']:.6f}")

if signal['direction'] == 'BUY':
    print("\n🟢 BUY SIGNAL - Ready to execute trade!")
elif signal['direction'] == 'SELL':
    print("\n🔴 SELL SIGNAL - Ready to execute trade!")
else:
    print("\n⚪ HOLD - Waiting for better opportunity")
```

### Step 11: Run Signal Test
```bash
python test_signals.py
```

---

## Phase 6: Run Full Bot

### Step 12: Start the Trading Bot
```bash
python main.py
```

**Output should show:**
```
2024-01-15 10:30:45 - root - INFO - Starting Forex Trading Bot...
2024-01-15 10:30:46 - root - INFO - ForexTrader initialized for 000-000-0000000-001 in practice environment
2024-01-15 10:30:46 - root - INFO - Trading bot initialized successfully
2024-01-15 10:30:46 - root - INFO - Starting trading loop
```

---

## Phase 7: Monitoring & Debugging

### Step 13: Check Logs
```bash
# View trading logs
tail -f logs/trading_bot.log

# On Windows PowerShell:
Get-Content logs/trading_bot.log -Tail 20 -Wait
```

### Step 14: Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError` | Run `pip install -r requirements.txt` |
| `KeyError: 'OANDA_API_KEY'` | Check `.env` file exists and has correct format |
| `Connection refused` | Verify internet connection and OANDA API status |
| `401 Unauthorized` | Check API key and account ID are correct |
| `Empty price data` | Currency pair might not be available (try EUR_USD) |

---

## Phase 8: Enhance Your Bot

### Step 15: Add More Features
1. **Risk Management Module** - Calculate position size
2. **Order Management** - Place, modify, close trades
3. **Backtesting** - Test strategy on historical data
4. **Email Notifications** - Get alerts on trades
5. **Web Dashboard** - Monitor bot performance
6. **Machine Learning** - Improve signal accuracy

---

## Testing Checklist

- [ ] OANDA account created
- [ ] API credentials obtained
- [ ] Repository cloned
- [ ] Virtual environment activated
- [ ] Dependencies installed
- [ ] `.env` file configured
- [ ] Connection test passed
- [ ] Data retrieval test passed
- [ ] Indicators calculation test passed
- [ ] Signal generation test passed
- [ ] Main bot runs without errors
- [ ] Logs are being generated

---

## Next Steps

After successful implementation:
1. Let the bot run for a few days to monitor performance
2. Backtest the strategy on historical data
3. Adjust indicator parameters based on results
4. Add risk management features
5. Deploy additional trading strategies
6. Scale to live account (after thorough testing)

**Remember**: Always start with a practice account and thoroughly test before using real money!
