# 🔐 SSO Login - External Authentication Plugin for Moodle

This plugin allows single sign-on (SSO) authentication in Moodle from an external system, using AES-256-CBC encryption and HMAC-SHA256 signature to ensure data security and integrity.

---

## 🚀 Installation

1. Access Moodle as **administrator**
2. Go to **Site Administration > Notifications**
3. Moodle will detect the plugin and ask to update the database

---

## ⚙️ Configuration

After installing, access:

**Site Administration > Plugins > Local Plugins > SSO Login**

Configure the following parameters:

- 🔑 **Shared Secret Key (HMAC)**
- ⏱️ **Maximum time allowed for timestamp** (e.g. `300` seconds)
- ✅ **Enable authentication logs**

---

## 🔗 Integration with External System

### ✅ Expected parameters:

| Parameter | Description |
|----------|----------|
| `data` | JSON encrypted with AES-256-CBC + Base64 encoded |
| `sig` | SHA256 `hash_hmac` signature of the JSON payload (before encryption) |

### 📋 The Plugin:

1. **Decrypts** the data received via `data`
2. **Validates** the `sig` signature with HMAC
3. **Checks** the sending time (timestamp)
4. **Automatically authenticates** the user in Moodle
5. **Generates success or failure logs** (if enabled)

---

## 💻 Sample Code for Integration

### 🔹 PHP
```php
function redirect_to_moodle_sso($username, $shared_secret, $moodle_login_url) {
    // Generate current timestamp
    $timestamp = time();

    // Create payload with user data
    $payload = json_encode([
        'username' => $username,
        'timestamp' => $timestamp
    ]);

    // Encrypt payload with AES-256-CBC
    $iv = openssl_random_pseudo_bytes(16);
    $ciphertext = openssl_encrypt($payload, 'aes-256-cbc', $shared_secret, 0, $iv);
    $encrypted = base64_encode($ciphertext . '::' . $iv);

    // Generate HMAC signature from payload
    $sig = hash_hmac('sha256', $payload, $shared_secret);

    // Set up redirect URL
    $url = $moodle_login_url . '?data=' . urlencode($encrypted) . '&sig=' . $sig;

    // Redirect to Moodle
    return $url;    
}

require_once 'config.php'; // Define configuration
<?php
$moodle_url = 'https://localhost.com/local/ssologin/login.php';
$shared_secret = 'SECRET_KEY'; // Secret Key
$username = 'USERNAME';
$sso_url = redirect_to_moodle_sso($username, $shared_secret, $moodle_url);
?>
<a href="#" onclick="window.open('<?php echo $sso_url; ?>', '_blank')">
```

### 🔹 Python
```python
import time, json, base64, hmac, hashlib
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import urllib.parse, webbrowser

secret = b'SECRET_KEY'.ljust(32, b'\0')
username = 'jose'
timestamp = int(time.time())

payload = json.dumps({'username': username, 'timestamp': timestamp}).encode()
iv = get_random_bytes(16)

cipher = AES.new(secret, AES.MODE_CBC, iv)
padding = 16 - len(payload) % 16
payload += bytes([padding]) * padding
encrypted = base64.b64encode(cipher.encrypt(payload) + b'::' + iv).decode()

sig = hmac.new(secret, payload[:-padding], hashlib.sha256).hexdigest()

url = 'https://localhost.com/local/ssologin/login.php?data={}&sig={}'.format(
 urllib.parse.quote(encrypted), sig
)

webbrowser.open(url)
```

### 🔹 Java
```java
String secret = "SECRET_KEY";
String username = "jose";
long timestamp = System.currentTimeMillis() / 1000;

String json = "{"username":"" + username + "","timestamp":" + timestamp + "}";

// 🔐 Encrypt JSON with AES/CBC/PKCS5Padding
// 🔐 HMAC-SHA256 signature
// 🔗 Base64 encode + redirect to Moodle

// ⚠️ The implementation depends on your Java stack (e.g. BouncyCastle, Apache Commons Crypto)
```

---

### 🔒 Security Considerations

- Always use **HTTPS**
- Store the secret key securely
- Periodically review the authentication logs
- Limit the token expiration time (recommended ≤ 300s)
- Update the plugin regularly

---

### 📜 License

GNU GPLv3 - LICENSE file

Warning: Security audit is recommended before using in production

---

### 👨‍💻 Author

Developed by Richard Guedes - Cyber ​​Defense Institute (IDCiber) – [idciber.org](https://idciber.org)

Contact: contato@idciber.org

[SSO Login](https://github.com/richardg7/sso_login) by [Richard Guedes](https://www.linkedin.com/in/richard-guedes/) is licensed under [Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1)