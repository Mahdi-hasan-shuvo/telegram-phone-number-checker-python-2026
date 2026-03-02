<div align="center">

<!-- BANNER -->
<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:1a1f2e,100:2d79c7&height=200&section=header&text=Telegram%20Phone%20Checker&fontSize=42&fontColor=ffffff&fontAlignY=38&desc=Check%20if%20any%20phone%20number%20is%20registered%20on%20Telegram%20%7C%20Python%20%7C%202025&descAlignY=58&descSize=16&animation=fadeIn" width="100%"/>

<!-- BADGES -->
<p align="center">
  <img src="https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Telethon-Latest-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white"/>
  <img src="https://img.shields.io/badge/Status-Working%20%E2%9C%85-27ae60?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Updated-2025-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/github/stars/Mahdi-hasan-shuvo/telegram-checkphone?style=for-the-badge&color=yellow"/>
</p>

<br/>

> 🔍 **The most complete, working Python guide to check if a phone number is registered on Telegram — updated for 2025.**  
> Covers Telethon, Pyrogram, Bellingcat tool, rate limiting, anti-ban tips & more.

<br/>

</div>

---

## 📋 Table of Contents

- [🌟 What This Project Does](#-what-this-project-does)
- [⚡ Quick Start (Easiest Method)](#-quick-start-easiest-method)
- [🛠️ Method 1 — Bellingcat CLI Tool](#️-method-1--bellingcat-cli-tool-recommended)
- [🐍 Method 2 — Direct Telethon Implementation](#-method-2--direct-telethon-implementation)
- [🔮 Method 3 — Pyrogram (Simple Syntax)](#-method-3--pyrogram-simple-syntax)
- [📡 Getting Telegram API Credentials](#-getting-telegram-api-credentials)
- [⚠️ Rate Limiting & Anti-Ban Guide](#️-rate-limiting--anti-ban-guide)
- [🔧 Common Errors & Fixes](#-common-errors--fixes)
- [📊 Method Comparison Table](#-method-comparison-table)
- [⚖️ Legal & Ethical Notice](#️-legal--ethical-notice)
- [👨‍💻 Author](#-author)

---

## 🌟 What This Project Does

This project provides **multiple working Python solutions** to answer one simple question:

> 💬 *"Is this phone number registered on Telegram?"*

### ✅ What you can find out:
| Info | Available? |
|------|------------|
| Is the number on Telegram? | ✅ Yes |
| Username (@handle) | ✅ Yes |
| Display Name | ✅ Yes |
| Is it a verified account? | ✅ Yes |
| Is it a Telegram Premium user? | ✅ Yes |
| Is it a bot account? | ✅ Yes |

### ❌ Important Reality Check:
- If a user **disabled "Find Me By Phone"** in privacy settings → returns "not found" (false negative — unfixable by design)
- SMS verification is **deprecated since Feb 2023** — codes come via Telegram app, email, or call
- Bulk checking **will trigger rate limits** and may get accounts restricted

---

## ⚡ Quick Start (Easiest Method)

```bash
# Step 1: Install
pip install telegram-phone-number-checker

# Step 2: Create .env file with your credentials
echo "API_ID=your_api_id" > .env
echo "API_HASH=your_api_hash" >> .env
echo "PHONE_NUMBER=+your_phone" >> .env

# Step 3: Check a number!
telegram-phone-number-checker --phone-numbers +1234567890
```

That's it! 🎉 You'll get results like:
```json
{
  "phone": "+1234567890",
  "registered": true,
  "username": "john_doe",
  "name": "John Doe",
  "verified": false,
  "premium": true
}
```

---

## 🛠️ Method 1 — Bellingcat CLI Tool *(Recommended)*

> ✅ Best for beginners • Actively maintained (June 2024) • No coding needed

### Installation
```bash
pip install telegram-phone-number-checker
```

### Setup `.env` file
```env
API_ID=12345678
API_HASH=abcdef1234567890abcdef1234567890
PHONE_NUMBER=+1234567890
```

### Usage Examples
```bash
# Check a single number
telegram-phone-number-checker --phone-numbers +1234567890

# Check multiple numbers at once
telegram-phone-number-checker --phone-numbers +1234567890,+9876543210

# Download profile photos too
telegram-phone-number-checker --phone-numbers +1234567890 --download-profile-photos

# Save results to a custom file
telegram-phone-number-checker --phone-numbers +1234567890 --output my_results.json
```

> ⚠️ **Warning from Bellingcat devs**: *"Do not use your personal account for automation. A fresh account from a residential IP works best."*

---

## 🐍 Method 2 — Direct Telethon Implementation

> ✅ Best for custom integrations • Maximum control • Full error handling

### Installation
```bash
pip install telethon
```

### Full Production-Ready Code

```python
import asyncio
import random
from telethon import TelegramClient, functions, types
from telethon.tl.types import InputPhoneContact
from telethon.tl.functions.contacts import ImportContactsRequest
from telethon.errors import FloodWaitError, PhoneNumberBannedError

API_ID = 'your_api_id'
API_HASH = 'your_api_hash'
PHONE = 'your_phone_number'

class TelegramPhoneChecker:
    def __init__(self, api_id, api_hash, session_name='checker'):
        self.client = TelegramClient(session_name, api_id, api_hash)

    async def connect(self):
        await self.client.connect()
        if not await self.client.is_user_authorized():
            await self.client.send_code_request(PHONE)
            code = input('Enter the code: ')
            await self.client.sign_in(PHONE, code)

    async def check_phone(self, phone_number):
        try:
            contact = InputPhoneContact(
                client_id=random.randrange(-2**63, 2**63),
                phone=phone_number,
                first_name="Check",
                last_name="User"
            )
            result = await self.client(ImportContactsRequest([contact]))

            if result.imported:
                user = result.users[0]
                return {
                    'phone': phone_number,
                    'registered': True,
                    'user_id': user.id,
                    'username': user.username or 'No username',
                    'first_name': user.first_name,
                    'last_name': user.last_name or '',
                    'premium': user.premium,
                    'verified': user.verified,
                    'bot': user.bot
                }
            else:
                return {'phone': phone_number, 'registered': False}

        except FloodWaitError as e:
            return {'phone': phone_number, 'error': f'Rate limited. Wait {e.seconds}s'}
        except PhoneNumberBannedError:
            return {'phone': phone_number, 'error': 'Phone number is banned'}
        except Exception as e:
            return {'phone': phone_number, 'error': str(e)}

    async def check_multiple(self, phone_numbers, delay=10):
        results = []
        for phone in phone_numbers:
            result = await self.check_phone(phone)
            results.append(result)
            print(f"✔ Checked {phone}: {result}")
            await asyncio.sleep(delay)  # 🔑 Required delay!
        return results

    async def disconnect(self):
        await self.client.disconnect()

async def main():
    checker = TelegramPhoneChecker(API_ID, API_HASH)
    await checker.connect()
    phones = ['+1234567890', '+9876543210']
    results = await checker.check_multiple(phones)
    await checker.disconnect()
    return results

if __name__ == '__main__':
    asyncio.run(main())
```

> 💡 **Pro Tip**: The 10-second delay is **not optional**. Skipping it will get your account rate-limited or banned.

---

## 🔮 Method 3 — Pyrogram (Simple Syntax)

> ⚠️ Development discontinued in 2024, but still functional • Cleaner syntax

```bash
pip install pyrogram tgcrypto
```

```python
from pyrogram import Client
import asyncio

api_id = 12345
api_hash = "your_api_hash"
app = Client("my_account", api_id, api_hash)

async def check_phones(phone_list):
    results = []
    async with app:
        for phone in phone_list:
            try:
                user = await app.get_users(phone)
                results.append({
                    "phone": phone,
                    "exists": True,
                    "username": user.username,
                    "name": f"{user.first_name or ''} {user.last_name or ''}".strip()
                })
            except Exception as e:
                results.append({"phone": phone, "exists": False, "error": str(e)})
            await asyncio.sleep(15)  # Must wait between checks
    return results

phones = ["+1234567890", "+9876543210"]
results = asyncio.run(check_phones(phones))
print(results)
```

---

## 📡 Getting Telegram API Credentials

You **must** have an API ID and API Hash. Here's how:

```
1. Go to 👉 https://my.telegram.org
2. Log in with your Telegram phone number
3. Click "API Development Tools"
4. Fill in:
   - App title: Phone Checker
   - Short name: checker
   - Platform: Other
5. Click "Create Application"
6. Copy your API_ID and API_HASH — save them safely!
```

> ⚠️ **Common Issues Getting API Keys:**
> - Using a VPN? Make sure it matches your phone number's country
> - Disable browser extensions (AdBlock, etc.)
> - Try incognito mode
> - VOIP numbers often don't work — use a real SIM

---

## ⚠️ Rate Limiting & Anti-Ban Guide

Telegram **actively monitors** unofficial API usage. Here's how to stay safe:

### Safe Limits
| Action | Safe Rate |
|--------|-----------|
| Contacts imported per minute | ~30 max |
| Numbers before account restriction | ~1,000 |
| Login attempts per day | ~5 per number |

### 🛡️ Anti-Ban Best Practices

```
✅ Use aged accounts (3+ months old)
✅ Use residential IPs — NOT VPNs or datacenter IPs
✅ Wait 15–30 seconds between each check
✅ Rotate accounts every 200–500 checks
✅ NEVER use your personal Telegram account
✅ Simulate normal usage before automating (join channels, send messages)
✅ Monitor FloodWaitError wait times — if they grow, slow down
```

### Account Rotation for Scale

```python
# Distribute checks across multiple accounts
accounts = [
    {'api_id': 111, 'api_hash': 'aaa', 'phone': '+1111'},
    {'api_id': 222, 'api_hash': 'bbb', 'phone': '+2222'},
    {'api_id': 333, 'api_hash': 'ccc', 'phone': '+3333'},
]
# Switch accounts every ~200 checks with 60s cooldown between switches
```

### ⚠️ Warning Signs Your Account is About to be Banned
- FloodWaitError wait times are **increasing** (minutes → hours)
- Can't join new groups
- Messages flagged as spam automatically

---

## 🔧 Common Errors & Fixes

<details>
<summary><b>❌ CheckPhoneRequest always returns True (deprecated)</b></summary>

`CheckPhoneRequest` is **deprecated** and always returns `True` — even for fake numbers.

✅ **Fix**: Use `ImportContactsRequest` instead (all code in this repo does this correctly).
</details>

<details>
<summary><b>❌ ImportContactsRequest returns empty list</b></summary>

Three possible reasons:
1. Number is **not registered** on Telegram
2. You've **hit rate limits** — check `retry_contacts` field
3. User's **privacy settings** block discovery

```python
result = await client(ImportContactsRequest([contact]))
if not result.imported:
    if contact.client_id in result.retry_contacts:
        print("⏳ Rate limited — retry later")
    else:
        print("❌ Not registered or privacy restricted")
```
</details>

<details>
<summary><b>❌ AuthKeyUnregisteredError / Session file error</b></summary>

```python
import os
session_file = 'session_name.session'
if os.path.exists(session_file):
    os.remove(session_file)
# Then re-run and re-authenticate
```
</details>

<details>
<summary><b>❌ Two-Factor Authentication (2FA) prompt</b></summary>

```python
from telethon.errors import SessionPasswordNeededError

try:
    await client.sign_in(phone, code=code)
except SessionPasswordNeededError:
    password = input('Enter your 2FA password: ')
    await client.sign_in(password=password)
```
</details>

<details>
<summary><b>❌ "Cannot get SMS code" error</b></summary>

Since **February 2023**, Telegram no longer sends SMS codes to unofficial clients.

✅ **Fix**: Your verification code will arrive via:
- 📱 The Telegram app itself
- 📞 Voice call
- 📧 Email

The `force_sms=True` parameter is **deprecated and non-functional**.
</details>

---

## 📊 Method Comparison Table

| Method | Maintenance | Ease | Best For | Setup |
|--------|-------------|------|----------|-------|
| 🏆 **Bellingcat Tool** | ✅ Active (2024) | ⭐⭐⭐⭐⭐ | Beginners, OSINT, quick checks | Minimal |
| 🐍 **Telethon** | ✅ Active | ⭐⭐⭐⭐ | Custom workflows, full control | Low |
| 🔮 **Pyrogram** | ❌ Discontinued | ⭐⭐⭐⭐⭐ | Simple projects, clean syntax | Low |
| 🏗️ **python-telegram (TDLib)** | ✅ Active | ⭐⭐⭐ | Official backing needed | High |

**My Recommendation:**
> Start with **Bellingcat's tool** for quick results → graduate to **Telethon** when you need customization.

---

## ⚖️ Legal & Ethical Notice

> This project is intended for **legitimate use cases only**, including:
> - Security research and OSINT investigations
> - Account ownership verification
> - Personal use (checking your own number)

**Telegram's Terms of Service prohibit:**
- Bulk phone checking for spam
- Automated flooding or metric manipulation
- Any form of harassment or stalking

**All users are responsible for their own compliance with applicable laws and Telegram's ToS.**

---
# Damo
<img width="2368" height="968" alt="Screenshot 2026-03-03 013132" src="https://github.com/user-attachments/assets/f6aab4aa-7473-4aba-bda9-168ccd82433f" />


## 👨‍💻 Author

<div align="center">

<img src="https://github.com/Mahdi-hasan-shuvo.png" width="100" style="border-radius:50%"/>

### Mahdi Hasan Shuvo

[![Portfolio](https://img.shields.io/badge/Portfolio-Visit-2CA5E0?style=for-the-badge&logo=google-chrome&logoColor=white)](https://mahdi-hasan-shuvo.github.io/Mahdi-hasan-shuvo/)
[![Email](https://img.shields.io/badge/Email-shuvobbhh%40gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:shuvobbhh@gmail.com)
[![Telegram](https://img.shields.io/badge/Telegram-%2B8801616397082-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/+8801616397082)

💼 **Available for freelance & paid projects** — Let's build something great together!

</div>

---

<div align="center">

## ⭐ If this helped you, please star the repo!

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:2d79c7,50:1a1f2e,100:0d1117&height=120&section=footer" width="100%"/>

**Topics:** `telegram` `python` `telethon` `pyrogram` `phone-checker` `osint` `telegram-api` `phone-number-lookup` `telegram-bot` `2025`

</div>
