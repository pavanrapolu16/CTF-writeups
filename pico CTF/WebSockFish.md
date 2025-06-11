## 🕸️ WebSockFish — picoCTF Write-up

**Category:** Web Exploitation  
**Challenge URL:** [WebSockFish](https://play.picoctf.org/practice/challenge/480?category=1&page=2)  
**Instance:** http://verbal-sleep.picoctf.net:60291/

---

### 🧠 Description

> Can you win in a convincing manner against this chess bot?  
> He won't go easy on you!  
>  
> Connect to: [http://verbal-sleep.picoctf.net:60291/](http://verbal-sleep.picoctf.net:60291/)

---

### 🔍 Analysis

The page uses **WebSockets** to talk to a chess engine. It sends JSON messages like:

```json
{ "type": "eval", "eval": 17 }
```

When the bot thinks it's winning, it sends back:

```
I think my game is going pretty swimmingly :)
```

But if we trick it into thinking it's losing **badly**, it might forfeit.

---

### 🛠️ Exploit Strategy (Client-side WebSocket Manipulation)

Browsers block direct WebSocket tampering due to CORS or connection errors.  
So, I used **Burp Suite** to intercept and modify WebSocket frames.

---

### 🧪 Steps (Using BurpSuite)

1. Open [http://verbal-sleep.picoctf.net:60291/](http://verbal-sleep.picoctf.net:60291/) in browser.
2. In **Burp Suite**, go to:  
   `Proxy > WebSockets > Intercept`

3. Locate the WebSocket connection to `/ws`

4. Find a frame like:
   ```json
   { "type": "eval", "eval": 17 }
   ```

5. Right-click → **"Send to Intruder"**  
   Modify the payload to:
   ```json
   { "type": "eval", "eval": -100000000000000000000000 }
   ```

6. Click **Forward** or **Send**

7. 💥 The bot responds with:

```
Huh???? How can I be losing this badly... I resign... here's your flag: picoCTF{c1i3nt_s1d3_w3b_s0ck3t5_dc1dbff7}
```

---

### 🏁 Flag

```
picoCTF{c1i3nt_s1d3_w3b_s0ck3t5_dc1dbff7}
```

---

### 📚 Learning

- WebSockets can be manipulated with tools like **BurpSuite**, **wscat**, or browser console.
- Always inspect client-server interaction in Web challenges.
- Client-side logic can be tricked — especially when the logic relies on inputs like `eval` scores.

