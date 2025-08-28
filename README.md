# Web-Scraper-with-Proxy-Rotation
Develop a robust web scraper that intelligently rotates through proxy IP addresses to  bypass geo-restrictions and anti-bot mechanisms. The system will handle CAPTCHAs  using third-party solving services, implement retry strategies for failed requests, and  extract structured data from target websites while avoiding detection.
import requests
from bs4 import BeautifulSoup
import random
import time


PROXIES = [
    "http://123.456.789.001:8080",
    "http://123.456.789.002:8080",
    "http://123.456.789.003:8080"
]


HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/114.0 Safari/537.36"
}

# --- Simple CAPTCHA solving stub ---
def solve_captcha(image_url):
    """
    Fake solver — in real use, send image_url to a CAPTCHA service API.
    Example: 2Captcha, AntiCaptcha, etc.
    """
    print(f"⚠️ CAPTCHA detected: {image_url}")
    # Here you'd integrate with an API and return the solved text.
    return "solved_captcha_text"


# --- Robust fetch with retries and proxy rotation ---
def fetch_url(url, max_retries=5):
    for attempt in range(max_retries):
        proxy = {"http": random.choice(PROXIES), "https": random.choice(PROXIES)}
        try:
            print(f"🌍 Trying {url} with proxy {proxy['http']}")
            response = requests.get(url, headers=HEADERS, proxies=proxy, timeout=10)

            if response.status_code == 200:
                # Check if page contains CAPTCHA
                if "captcha" in response.text.lower():
                    # Handle CAPTCHA (stub example)
                    solve_captcha("captcha_image_url_here")
                    continue  # retry after solving
                return response.text
            else:
                print(f"⚠️ Bad status code: {response.status_code}")
        except requests.RequestException as e:
            print(f"❌ Request failed: {e}")

        # wait before retrying (avoid bans)
        time.sleep(random.uniform(1, 3))

    print("❌ Failed to fetch after retries.")
    return None


# --- Scraper logic ---
def scrape_site(url):
    html = fetch_url(url)
    if not html:
        return []

    soup = BeautifulSoup(html, "html.parser")

    # Example: scrape all article titles
    data = []
    for item in soup.select("h2"):   # change selector for your target
        data.append(item.get_text(strip=True))

    return data


# --- Main ---
if __name__ == "__main__":
    target_url = "https://example.com/news"
    results = scrape_site(target_url)

    if results:
        print("✅ Extracted Data:")
        for r in results:
            print("-", r)
    else:
        print("⚠️ No data extracted.")
