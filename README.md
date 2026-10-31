# DeepVayu

**AI-powered Diwali air quality prediction**
*See it. Understand it. Predict it.*

Team **Ghost Bytes**: Sidh Thukral, Samir, Pratham Kapoor, Ronak Dhannani

Diwali 2026 is on **8 November 2026**.




---

## 1. Problem statement

Develop an AI-powered platform to predict neighborhood-level air pollution before, during, and after Diwali by analysing historical air-quality data, weather conditions, traffic patterns, and other relevant environmental indicators.

### Why it is hard

- **One number hides the story.** A single city-wide AQI does not show how different each neighborhood is.
- **Diwali night is abrupt.** Air can change sharply between morning, evening and night, and daily averages miss that.
- **Causes are mixed.** Wind, crop-residue fires, traffic and fireworks all add up, so blaming crackers alone is wrong.

---

## 2. What DeepVayu does

DeepVayu records and compares air quality across six windows around Diwali, explains what is driving it, and predicts air quality for each neighborhood with a confidence level.

| Window | Period |
|---|---|
| Before | 1 month before, 7 days before, 1 day before |
| During | Diwali day: morning, evening, night |
| After | 7 days after, 1 month after |

It does not just show how bad the air is. It tells you why, where, when to step out, and what is coming next.

---

## 3. Features

1. **Six-window Diwali timeline** (month before to month after, with Diwali day split into morning, evening and night).
2. **Neighborhood forecast map** with zones coloured low, moderate and high.
3. **Diwali Night Spike Score (DNSS):** how far Diwali night rises above that area's normal level (formula in section 5).
4. **What's driving it:** wind and weather, nearby fires, traffic proxy, and the Diwali effect.
5. **Confidence and transparency:** every forecast shows a prediction range, number of nearby stations, data coverage and a reliability level.
6. **Source labels:** each number says whether it came from a ground station, a satellite, or the model.
7. **Better Air Window:** compares forecast periods (for example 8 AM, 2 PM, 10 PM) as relatively better, use caution, or higher risk, with health tips for children, older adults and people with asthma. It is not medical advice and points users to official health advisories.
8. **City officials view:** hotspot ranking, areas where pollution is rising fast, and areas with too few monitors.
9. **Hindi and English.**
10. **Cached demo data:** saved copies of past Diwali data keep the demo running if the internet or an API fails. The app labels it as saved data.
11. **Year-to-year comparison** (for example 2024 vs 2025), with a note that weather also changes every year.
12. **What-if simulator (next step):** explore "what if wind drops or traffic rises". It shows what the model has learned, not a guaranteed outcome.

---

## 4. Data sources

| Source | What it gives | Used for | Limit |
|---|---|---|---|
| OpenAQ (CPCB stations) | Hourly ground readings (PM2.5, PM10 and more) | Hourly morning, evening and night data; model training | Few stations per city |
| NASA Earthdata | Satellite layers for gases and particles | Daily area-wide picture | About one pass a day, needs daylight, blocked by cloud and smog |
| NASA GIBS / Worldview | Satellite images | Map layers | Same as above |
| NASA FIRMS | Fire hotspots | Checking whether nearby fires explain a spike | Shows fires, not smoke amount |
| NASA POWER | Weather data | Wind, humidity and similar | Coarser than local sensors |
| Open-Meteo | Hourly weather and air-quality model data | Hour-by-hour weather | Model data, not a measurement |
| Traffic proxy | Road density or night-lights as a stand-in | Estimating traffic pressure per neighborhood | Not live traffic |

**Why both satellites and ground stations?** Most satellites pass over a place about once a day, need daylight, and are blocked by cloud and thick smog. NASA TEMPO is hourly but covers North America only, not India. So satellites give the daily area-wide view, and ground stations give the hourly morning, evening and night readings.

Check each source's login or key rules before use.

---

## 5. Method and formulas

All formulas are written in plain text.

### Pipeline

1. **Collect** data from the sources above.
2. **Prepare:** clean and align by hour and place.
3. **Features:** wind, humidity, nearby fires, hour of day, days from Diwali.
4. **Baseline:** compute the normal-day level for each station and hour.
5. **Predict:** a LightGBM model forecasts each station, trained on past Diwalis.
6. **Spread:** estimate neighborhoods from nearby stations.
7. **Explain and label:** show drivers, source and confidence.
8. **Serve:** map, timeline, advice and officials view.

### Baseline (normal-day level)

```
Pb = average PM2.5 for the same station and the same hour on normal (non-festival) days
```

The exact choice of "normal days" is our design decision. Document the dates you use.

### Diwali Night Spike Score (DNSS)

```
DNSS = ((Pd - Pb) / Pb) * 100
```

- `Pd` = predicted Diwali PM2.5
- `Pb` = normal-day baseline PM2.5
- Result is a percentage above (positive) or below (negative) normal.

Example (*illustrative*): Pb = 80, Pd = 160, so DNSS = ((160 - 80) / 80) * 100 = 100%.

DNSS is our own comparison indicator, **not an official AQI measurement**.

### Neighborhood estimate (suggested method, adjust to what is built)

```
wi = 1 / (di * di)
P_neighbourhood = (w1*P1 + w2*P2 + ... + wn*Pn) / (w1 + w2 + ... + wn)
```

- `Pi` = prediction at nearby station i
- `di` = distance from the neighborhood to station i
- Neighborhoods far from any station get lower confidence and a "low data" warning.

### Model evaluation (proof plan)

Train on earlier Diwalis and test on a held-out past Diwali.

```
MAE  = average of |predicted - actual|
RMSE = square root of (average of (predicted - actual)^2)
```

Show one chart of predicted vs actual. Report only real results from our own test.

---

## 6. Honest limits

- Traffic is a **proxy**, not live traffic.
- Satellites show change from a baseline, **not proof of cause**. We say "consistent with fireworks", not "caused by".
- Many neighborhood values are **estimates** from sparse stations, so we show confidence and a "low data" warning.
- We start with **one demo city (Delhi NCR)** and a city selector.
- Diwali 2026 has not happened yet. Past Diwalis (2024, 2025 and earlier) train and test the model, and 2026 data is added as it arrives.
- Better Air Window is **not medical advice**.
- Example numbers such as 185 ug/m3, range 155-215, and the zone scores are *illustrative*.

---

## 7. Tech stack

- **Data and model:** Python, LightGBM
- **Backend:** FastAPI
- **Frontend:** React, with a web map library
- **Hosting:** free tiers

---

## 8. Suggested project structure

Update this to match your real repository.

```
deepvayu/
  data/
    cache/            saved past Diwali data for the offline demo
  pipeline/           fetching, cleaning, alignment, features
  model/              baseline, LightGBM training, evaluation
  api/                FastAPI backend
  web/                React frontend (map, timeline, advice, officials view)
  README.md
```

---

## 9. Getting started

Fill in the real commands once the project is set up.

```
# 1. Clone the repository
# 2. Install backend dependencies
# 3. Add your API keys or logins for the data sources (do not commit them)
# 4. Run the data pipeline, or use the cached demo data
# 5. Start the backend
# 6. Start the frontend
```

---

## 10. Project status

Tick what is truly done.

- [✔] Data pulled from OpenAQ
- [✔] Weather data pulled (NASA POWER / Open-Meteo)
- [✔] Fire data pulled (NASA FIRMS)
- [✔] Satellite layers added
- [✔] Baseline computed
- [✔] LightGBM model trained
- [✔] Held-out Diwali test done (predicted vs actual chart)
- [✔] Neighborhood map working
- [✔] DNSS shown in the app
- [✔] Confidence labels shown
- [✔] Better Air Window working
- [✔] City officials view working
- [✔] Hindi and English
- [✔] Cached demo data working
- [✔] What-if simulator (stretch goal)

---

## 11. Existing solutions and how we differ

Similar tools exist, including SAFAR (government forecasting), Google Air View+, Airveda, Ambee, Kaiterra Live Air, AQI India, and MANAS (Mumbai). They cover hyperlocal air quality or Diwali hot-spot forecasts.

DeepVayu is built around the Diwali story itself: six windows, the night spike against a normal baseline, an explanation of what is driving the air, an honest confidence level for every neighborhood, and advice for both citizens and city officials.

We do not claim to be better than these systems. They have far more sensors and data than we do.

---

## 12. Roadmap

- Live traffic data
- What-if simulator
- Satellite upgrades
- Alerts
- More cities and more festivals

---

## 13. Team

**Team Ghost Bytes**
Sidh Thukral, Samir, Pratham Kapoor, Ronak Dhannani




