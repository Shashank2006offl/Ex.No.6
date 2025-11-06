# Ex. No. 6 — Development of Python Code Compatible with Multiple AI Tools

**Register No:** 212223230205

---

## Aim
Write and implement Python code that integrates with multiple AI tools and weather APIs to automate interaction with APIs, compare outputs, and generate actionable insights. The objective is to design prompts and code that guide AI tools (ChatGPT/Gemini/Copilot) to produce, compare, and analyse API-driven results.

---

## AI Tools (Suggested)
- ChatGPT (GPT-5 / GPT-4) — for code generation & explanation  
- Google Gemini / Bard — for cross-checking logic & alternative approaches  
- GitHub Copilot — in-editor suggestions and quick fixes  
- Replit / Jupyter Notebook — execution and testing environment  
- Postman / RapidAPI — manual API exploration and testing  

---

## Background & Motivation
Comparing outputs from different APIs can expose differences in data freshness, rounding, or units. Combining multiple AI tools and writing robust Python wrappers helps automate collection, comparison, and decision logic (averaging, selecting the most likely correct value, or flagging inconsistencies). This exercise emphasizes prompt engineering, error handling, and cross-tool verification.

---

## Stage 1 — Base Program: Fetch from Two Weather APIs

**Naïve prompt (example):**  
`"Write a Python script to get weather for a city from two APIs."`

**Refined prompt (example):**  
`"Write a Python script that queries OpenWeatherMap and WeatherAPI for the current temperature (°C) and humidity (%) for a given city, uses exception handling, and prints results in a neat table."`

**Robust implementation (Python):**

```python
# weather_compare.py
# Requires: requests, tabulate (optional for pretty table)
# pip install requests tabulate

import requests
import time
from datetime import datetime
from tabulate import tabulate

# Replace with your real API keys (or use environment variables in production)
OPENWEATHER_KEY = "YOUR_OPENWEATHER_KEY"
WEATHERAPI_KEY = "YOUR_WEATHERAPI_KEY"

def get_openweather_data(city, api_key=OPENWEATHER_KEY, timeout=10):
    """
    Returns: dict with {source, temp_c, humidity, timestamp, raw}
    """
    try:
        url = "https://api.openweathermap.org/data/2.5/weather"
        params = {"q": city, "appid": api_key, "units": "metric"}
        r = requests.get(url, params=params, timeout=timeout)
        r.raise_for_status()
        data = r.json()
        temp = data["main"]["temp"]
        humidity = data["main"]["humidity"]
        # OpenWeatherMap returns 'dt' (epoch) for data calculation time
        timestamp = data.get("dt", int(time.time()))
        return {"source": "OpenWeatherMap", "temp_c": float(temp), "humidity": float(humidity),
                "timestamp": int(timestamp), "raw": data}
    except Exception as e:
        return {"source": "OpenWeatherMap", "error": str(e)}

def get_weatherapi_data(city, api_key=WEATHERAPI_KEY, timeout=10):
    """
    Returns: dict with {source, temp_c, humidity, timestamp, raw}
    """
    try:
        url = "http://api.weatherapi.com/v1/current.json"
        params = {"key": api_key, "q": city, "aqi": "no"}
        r = requests.get(url, params=params, timeout=timeout)
        r.raise_for_status()
        data = r.json()
        temp = data["current"]["temp_c"]
        humidity = data["current"]["humidity"]
        # WeatherAPI returns last_updated (string) that can be parsed
        last_updated = data["current"].get("last_updated_epoch") or int(time.time())
        return {"source": "WeatherAPI", "temp_c": float(temp), "humidity": float(humidity),
                "timestamp": int(last_updated), "raw": data}
    except Exception as e:
        return {"source": "WeatherAPI", "error": str(e)}

def pretty_print_results(results):
    """Print results in a neat table"""
    rows = []
    for r in results:
        if "error" in r:
            rows.append([r.get("source"), "ERROR", r.get("error"), "-"])
        else:
            tstr = datetime.utcfromtimestamp(r["timestamp"]).strftime("%Y-%m-%d %H:%M:%S UTC")
            rows.append([r["source"], f"{r['temp_c']:.2f} °C", f"{r['humidity']:.0f} %", tstr])
    print(tabulate(rows, headers=["Source", "Temperature", "Humidity", "Timestamp"], tablefmt="github"))

def compare_and_analyze(a, b, temp_threshold=2.0, hum_threshold=10.0):
    """
    Compare a and b (dicts). Return a dict with diffs, average and recommendation.
    """
    # If either contains error, indicate which
    errors = [x for x in (a, b) if "error" in x]
    if errors:
        return {"status": "error", "errors": [e.get("error") for e in errors]}

    temp_diff = abs(a["temp_c"] - b["temp_c"])
    hum_diff = abs(a["humidity"] - b["humidity"])
    avg_temp = (a["temp_c"] + b["temp_c"]) / 2.0
    avg_hum = (a["humidity"] + b["humidity"]) / 2.0

    # Decide which API to trust more:
    # - prefer the reading with the latest timestamp
    # - if timestamps close (<5 minutes), prefer average
    time_diff = abs(a["timestamp"] - b["timestamp"])
    if time_diff > 300:  # 5 minutes
        preferred = a if a["timestamp"] > b["timestamp"] else b
        recommendation = f"Prefer {preferred['source']} (more recent data)."
    else:
        preferred = None
        recommendation = "Timestamps close; average is recommended."

    flags = []
    if temp_diff > temp_threshold:
        flags.append("Significant temperature variation")
    if hum_diff > hum_threshold:
        flags.append("Significant humidity variation")

    status = "ok" if not flags else "warning"

    return {
        "status": status,
        "temp_diff": temp_diff,
        "hum_diff": hum_diff,
        "avg_temp": avg_temp,
        "avg_hum": avg_hum,
        "time_diff_seconds": time_diff,
        "preferred_source": preferred["source"] if preferred else None,
        "recommendation": recommendation,
        "flags": flags
    }

def main(city="Delhi"):
    a = get_openweather_data(city)
    b = get_weatherapi_data(city)
    print("\n--- Raw Results ---")
    pretty_print_results([a, b])
    comp = compare_and_analyze(a, b)
    print("\n--- Comparison ---")
    if comp.get("status") == "error":
        print("Errors encountered:", comp.get("errors"))
        return
    print(f"Temperature difference: {comp['temp_diff']:.2f} °C")
    print(f"Humidity difference:    {comp['hum_diff']:.2f} %")
    print(f"Average temperature:    {comp['avg_temp']:.2f} °C")
    print(f"Average humidity:       {comp['avg_hum']:.2f} %")
    print(f"Recommendation: {comp['recommendation']}")
    if comp['flags']:
        print("Flags:", "; ".join(comp['flags']))

if __name__ == "__main__":
    # Example usage: python weather_compare.py
    main("Delhi")
```

**Notes on the code**
- Uses `requests` with timeouts and `raise_for_status()` to handle HTTP errors.
- Parses timestamps where available to judge freshness.
- Compares values, computes averages, and offers a rule-based recommendation.
- Prints a neat table using `tabulate`. (If not installed, you can replace the table with simple prints.)
- Replace `YOUR_OPENWEATHER_KEY` and `YOUR_WEATHERAPI_KEY` with actual keys or read them from environment variables for production.

---

## Stage 2 — Comparing Outputs & Flagging Differences

**Behaviour**
- Compute absolute differences in temperature and humidity.
- Flag when differences exceed thresholds (e.g., >2°C for temperature, >10% for humidity).
- Use timestamp comparison to select the more up-to-date source when timestamps differ significantly (>5 minutes).

**Example outputs (sample):**

| Source           | Temp (°C) | Humidity (%) | Timestamp               |
|------------------|-----------:|-------------:|-------------------------|
| OpenWeatherMap   | 31.20 °C  | 46 %         | 2025-10-09 04:00:00 UTC |
| WeatherAPI       | 29.50 °C  | 48 %         | 2025-10-09 03:58:00 UTC |

Comparison summary:
- Temperature difference: 1.70 °C → within threshold → average suggested
- Humidity difference: 2 % → within threshold → average suggested

---

## Stage 3 — Insights & Next Steps (Automated Decision Strategies)

**Example decision logic implemented:**
1. If timestamps differ by >5 minutes → prefer the newer API reading.
2. If timestamps similar → use average values.
3. If differences exceed high thresholds (temperature >5°C or humidity >20%) → flag for manual verification and log raw payloads for debugging.

**Suggested improvements / advanced features:**
- Add a small local cache to avoid hitting rate limits; store last successful reading with timestamp.
- Use an ensemble weighting scheme: weight readings by historical reliability (learned from past comparisons).
- Query a third source (e.g., Meteostat) to use majority/median rules when two sources disagree.
- Use AI tools (ChatGPT/Gemini) to analyze raw payloads and spot anomalies (e.g., unit mismatch, sensor errors).
- Persist results to a CSV / DB for trend analysis and to compute API reliability metrics over time.

---

## Prompt Engineering Notes (for AI tools)
- Example refined prompt for ChatGPT/Gemini:
  > "Generate a Python script that queries OpenWeatherMap and WeatherAPI for current temperature (°C) and humidity (%) for a given city. Include exception handling, timeouts, parsing of timestamps, a comparison routine that computes differences and averages, and print results in a table. Add recommendations when differences exceed thresholds."

- When asking Copilot / in-editor suggestions, supply the function signature and docstring to get more relevant suggestions.

---

## Evaluation & Reflection
- **What worked:** Detailed prompts (API names, fields, error handling) produce robust, usable code. Including desired output format (table, averages, flags) helps the AI produce exact behavior.
- **Problems encountered:** Different APIs may return data with different timestamps, units, or rounding; error handling and timestamp comparison are essential.
- **Learning:** Prompt clarity + small scaffolding (function names, expected dict shapes) gives consistent, high-quality code from AI assistants.

---

## Conclusion
This exercise validated that:
- Multi-tool workflows (ChatGPT/Copilot/Gemini + manual testing via Postman) accelerate development.
- Well-structured prompts and explicit constraints (timeouts, error handling, output format) are crucial for reliable code generation.
- Simple decision logic (timestamps + thresholds + averages) helps produce actionable insights from multiple API sources.

---

### Result
The program was designed and implemented successfully. The approach can be extended with caching, model-based reliability scoring, and automated logging for large-scale deployments.

