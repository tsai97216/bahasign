import os
import json
import sys
import requests
from datetime import datetime

# 從環境變數讀取資料
account = os.getenv("USER_ACCOUNT")
password = os.getenv("USER_PASSWORD")
webhook_url = os.getenv("DISCORD_WEBHOOK")


def send_discord_msg(content):
    if webhook_url:
        try:
            requests.post(
                webhook_url,
                json={
                    "username": "簽到",
                    "embeds": [
                        {
                            "title": "巴哈姆特簽到通知",
                            "description": content,
                            "color": 0x00FF99,
                            "footer": {
                                "text": "Auto Check-in Bot"
                            },
                            "timestamp": datetime.utcnow().isoformat()
                        }
                    ]
                },
                timeout=15
            )
        except Exception as e:
            print(f"Discord 發送失敗: {e}")


# 使用 Session，自動管理 Cookie
session = requests.Session()

session.headers.update({
    "User-Agent": "Bahadroid (https://www.gamer.com.tw/)"
})

# =====================================================
# 1. 登入
# =====================================================

print("=== Login ===")

resp = session.post(
    "https://api.gamer.com.tw/mobile_app/user/v3/do_login.php",
    headers={
        "Cookie": "ckAPP_VCODE=7045"
    },
    data={
        "uid": account,
        "passwd": password,
        "vcode": "7045"
    },
    timeout=20
)

print("Login Status:", resp.status_code)
print("Login Cookies:", session.cookies.get_dict())

rune = session.cookies.get("BAHARUNE")

if rune is None:
    msg = "❌ 巴哈登入失敗（密碼錯誤、兩步驟驗證或登入 API 已變更）"

    print(resp.text)

    send_discord_msg(msg)

    with open("README", "w", encoding="utf-8") as f:
        f.write(f"{datetime.now():%Y/%m/%d %H:%M:%S}\n")
        f.write(msg)

    sys.exit("Login Failed")

# =====================================================
# 2. 取得 Token
# =====================================================

print("=== Get Token ===")

token_resp = session.get(
    "https://www.gamer.com.tw/ajax/get_csrf_token.php",
    timeout=20
)

print("Token Status:", token_resp.status_code)

token = token_resp.text.strip()

print("Token:", token)

# =====================================================
# 3. 簽到
# =====================================================

print("=== Signin ===")

signin_resp = session.post(
    "https://www.gamer.com.tw/ajax/signin.php",
    data={
        "action": 1,
        "token": token
    },
    timeout=20
)

print("Status:", signin_resp.status_code)
print("Content-Type:", signin_resp.headers.get("Content-Type"))

print("========== Response ==========")
print(signin_resp.text)
print("==============================")

try:
    resp_json = signin_resp.json()

except Exception as e:
    msg = (
        "❌ 巴哈簽到失敗\n\n"
        f"HTTP Status：{signin_resp.status_code}\n"
        f"Content-Type：{signin_resp.headers.get('Content-Type')}\n"
        "API 沒有回傳 JSON，請查看 GitHub Actions Log。"
    )

    send_discord_msg(msg)

    with open("README", "w", encoding="utf-8") as f:
        f.write(f"{datetime.now():%Y/%m/%d %H:%M:%S}\n")
        f.write(msg)
        f.write("\n\n")
        f.write(signin_resp.text)

    raise

# =====================================================
# 4. 判斷結果
# =====================================================

if "error" in resp_json:
    err_msg = resp_json["error"].get("message", "未知錯誤")

    if "今天您已經簽到過了" in err_msg or "今日已簽到" in err_msg:
        discord_msg = "✅ 巴哈姆特 今日已簽到"

    else:
        discord_msg = f"❌ 巴哈簽到失敗：{err_msg}"

elif "data" in resp_json:
    days = resp_json["data"].get("days", 0)
    discord_msg = f"✅ 巴哈姆特 簽到成功（連續 {days} 天）"

else:
    discord_msg = "❌ 巴哈簽到失敗：未知回傳格式"

# =====================================================
# 5. Discord
# =====================================================

send_discord_msg(discord_msg)

# =====================================================
# 6. README
# =====================================================

with open("README", "w", encoding="utf-8") as f:
    f.write(f"{datetime.now():%Y/%m/%d %H:%M:%S}\n")
    f.write(discord_msg)
    f.write("\n\n")
    f.write(json.dumps(resp_json, ensure_ascii=False, indent=4))

print(discord_msg)
