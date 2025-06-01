import streamlit as st
import pandas as pd

st.title("📊 FXバックテストツール（EMAクロス）")

# ファイルアップロード
uploaded_file = st.file_uploader("✅ TrueFXのCSVファイルをアップロードしてください", type=["csv"])

if uploaded_file is not None:
    # CSV読み込み
    df = pd.read_csv(uploaded_file, skiprows=1)
    df.columns = ["Date", "Open", "High", "Low", "Close", "EMA50", "EMA100", "Signal", "Position"]
    df["Date"] = pd.to_datetime(df["Date"])

    st.success("✅ データ読み込み成功！")
    st.dataframe(df.head())

    # バックテストボタン
    if st.button("▶️ バックテスト実行"):

        trades = []
        position = None
        entry_price = None
        entry_time = None

        for i in range(1, len(df)):
            row = df.iloc[i]
            prev_row = df.iloc[i - 1]

            if position is None:
                if row["Signal"] == 1:
                    position = "long"
                    entry_price = row["Close"]
                    entry_time = row["Date"]
                elif row["Signal"] == -1:
                    position = "short"
                    entry_price = row["Close"]
                    entry_time = row["Date"]

            elif position == "long" and row["Signal"] != 1:
                exit_price = row["Close"]
                exit_time = row["Date"]
                pnl = (exit_price - entry_price) * 10000
                trades.append([entry_time, exit_time, "long", entry_price, exit_price, pnl])
                position = None

            elif position == "short" and row["Signal"] != -1:
                exit_price = row["Close"]
                exit_time = row["Date"]
                pnl = (entry_price - exit_price) * 10000
                trades.append([entry_time, exit_time, "short", entry_price, exit_price, pnl])
                position = None

        # 結果の表示
        result_df = pd.DataFrame(trades, columns=["Entry Time", "Exit Time", "Direction", "Entry Price", "Exit Price", "PnL (pips)"])
        st.success(f"💡 トレード数：{len(result_df)}")
        st.dataframe(result_df)

        # CSVダウンロード
        csv = result_df.to_csv(index=False).encode("utf-8")
        st.download_button("⬇️ 結果をCSVでダウンロード", csv, "backtest_results.csv", "text/csv")
