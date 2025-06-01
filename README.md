# BackTest
import pandas as pd

# データの読み込み
df = pd.read_csv("sample_H1_data.csv", skiprows=1)  # 最初のヘッダー行をスキップ
df.columns = ["Date", "Open", "High", "Low", "Close", "EMA50", "EMA100", "Signal", "Position"]
df["Date"] = pd.to_datetime(df["Date"])

# バックテスト処理
trades = []
position = None
entry_price = None
entry_time = None

for i in range(1, len(df)):
    row = df.iloc[i]
    prev_row = df.iloc[i - 1]

    # エントリー条件
    if position is None:
        if row["Signal"] == 1:
            position = "long"
            entry_price = row["Close"]
            entry_time = row["Date"]
        elif row["Signal"] == -1:
            position = "short"
            entry_price = row["Close"]
            entry_time = row["Date"]

    # エグジット条件（シグナルが反対 or 0）
    elif position == "long" and row["Signal"] != 1:
        exit_price = row["Close"]
        exit_time = row["Date"]
        pnl = (exit_price - entry_price) * 10000  # pips換算（例：EURUSDなら×10000）
        trades.append([entry_time, exit_time, "long", entry_price, exit_price, pnl])
        position = None

    elif position == "short" and row["Signal"] != -1:
        exit_price = row["Close"]
        exit_time = row["Date"]
        pnl = (entry_price - exit_price) * 10000
        trades.append([entry_time, exit_time, "short", entry_price, exit_price, pnl])
        position = None

# データフレーム化と保存
result_df = pd.DataFrame(trades, columns=["Entry Time", "Exit Time", "Direction", "Entry Price", "Exit Price", "PnL (pips)"])
result_df.to_csv("backtest_results.csv", index=False)

print("✅ バックテスト完了！backtest_results.csv に結果を保存しました。")
