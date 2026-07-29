---
title: JWT attacks
published: 2026-07-29
draft: false
tags: []
status: completed
platform: portswigger
---
>Note:

![image](./images/jwt-attacks/HJj_u243bl.png)

# 1. WHAT ARE JSON WEB TOKENS (JWTs)?
JSON web tokens (JWTs) are a standardized format for sending cryptographically signed JSON data between systems. They can theoretically contain any kind of data, but are most commonly used to send information ("claims") about users as part of authentication, session handling, and access control mechanisms.

>**JWT format**: header.payload.signature

**Example:** 
![image](./images/jwt-attacks/SyB0MK82be.png)

> [!NOTE]
> **JWT signature:**
>The server that issues the token typically **generates the signature by hashing the header and payload**. In some cases, they also encrypt the resulting hash. Either way, this process involves a secret signing key. This mechanism provides a way for servers to verify that none of the data within the token has been tampered(giả mạo) with since it was issued:
>- As the signature is directly derived from the rest of the token, changing a single byte of the header or payload results in a mismatched signature.
>- Without knowing the server's secret signing key, it shouldn't be possible to generate the correct signature for a given header or payload.
>
>Link to experiment: https://www.jwt.io/

> [!TIP]
> JWT & JWS & JWE:
>The JWT spec is extended by both the JSON Web Signature (JWS) and JSON Web Encryption (JWE) specifications, which define concrete ways of actually implementing JWTs.
>![image](./images/jwt-attacks/r1iEwFU3Zx.png)
>For simplicity, throughout these materials, "JWT" refers primarily to JWS tokens, although some of the vulnerabilities described may also apply to JWE tokens.
# 2. WHAT ARE JWT ATTACKS?
JWT attacks involve a user sending modified JWTs to the server in order to achieve a malicious goal. Typically, this goal is to bypass authentication and access controls by impersonating(mạo danh) another user who has already been authenticated.

# 3. IMPACT OF JWT ATTACKS
The impact of JWT attacks is usually severe. If an attacker is able to create their own valid tokens with arbitrary values, they may be able to escalate their own privileges or impersonate other users, taking full control of their accounts.

# 4. HOW VULNERABILITIES ARISE (Phát sinh)
JWT vulnerabilities typically arise due to flawed JWT handling within the application itself. The various specifications related to JWTs are relatively **flexible by design, allowing website developers to decide many implementation details for themselves**. This can result in them accidentally introducing vulnerabilities even when using battle-hardened (được kiểm chứng) libraries.

These implementation flaws usually mean that the signature of the JWT is not verified properly. This enables an attacker to tamper (can thiệp) with the values passed to the application via the token's payload. Even if the signature is robustly verified, whether it can truly be trusted relies heavily on the server's secret key remaining a secret. **If this key is leaked in some way, or can be guessed or brute-forced**, an attacker can generate a valid signature for any arbitrary token, compromising the entire mechanism.
# 5. WORKING WITH JWTs IN BURP SUITE
>**1. Viewing JWTs:**

![image](./images/jwt-attacks/HJNvxcUh-x.png)

>**2. Editing JWTs:**

![image](./images/jwt-attacks/Hkvng5I2Wg.png)

>**3. Adding a JWT signing key:**

![image](./images/jwt-attacks/BJm3-q83We.png)

# 6. EXPLOITING FLAWED JWT SIGNATURE VERIFICATION
> **Key sight:** Flow chuẩn của JWT attack
> ```
> Đăng nhập user thường (wiener/peter)
>        ↓
> Có JWT hợp lệ trong cookie session
>        ↓
>Decode JWT → thấy cấu trúc header/payload
>        ↓
>Sửa payload (sub: wiener → administrator)
>        ↓
>Gửi lại → bypass authentication
> ```

> [!CAUTION]
> Token & JWT?
>![image](./images/jwt-attacks/BJI7v38h-e.png)

> [!CAUTION]
> API & URL?
>![image](./images/jwt-attacks/SkRpv3I2Zl.png)
>![image](./images/jwt-attacks/HJg5_38nZl.png)
>![image](./images/jwt-attacks/r1WWdnU3Zx.png)


> [!IMPORTANT]
> >By design, servers don't usually store any information about the JWTs that they issue. Instead, each token is an entirely self-contained entity. Therefore, if the server doesn't verify the signature properly, there's nothing to stop an attacker from making arbitrary changes to the rest of the token.

> [!TIP]
> Technique 1: Accepting tokens with no signature
>![image](./images/jwt-attacks/rywWU98hZl.png)
>
> The server could be using naive/basic string parsing to block this
>![image](./images/jwt-attacks/BkXO8cInbg.png)

> [!NOTE]
> Note:
> Even if the token is unsigned, the payload part must still be terminated with a trailing dot.

![image](./images/jwt-attacks/SyrZWnL2bg.png)
![image](./images/jwt-attacks/SkdrW3L2be.png)

**Trường hợp:** Không verify signature/Bypass via flawed signature verification thì flow nó giống nhau
>![image](./images/jwt-attacks/B1cfV2UnWx.png)
>
>Dán cookie vào xong reload lại page.
>![image](./images/jwt-attacks/rkaJrn83Zx.png)

# 7. BRUTE-FORCING SECRET KEYS
Some signing algorithms, such as HS256 (HMAC + SHA-256), use an arbitrary (tùy ý), standalone string as the secret key. Just like a password, it's crucial that this secret can't be easily guessed or brute-forced by an attacker. Otherwise, they may be able to create JWTs with any header and payload values they like, then use the key to re-sign the token with a valid signature.

When implementing JWT applications, developers sometimes make mistakes like forgetting to change default or placeholder secrets. They may even copy and paste code snippets they find online, then forget to change a hardcoded secret that's provided as an example. In this case, it can be trivial (dễ dàng) for an attacker to brute-force a server's secret using a [wordlist of well-known secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list).

![image](./images/jwt-attacks/HyRuy8D2Ze.png)

> [!TIP]
> Tool Hashcat:
> **Hashcat** là tool crack password nhanh nhất thế giới hiện tại, chạy bằng GPU/CPU.
> ![image](./images/jwt-attacks/rJj0MUvnbe.png)
>
> **Cấu trúc:**
> ![image](./images/jwt-attacks/ryodLUv3bl.png)
> - attack_mode: 
>![image](./images/jwt-attacks/ByPjQUv3Zg.png)
> - hash_type:
> ![image](./images/jwt-attacks/ryTp78vnZg.png)
> - mask:
> ![image](./images/jwt-attacks/ryQzI8wnZl.png)
> - Các flag hữu ích:
> ![image](./images/jwt-attacks/Sks3IUD2bx.png)

> [!NOTE]
> > Dùng repo trên github thì lấy link raw để tải về. Example:
> **Cách 1: wget**
> wget https://raw.githubusercontent.com/wallarm/jwt-secrets/refs/heads/master/jwt.secrets.list
> 
> **Cách 2: curl**
> curl -O https://raw.githubusercontent.com/wallarm/jwt-secrets/refs/heads/master/jwt.secrets.list

> [!WARNING]
> Example:
>```
># Lệnh chạy
>hashcat -a 0 -m 16500 eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c jwt.secrets.list
>
># Kết quả in ra
>eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c:mysecret
>```

**Bước 1:** Brute force key
![image](./images/jwt-attacks/HJ4nC8D2bx.png)

**Bước 2:** Đổi payload
![image](./images/jwt-attacks/H1nSJvD3Ze.png)

**Bước 3:** Dùng secret key đã lấy được hoàn thiện signature.
![image](./images/jwt-attacks/rJpyOPPn-l.png)
![image](./images/jwt-attacks/ryziIwDhWl.png)
![image](./images/jwt-attacks/BkXywDvh-l.png)
![image](./images/jwt-attacks/SynlwvPnWg.png)
![image](./images/jwt-attacks/BJS_DDPhWe.png)

# 8. JWT HEADER PARAMETER IJECTIONS
> Header JWT có một số tham số cho phép client tự chỉ định key nào dùng để verify → đây là chỗ attacker khai thác.
![image](./images/jwt-attacks/rktzbdP3Ze.png)
> Dấu hiệu nhận biết sơ lược:
> ![image](./images/jwt-attacks/S1d1ctDh-g.png)


> [!TIP]
> 1. JWK (JSON Web Key) parameter:
> - Tấn công JWK Injection chỉ dùng được với thuật toán bất đối xứng (asymmetric)
> - **jwk** chỉ xuất hiện sau khi bạn bấm **Attack → Embedded JWK → Burp mới nhúng vào**. Đó chính là hành động tấn công, không phải thứ có sẵn.
> ![image](./images/jwt-attacks/S1UWDuDnZx.png)
> **Example:**
> ![image](./images/jwt-attacks/HJoKf_vhWg.png)

> [!WARNING]
> PUBLIC AND PRIVATE KEYS:
> HS256 (HMAC + SHA-256),... use a "symmetric" key. 
>![image](./images/jwt-attacks/rkhCmdD2Wx.png)
> RS256 (RSA + SHA-256),... use an "asymmetric" key pair. 
>![image](./images/jwt-attacks/rJ8SVOwh-x.png)

**Bước 1:**
![image](./images/jwt-attacks/r1X3Y_whbe.png)
![image](./images/jwt-attacks/SyuJKODhWl.png)

**Bước 2:**
![image](./images/jwt-attacks/S1aki_DhWx.png)
![image](./images/jwt-attacks/Bk769dP2Zg.png)

> [!TIP]
> 2. JKU (JSON Web Key Set URL) parameter:
> ![image](./images/jwt-attacks/BJxtkKD3-l.png)
> **JWK set:**
> Server thường lưu tất cả public key ở một chỗ gọi là JWK Set, thường public tại:
> ```
> https://example.com/.well-known/jwks.json
> ```
> ![image](./images/jwt-attacks/rJUpC_P3Wl.png)
> ![image](./images/jwt-attacks/Hyxz1Fwh-e.png)
> ![image](./images/jwt-attacks/BkqQkFvn-g.png)
> ![image](./images/jwt-attacks/ryY8dYD3Zl.png)
>
> **Example:**
>![image](./images/jwt-attacks/SJ1qodPnWg.png)

> Server để public JWK Set tại **/.well-known/jwks.json** là bình thường. Vấn đề là khi server fetch URL từ JWT mà không kiểm tra kỹ domain → attacker bypass bằng các trick URL như SSRF.

**Bước 1:**
![image](./images/jwt-attacks/SJ0nlFP2bl.png)
Lấy url và điều chỉnh body để chèn vào.
![image](./images/jwt-attacks/HJOy8KD3Zl.png)

**Bước 2:**
![image](./images/jwt-attacks/ry5ZVtDn-e.png)

> [!TIP]
> 3. KID (Key ID) parameter:
>![image](./images/jwt-attacks/rJVXYFvhWx.png)
>![image](./images/jwt-attacks/S1McNcv2Zg.png)

> [!NOTE]
> dev/null là gi?
>![image](./images/jwt-attacks/BJTHHcPhWe.png)
>![image](./images/jwt-attacks/HykaS9vhWe.png)
>![image](./images/jwt-attacks/SJX0r5Dn-l.png)
> **Chú ý:** Chuỗi rỗng "" trong đó ta sẽ chuyển thành AA== (base64)

**Bước 1:**
![image](./images/jwt-attacks/BJFm05P3Zl.png)

**Bước 2:**
![image](./images/jwt-attacks/HJC599D2bl.png)
![image](./images/jwt-attacks/S1xb09Ph-g.png)

> If the server stores its verification keys in a database, the kid header parameter is also a potential vector for SQL injection attacks.

> [!TIP]
> 4. Other parameters:
![image](./images/jwt-attacks/BJymyov3We.png)
![image](./images/jwt-attacks/B1MrkoDhbx.png)


# 9. ALGORITHM CONFUSION ATTACKS
Link: https://portswigger.net/web-security/jwt/algorithm-confusion

# 10. PREVENTING ATTACKS
Dùng thư viện JWT cập nhật và hiểu rõ cách hoạt động. Luôn xác thực signature chặt chẽ, kể cả các edge-case như thuật toán bất thường. Whitelist chặt các host trong header jku, và kiểm tra kid để tránh path traversal hay SQL injection.

Best practice nên áp dụng thêm:
- **Expiration date:** Luôn đặt thời hạn cho token, tránh token tồn tại vĩnh viễn nếu bị lộ.
- **Không truyền token qua URL:** URL dễ bị lưu trong log, browser history, hoặc Referer header — dùng Authorization header thay thế.
- **Claim aud (audience):** Ghi rõ token này chỉ dành cho service/website nào, tránh bị dùng lại trên hệ thống khác.
- **Hỗ trợ revoke token:** Server cần có cơ chế thu hồi token khi user logout hoặc khi phát hiện bất thường.