Tady je vysvětlení jednotlivých hlaviček:

**Request line**
`GET /2026/07/ukraine-is-stockpiling-home-built-interceptors.../ HTTP/2` – prohlížeč žádá o konkrétní HTML stránku (článek) protokolem HTTP/2.

**Host**
`www.19fortyfive.com` – doménové jméno serveru, na který se požadavek posílá (nutné, protože jeden server může hostit více domén).

**User-Agent**
Identifikuje prohlížeč a systém: Firefox 152 na Windows 10/11 64-bit. Servery to používají třeba pro statistiky nebo úpravu obsahu podle prohlížeče.

**Accept**
`text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8` – jaké typy obsahu prohlížeč preferuje a v jakém pořadí (q = quality/priorita). Chce hlavně HTML.

**Accept-Language**
`cs,sk;q=0.9,en-US;q=0.8,en;q=0.7` – preferované jazyky odpovědi: čeština nejvíc, pak slovenština, americká a obecná angličtina. Odpovídá nastavení prohlížeče (typicky čeština jako primární jazyk uživatele).

**Accept-Encoding**
`gzip, deflate, br, zstd` – jaké komprese dat prohlížeč umí rozbalit, aby server mohl poslat menší (komprimovaná) data.

**Connection**
`keep-alive` – TCP spojení se má po odpovědi ponechat otevřené pro další požadavky (rychlejší načítání dalších zdrojů stránky).

**Referer**
`https://www.19fortyfive.com/` – stránka, ze které uživatel na tento odkaz klikl (v tomto případě hlavní stránka webu – tedy interní navigace).

**Upgrade-Insecure-Requests: 1**
Prohlížeč signalizuje, že preferuje/zvládne HTTPS verzi zdroje, pokud by byla nabídnuta HTTP verze.

**Sec-Fetch-*** (Fetch Metadata)
- `Sec-Fetch-Dest: document` – požadavek je pro celý dokument (ne obrázek, skript apod.)
- `Sec-Fetch-Mode: navigate` – jde o navigaci (klik na odkaz / zadání URL), ne o AJAX request
- `Sec-Fetch-Site: same-origin` – požadavek jde ze stejného originu (referer i cíl jsou stejná doména)
- `Sec-Fetch-User: ?1` – požadavek byl vyvolán skutečnou akcí uživatele (klik), ne skriptem

**Sec-GPC: 1**
Global Privacy Control – prohlížeč signalizuje serveru, že uživatel nechce, aby se jeho data prodávala/sdílela (obdoba Do Not Track, ale s právní vahou v některých jurisdikcích jako Kalifornie).

**Priority: u=0, i**
Hint o prioritě požadavku (HTTP prioritization) – `u=0` znamená nejvyšší urgentnost, `i` = incremental (obsah lze zpracovávat postupně, jak přichází).

---

### Proč chybí Origin?

Hlavička `Origin` se posílá hlavně u:
1. **Cross-origin požadavků** (CORS) – fetch/XHR z JavaScriptu na jinou doménu
2. **POST/PUT požadavků** i na stejný origin (kvůli CSRF ochraně)

Tady jde ale o **prostou navigaci** (`Sec-Fetch-Mode: navigate`) metodou **GET**, kterou uživatel vyvolal kliknutím na odkaz ve stejné doméně (`Sec-Fetch-Site: same-origin`). Prohlížeče u běžné navigace přes GET hlavičku `Origin` neposílají – stačí jim `Referer`, který dává serveru podobnou informaci (odkud požadavek přišel), a `Sec-Fetch-Site` to ještě upřesňuje. `Origin` se prostě rezervuje pro situace, kde je potřeba přísněji ověřit zdroj požadavku (CORS, formuláře, API volání).
