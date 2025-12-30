# Web Unlocker API

[![Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/) 

[Web Unlocker](https://brightdata.jp/products/web-unlocker) は強力なスクレイピング API であり、高度なボット保護を回避しながらあらゆるWebサイトへアクセスできます。複雑なアンチボット基盤を管理することなく、1回のAPI呼び出しでクリーンなHTML/JSONレスポンスを取得できます。

# Table of Contents
- [Features](#features)
- [Getting Started](#getting-started)
   - [Direct API Access](#direct-api-access)
   - [Native Proxy-Based Access](#native-proxy-based-access)
- [Practical Example: Scraping G2 Reviews](#practical-example-scraping-g2-reviews)
   - [Basic Request (Without Web Unlocker)](#basic-request-without-web-unlocker)
   - [Enhanced Request (With Web Unlocker)](#enhanced-request-with-web-unlocker)
      - [Direct API Access](#direct-api-access)
      - [Proxy-Based Access](#proxy-based-access)
      - [Waiting for Specific Elements](#waiting-for-specific-elements)
      - [Mobile User-Agent Targeting](#mobile-user-agent-targeting)
      - [Geolocation Targeting](#geolocation-targeting)
      - [Debugging Requests](#debugging-requests)
      - [Success Rate Statistics](#success-rate-statistics)
- [Final Notes](#final-notes)

## Features
Web Unlocker は包括的なWebスクレイピング機能を提供します:
- 自動プロキシ管理とCAPTCHA解決
- 実ユーザー行動のシミュレーション
- 組み込みのJavaScriptレンダリング
- グローバルなジオロケーションターゲティング
- 自動リトライメカニズム
- 成功課金（pay-per-success）の価格モデル

## Getting Started
Web Unlocker を使用する前に、[quickstart guide](https://docs.brightdata.com/scraping-automation/web-unlocker/quickstart) に従ってセットアップを完了してください。

### Direct API Access
Web Unlocker を統合するための推奨方法です。


**Example: cURL Command**
```bash
curl -X POST "https://api.brightdata.com/request" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer INSERT_YOUR_API_TOKEN" \
-d '{
  "zone": "INSERT_YOUR_WEB_UNLOCKER_ZONE_NAME",
  "url": "http://lumtest.com/myip.json",
  "format": "raw"
}'
```

1. API Endpoint: `https://api.brightdata.com/request`
2. Authorization Header: Web Unlocker API zone の [API token](https://docs.brightdata.com/scraping-automation/web-unlocker/send-your-first-request#generating-your-bright-data-api-token)
3. Payload:
   - `zone`: Web Unlocker API zone 名
   - `url`: アクセス対象のターゲットURL
   - `format`: レスポンス形式（サイトのレスポンスを直接返すには `raw` を使用します）

**Example: Python Script**
```python
import requests

API_URL = "https://api.brightdata.com/request"
API_TOKEN = "INSERT_YOUR_API_TOKEN"
ZONE_NAME = "INSERT_YOUR_WEB_UNLOCKER_ZONE_NAME"
TARGET_URL = "http://lumtest.com/myip.json"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_TOKEN}"
}

payload = {
    "zone": ZONE_NAME,
    "url": TARGET_URL,
    "format": "raw"
}

response = requests.post(API_URL, headers=headers, json=payload)

if response.status_code == 200:
    print("Success:", response.text)
else:
    print(f"Error {response.status_code}: {response.text}")
```

### Native Proxy-based Access

プロキシベースのルーティングを使用する代替方法です。

**Example: cURL Command**
```bash
curl "http://lumtest.com/myip.json" \
--proxy "brd.superproxy.io:33335" \
--proxy-user "brd-customer-<CUSTOMER_ID>-zone-<ZONE_NAME>:<ZONE_PASSWORD>"
```

必要な認証情報:
1. Customer ID: [Account settings](https://brightdata.jp/cp/setting/customer_details) にあります
2. Web Unlocker API zone 名: overview タブにあります
3. Web Unlocker API password: overview タブにあります

**Example: Python Script**
```python
import requests

customer_id = "<customer_id>"
zone_name = "<zone_name>"
zone_password = "<zone_password>"

host = "brd.superproxy.io"
port = 33335
proxy_url = f"http://brd-customer-{customer_id}-zone-{zone_name}:{zone_password}@{host}:{port}"

proxies = {"http": proxy_url, "https": proxy_url}

response = requests.get("http://lumtest.com/myip.json", proxies=proxies)

if response.status_code == 200:
    print(response.json())
else:
    print(f"Error: {response.status_code}")
```

## Practical Example: Scraping G2 Reviews
Cloudflare によって強固に保護されているサイトである [G2.com](https://www.g2.com/) から、レビューをスクレイピングする方法を見ていきます。

### Basic Request (Without Web Unlocker)
シンプルなPythonスクリプトを使用して [G2 reviews](https://www.g2.com/products/mongodb/reviews) をスクレイピングします:
```python
import requests
from bs4 import BeautifulSoup

url = 'https://www.g2.com/products/mongodb/reviews'
response = requests.get(url)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = soup.find_all('h2')
    
    if headings:
        print("\nHeadings Found:")
        for heading in headings:
            print(f"- {heading.get_text(strip=True)}")
    else:
        print("No headings found")
else:
    print("Request blocked")
```

**Result:** Cloudflare のアンチボット対策によりスクリプトは失敗（`403` エラー）します。


### Enhanced Request (With Web Unlocker)
このような制限を回避するには、Web Unlocker を使用します。以下はPythonによる実装です:

#### Direct API Access
```python
import requests
from bs4 import BeautifulSoup

API_URL = "https://api.brightdata.com/request"
API_TOKEN = "INSERT_YOUR_API_TOKEN"
ZONE_NAME = "INSERT_YOUR_ZONE"
TARGET_URL = "https://www.g2.com/products/mongodb/reviews"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_TOKEN}"
}
payload = {"zone": ZONE_NAME, "url": TARGET_URL, "format": "raw"}

response = requests.post(API_URL, headers=headers, json=payload)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = [h.get_text(strip=True) for h in soup.find_all('h2')]
    print("\nExtracted Headings:", headings)
else:
    print(f"Error {response.status_code}: {response.text}")
```
**Result:** 保護を正常に回避し、ステータス `200` でコンテンツを取得します。

#### Proxy-Based Access
代替として、プロキシベースの方法を使用します:
```python
import requests
from bs4 import BeautifulSoup

proxy_url = "http://brd-customer-<customer_id>-zone-<zone_name>:<zone_password>@brd.superproxy.io:33335"
proxies = {"http": proxy_url, "https": proxy_url}

url = "https://www.g2.com/products/mongodb/reviews"
response = requests.get(url, proxies=proxies, verify=False)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = [h.get_text(strip=True) for h in soup.find_all('h2')]
    print("\nExtracted Headings:", headings)
else:
    print(f"Error {response.status_code}: {response.text}")
```

**Note:** 以下を追加してSSL証明書の警告を抑制します:
```python
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```

#### Waiting for Specific Elements
`x-unblock-expect` ヘッダーを使用して、特定の要素またはテキストを待機します:
```python
headers["x-unblock-expect"] = '{"element": ".star-wrapper__desc"}'
# or
headers["x-unblock-expect"] = '{"text": "reviews"}'
```

👉 完全なコードは [g2_wait.py](https://github.com/bright-jp/web-unlocker/blob/main/src/g2_wait.py) で確認できます

#### Mobile User-Agent Targeting
デスクトップではなくモバイルのuser agentを使用するには、username に `-ua-mobile` を付与します:
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-ua-mobile"
```
👉 完全なコードは [g2_mobile.py](https://github.com/bright-jp/web-unlocker/blob/main/src/g2_mobile.py) で確認できます

#### Geolocation Targeting
Web Unlocker は最適なIPロケーションを自動選択しますが、ターゲットロケーションを指定することもできます:
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-country-us"
username = f"brd-customer-{customer_id}-zone-{zone_name}-country-us-city-sanfrancisco"
```

👉 詳細は [here](https://docs.brightdata.com/api-reference/proxy/geolocation-targeting) で確認できます。

#### Debugging Requests
`-debug-full` フラグを追加して詳細なデバッグ情報を有効化します:
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-debug-full"
```
👉 完全なコードは [g2_debug.py](https://github.com/bright-jp/web-unlocker/blob/main/src/g2_debug.py) で確認できます

#### Success Rate Statistics
特定ドメインのAPI成功率を監視します:
```python
import requests

API_TOKEN = "INSERT_YOUR_API_TOKEN"

def get_success_rate(domain):
    url = f"https://api.brightdata.com/unblocker/success_rate/{domain}"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {API_TOKEN}"
    }
    response = requests.get(url, headers=headers)
    print(response.json() if response.status_code == 200 else response.text)

get_success_rate("g2.com") # Get statistics for specific domain
get_success_rate("g2.*") # Get statistics for all top-level domains
```

## Final Notes
Web Unlocker を使用すると、最も強固に保護されたWebサイトであっても簡単にスクレイピングできます。以下の重要ポイントを覚えておいてください:

1. **Not Compatible With**:  
   - ブラウザ（Chrome, Firefox, Edge）  
   - アンチ検知ブラウザ（Adspower, Multilogin）  
   - 自動化ツール（Puppeteer, Playwright, Selenium）  

2. **Use Scraping Browser**:  
   ブラウザベースの自動化には、Bright Data の [Scraping Browser](https://brightdata.jp/products/scraping-browser) を使用してください。

3. **Premium Domains**:  
   [premium domain](https://docs.brightdata.com/scraping-automation/web-unlocker/features#web-unlocker-api-premium-domains) 機能で難易度の高いサイトへアクセスできます。

4. **CAPTCHA Solving**:  
   自動的に解決されますが、[disabled](https://docs.brightdata.com/scraping-automation/web-unlocker/features#disable-captcha-solving) にできます。Bright Data の [CAPTCHA Solver](https://brightdata.jp/products/web-unlocker/captcha-solver) についても詳しく確認してください。
   
5. **Custom Headers & Cookies**:  
   狙ったサイトバージョンを対象にするために、独自のヘッダーとCookieを送信できます。 [Learn more](https://docs.brightdata.com/scraping-automation/web-unlocker/features#manual-headers-and-cookies)。

詳細は [official documentation](https://docs.brightdata.com/scraping-automation/web-unlocker/introduction) を参照してください。