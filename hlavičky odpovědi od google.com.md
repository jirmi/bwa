# Vysvětli mi tyto hlavičky HTTP odpovědi:

```
HTTP/3 200 
content-type: text/html; charset=UTF-8
date: Sun, 14 Jun 2026 09:02:43 GMT
expires: -1
cache-control: private, max-age=0
strict-transport-security: max-age=31536000
content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-DbYF57A2J5Ie7gK2qX2alw' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
cross-origin-opener-policy: same-origin-allow-popups; report-to="gws"
report-to: {"group":"gws","max_age":2592000,"endpoints":[{"url":"https://csp.withgoogle.com/csp/report-to/gws/other"}]}
accept-ch: Sec-CH-Prefers-Color-Scheme
content-encoding: br
server: gws
content-length: 65988
x-xss-protection: 0
x-frame-options: SAMEORIGIN
set-cookie: __Secure-STRP=ANmZwa0oYNPo0YI8W1uVE8LC5K2ehFNiADiS0r-uiyo3Ka7lDs2swK7Mjn6h3r0aYrHAeKylGna3K4ZBZEfViVubxvM_AeVvwA9D; expires=Sun, 14-Jun-2026 09:07:43 GMT; path=/; domain=.google.com; Secure; SameSite=strict
set-cookie: SEARCH_SAMESITE=CgQIiqEB; expires=Fri, 11-Dec-2026 09:02:43 GMT; path=/; domain=.google.com; SameSite=strict
set-cookie: OTZ=; expires=Mon, 01-Jan-1990 00:00:00 GMT; path=/; domain=www.google.com
set-cookie: OTZ=; expires=Mon, 01-Jan-1990 00:00:00 GMT; path=/; domain=.www.google.com
set-cookie: OTZ=; expires=Mon, 01-Jan-1990 00:00:00 GMT; path=/; domain=google.com
set-cookie: OTZ=; expires=Mon, 01-Jan-1990 00:00:00 GMT; path=/; domain=.google.com
set-cookie: SIDCC=AKEyXzUmwEPvg4aRXngmqQ5i8tnKqoST2vDyynbVT9cVykqnJX4tyyX6WlCh0sxKCh_Dea_LUOg; expires=Mon, 14-Jun-2027 09:02:43 GMT; path=/; domain=.google.com; priority=high
set-cookie: __Secure-1PSIDCC=AKEyXzU-7KoN27nfd7Pg3ekeUrmptqFCy1v_cZZ3JHsWquqwqfwzh_ZlezBqtH_TzJ5dGDy2OsY; expires=Mon, 14-Jun-2027 09:02:43 GMT; path=/; domain=.google.com; Secure; HttpOnly; priority=high
set-cookie: __Secure-3PSIDCC=AKEyXzUsj2S8gPz0wMEoi8FVqa03PPh_KyHSJ6LdkweMTMOesjLSoFrs0mU7_8G3LGSjhi3RbIo0; expires=Mon, 14-Jun-2027 09:02:43 GMT; path=/; domain=.google.com; Secure; HttpOnly; priority=high; SameSite=none
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
```

Tato HTTP odpověď vypadá jako odpověď z Google (server `gws`) při načtení stránky přes HTTP/3. Projdu hlavičky postupně a vysvětlím, co znamenají a k čemu slouží.

---

## Stavová řádka

```
HTTP/3 200
```

* **HTTP/3** – třetí generace HTTP protokolu.

  * běží nad **QUIC** protokolem místo TCP
  * používá šifrování TLS 1.3 přímo v QUIC
  * zlepšuje latenci a odolnost proti změně sítě

* **200** – HTTP stavový kód:

  * požadavek byl úspěšný
  * server posílá požadovaný obsah

---

## Typ odpovědi

```
content-type: text/html; charset=UTF-8
```

Určuje obsah těla odpovědi.

Zde:

* `text/html` → prohlížeč má očekávat HTML stránku
* `charset=UTF-8` → kódování znaků

Například:

```html
<html>
  <body>Hello</body>
</html>
```

---

## Datum

```
date: Sun, 14 Jun 2026 09:02:43 GMT
```

Čas vytvoření odpovědi serverem.

Používá se například:

* pro cache
* synchronizaci
* logování

---

# Cache hlavičky

## Expires

```
expires: -1
```

Historicky znamená:

> tento obsah nemá být považován za dlouhodobě platný

Hodnota `-1` je neformální způsob některých serverů říct:

"neukládej do klasické cache".

---

## Cache-Control

```
cache-control: private, max-age=0
```

Důležitá hlavička.

### `private`

Odpověď smí ukládat pouze:

* lokální cache prohlížeče

nesmí ji ukládat:

* proxy cache
* CDN cache

---

### `max-age=0`

Říká:

"považuj obsah za okamžitě prošlý".

Prohlížeč může mít kopii, ale musí si ověřit aktuálnost.

Typické pro:

* přihlášené stránky
* personalizovaný obsah

---

# HTTPS bezpečnost

## Strict Transport Security

```
strict-transport-security: max-age=31536000
```

HSTS.

Říká prohlížeči:

> příštích 365 dní komunikuj s touto doménou pouze přes HTTPS

31536000 sekund = 1 rok.

Po návštěvě:

```
http://google.com
```

prohlížeč automaticky udělá:

```
https://google.com
```

Chrání proti:

* SSL stripping útokům
* downgrade útokům

---

# Content Security Policy

```
content-security-policy-report-only:
object-src 'none';
base-uri 'self';
script-src ...
```

Jedna z nejzajímavějších hlaviček.

CSP chrání proti:

* XSS
* načítání škodlivých skriptů
* injekci HTML

Ale zde je:

```
report-only
```

To znamená:

> pravidla se pouze testují, ale neblokují

---

Rozbor:

## object-src

```
object-src 'none'
```

Zakazuje:

```html
<object>
<embed>
<applet>
```

Tedy staré pluginy.

---

## base-uri

```
base-uri 'self'
```

Zakazuje změnu základní URL stránky.

Chrání například proti:

```html
<base href="https://evil.com">
```

---

## script-src

```
script-src 'nonce-DbYF57A2J5Ie7gK2qX2alw'
```

Povoluje skripty pouze s konkrétním nonce.

Například:

Povolené:

```html
<script nonce="DbYF57A2J5Ie7gK2qX2alw">
 alert(1)
</script>
```

Zakázané:

```html
<script>
 alert(1)
</script>
```

To je velmi běžná ochrana proti XSS.

---

Další:

```
'strict-dynamic'
```

znamená:

Pokud důvěryhodný skript načte další skript:

```javascript
loadScript("x.js")
```

tak ten nový může být povolen.

---

```
'unsafe-inline'
```

povoluje inline skripty.

To je obvykle slabina CSP, ale Google to často kombinuje s nonce.

---

```
'unsafe-eval'
```

umožní:

```javascript
eval()
new Function()
```

Je to rizikové, ale některé frameworky to potřebují.

---

```
report-uri
```

kam se posílají CSP porušení:

```
https://csp.withgoogle.com/...
```

Například:

uživatel spustí:

```html
<script>alert(1)</script>
```

CSP může poslat report.

---

# Cross-Origin ochrana

## COOP

```
cross-origin-opener-policy:
same-origin-allow-popups
```

Chrání izolaci oken.

Normálně:

stránka A:

```
window.open(evil.com)
```

může mít vztah s novým oknem.

COOP omezuje komunikaci.

`same-origin-allow-popups`:

* izoluje hlavní stránku
* ale dovoluje některé popup scénáře

Používá se proti:

* Spectre útokům
* úniku dat přes `window.opener`

---

# Reporting API

```
report-to:
{
 "group":"gws",
 "max_age":2592000
}
```

Definuje skupinu pro hlášení.

Používá se pro:

* CSP chyby
* bezpečnostní reporty
* síťové chyby

---

# Client Hints

```
accept-ch:
Sec-CH-Prefers-Color-Scheme
```

Server říká:

"posílej mi informaci o preferovaném barevném režimu".

Například:

uživatel má:

```
dark mode
```

prohlížeč může poslat:

```
Sec-CH-Prefers-Color-Scheme: dark
```

---

# Komprese

```
content-encoding: br
```

Brotli komprese.

Místo:

100 kB HTML

pošle:

30 kB komprimovaných dat

Podporují ji moderní prohlížeče.

---

# Server

```
server: gws
```

Google Web Server.

Neříká přesně:

* Apache
* nginx

protože Google používá vlastní infrastrukturu.

---

# Velikost odpovědi

```
content-length: 65988
```

Tělo odpovědi má:

65 988 bajtů

(po kompresi)

---

# Starší XSS ochrana

```
x-xss-protection: 0
```

Vypíná starý XSS auditor.

Historicky:

```
1; mode=block
```

zapínalo browser XSS filtr.

Dnes se nepoužívá, protože:

* byl nespolehlivý
* někdy způsobil nové chyby

CSP ho nahradil.

---

# Clickjacking ochrana

```
x-frame-options: SAMEORIGIN
```

Zakazuje vložení stránky do iframe z jiné domény.

Zakáže:

```html
<iframe src="https://google.com">
```

na:

```
evil.com
```

Chrání proti:

* clickjackingu

---

# Cookies

Například:

```
set-cookie:
__Secure-STRP=...
```

Server nastavuje cookies.

---

## Secure

```
Secure
```

Cookie jde pouze přes HTTPS.

---

## HttpOnly

Například:

```
HttpOnly
```

JavaScript ji nevidí:

```javascript
document.cookie
```

ji nevrátí.

Chrání proti krádeži session přes XSS.

---

## SameSite

Například:

```
SameSite=strict
```

Cookie se neposílá při cross-site požadavcích.

Chrání proti:

* CSRF

---

Příklad:

Uživatel je přihlášen na:

```
bank.cz
```

útočník:

```
evil.cz
```

zkusí:

```html
<form action="bank.cz/send">
```

Cookie se nepřidá.

---

## Domain

```
domain=.google.com
```

Cookie platí pro:

```
www.google.com
mail.google.com
docs.google.com
```

---

## Path

```
path=/
```

platí pro celou doménu.

---

## Expires

Například:

```
expires=Mon, 14-Jun-2027
```

Cookie životnost.

---

# Speciální cookies Google

Například:

```
SIDCC
__Secure-1PSIDCC
__Secure-3PSIDCC
```

Jsou interní bezpečnostní cookies.

Používají se pro:

* ochranu účtu
* detekci rizik
* synchronizaci stavu

---

# HTTP/3 podpora

```
alt-svc:
h3=":443"; ma=2592000
```

Říká:

> tento server podporuje HTTP/3 na portu 443

`ma=2592000`

= cacheuj tuto informaci 30 dní.

---

## Bezpečnostní profil této odpovědi

Z pohledu bezpečnosti má tato odpověď:

| Mechanismus          | Stav                     |
| -------------------- | ------------------------ |
| HTTPS                | ✅                        |
| HSTS                 | ✅                        |
| CSP                  | ✅ (report-only)          |
| XSS ochrana          | CSP místo starého filtru |
| Clickjacking ochrana | ✅                        |
| CSRF ochrana cookies | ✅ SameSite               |
| Cookie Secure        | ✅                        |
| HttpOnly             | ✅                        |
| HTTP/3               | ✅                        |

Je to typická moderní odpověď velkého webu: kombinuje TLS, CSP, bezpečné cookies a izolaci původu.
