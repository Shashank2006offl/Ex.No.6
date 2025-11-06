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
        r = requests.get(url
