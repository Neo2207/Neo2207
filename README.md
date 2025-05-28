import ephem
import yfinance as yf
import datetime
from kivy.app import App
from kivy.uix.label import Label
from kivy.uix.boxlayout import BoxLayout

def get_moon_dates(start_date, end_date):
    current = ephem.Date(start_date)
    full_moons = []
    while current < ephem.Date(end_date):
        next_full = ephem.localtime(ephem.next_full_moon(current))
        if next_full > end_date:
            break
        full_moons.append(next_full.date())
        current = ephem.Date(next_full + 1)
    return full_moons

def analyze_btc(full_moons):
    data = yf.download("BTC-USD", period="1y")
    data['Date'] = data.index.date
    data.reset_index(drop=True, inplace=True)

    results = []
    for moon_date in full_moons:
        if moon_date not in data['Date'].values:
            continue
        idx = data[data['Date'] == moon_date].index[0]
        if idx + 3 < len(data):
            before = data.loc[idx]['Close']
            after = data.loc[idx + 3]['Close']
            results.append((moon_date, after > before))

    buys = sum(1 for d in results if d[1])
    sells = len(results) - buys
    return "BUY" if buys > sells else "SELL"

class MoonApp(App):
    def build(self):
        layout = BoxLayout(orientation='vertical')
        today = datetime.date.today()
        start_date = today - datetime.timedelta(days=365)
        full_moons = get_moon_dates(start_date, today)
        signal = analyze_btc(full_moons)
        next_full = ephem.localtime(ephem.next_full_moon(ephem.now())).date()
        label = Label(text=f"Next Full Moon: {next_full}\nSignal: {signal}",
                      font_size='20sp')
        layout.add_widget(label)
        return layout

if __name__ == '__main__':
    MoonApp().run()
