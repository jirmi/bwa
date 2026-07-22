Vysvětli tyto HTTP hlavičky:

GET /2026/07/ukraine-is-stockpiling-home-built-interceptors-to-down-russian-ballistic-missiles-a-task-experts-call-the-champions-league/ HTTP/2
Host: [www.19fortyfive.com](https://www.19fortyfive.com)
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: cs,sk;q=0.9,en-US;q=0.8,en;q=0.7
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Referer: https://www.19fortyfive.com/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Sec-GPC: 1
Priority: u=0, i

Proč tu není hlavička Origin?

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

Jaké kompresní formáty jsou uvedené v Accept-Encoding?
`gzip, deflate, br, zstd` – jaké komprese dat prohlížeč umí rozbalit, aby server mohl poslat menší (komprimovaná) data.

Zde je přehled nástrojů pro Windows a PowerShell, kterými si můžete tyto kompresní formáty vyzkoušet:

## Grafické/CLI nástroje pro Windows

1. **7-Zip** – nejuniverzálnější zdarma nástroj. Umí gzip a deflate nativně (přes GUI i příkazovou řádku `7z.exe`). Brotli a Zstandard přímo nepodporuje ve starších verzích, ale novější buildy (7-Zip 21.07+) už mají experimentální podporu zstd.

2. **WinRAR** – umí zip/deflate, gzip jen pro rozbalování, ne pro kompresi.

3. **Samostatné CLI nástroje** (doporučuji, dají se snadno použít i z PowerShellu):
   - **gzip.exe** – součást GnuWin32 nebo lze získat přes Git for Windows (`Git\usr\bin\gzip.exe`)
   - **brotli.exe** – oficiální CLI od Google, ke stažení z GitHubu (google/brotli releases)
   - **zstd.exe** – oficiální CLI, ke stažení z GitHubu (facebook/zstd releases)

Nejpohodlnější je nainstalovat je přes **winget** nebo **Chocolatey**:

```powershell
winget install Facebook.zstd
winget install google.brotli
```

nebo

```powershell
choco install zstandard
choco install brotli
```

## Použití v PowerShellu

**Nativní podpora v PowerShellu 5+/7+** (bez externích nástrojů) – přes .NET třídy:

```powershell
# GZip komprese
$bytes = [System.IO.File]::ReadAllBytes("soubor.txt")
$output = [System.IO.File]::Create("soubor.txt.gz")
$gzipStream = New-Object System.IO.Compression.GZipStream($output, [System.IO.Compression.CompressionMode]::Compress)
$gzipStream.Write($bytes, 0, $bytes.Length)
$gzipStream.Close()
$output.Close()

# Deflate komprese (podobně)
$deflateStream = New-Object System.IO.Compression.DeflateStream($output, [System.IO.Compression.CompressionMode]::Compress)
```

Od **.NET 7 / PowerShell 7.2+** je nativně i **Brotli**:

```powershell
$output = [System.IO.File]::Create("soubor.txt.br")
$brStream = New-Object System.IO.Compression.BrotliStream($output, [System.IO.Compression.CompressionMode]::Compress)
$brStream.Write($bytes, 0, $bytes.Length)
$brStream.Close()
```

**Zstandard** nativní .NET podporu nemá – potřebujete externí `zstd.exe`:

```powershell
zstd soubor.txt -o soubor.txt.zst
zstd -d soubor.txt.zst   # dekomprese
```

Pro **brotli.exe** obdobně, pokud nechcete .NET:

```powershell
brotli.exe soubor.txt
brotli.exe -d soubor.txt.br
```

## Tip na porovnání velikostí

Pro rychlé srovnání komprese jednoho souboru všemi čtyřmi formáty:

```powershell
gzip -k soubor.txt
brotli soubor.txt
zstd soubor.txt
Compress-Archive -Path soubor.txt -DestinationPath soubor.zip   # deflate uvnitř ZIPu

Get-ChildItem soubor.* | Select-Object Name, Length
```

Tím uvidíte, jak si jednotlivé algoritmy stojí na stejných datech co do velikosti výstupu.

Ano, valná většina těchto hlaviček je nastavena prohlížečem automaticky a uživatel do nich nijak nezasahuje. Některé ale lze měnit, jiné vůbec.

## Co lze změnit ve Firefoxu (about:config)

**User-Agent** – ano, lze přepsat:
```
general.useragent.override
```
(needs vytvořit jako string; pak posílá váš vlastní UA místo výchozího). Firefox navíc od verze ~100+ zjednodušil UA (tzv. "UA freezing/reduction") kvůli fingerprintingu – verze OS se moc neliší napříč instalacemi.

**Accept-Language** – ano:
```
intl.accept_languages
```
Zde přesně nastavíte pořadí a q-hodnoty (výchozí `cs,sk;q=0.9,en-US;q=0.8,en;q=0.7` odpovídá tomu, že máte v nastavení Firefoxu přidanou češtinu a angličtinu jako záložní jazyky).

**Referer** – dá se omezit/vypnout:
```
network.http.sendRefererHeader   (0=nikdy, 1=jen kliky, 2=vždy - default)
network.http.referer.XOriginPolicy
network.http.referer.trimmingPolicy
```

**Sec-GPC** – zapíná/vypíná se přes:
```
privacy.globalprivacycontrol.enabled
```
(zapnuto typicky když máte v nastavení soukromí zaškrtnuté "Send websites a Global Privacy Control signal").

**Accept-Encoding** – **nelze** měnit přes about:config. Je hardcoded podle toho, jaké dekompresní knihovny má Firefox zabudované (gzip/deflate/br/zstd – zstd přibyl v Firefoxu relativně nedávno, cca verze 121+).

**Sec-Fetch-*, Upgrade-Insecure-Requests, Priority** – **nelze** měnit. Jsou to bezpečnostní/protokolové hlavičky vynucené specifikací (Fetch Metadata Request Headers) a prohlížeč je generuje automaticky podle kontextu požadavku (typ zdroje, navigace vs. subresource, stejný/jiný origin apod.) – manipulace by narušila jejich smysl (ochrana proti CSRF, side-channel útokům).

Pro dočasnou/jednorázovou změnu hlaviček (bez about:config) se v praxi spíš používají rozšíření jako **Header Editor** nebo přímo **DevTools → Network → Edit and Resend**.

## Rozdíly Chrome/Edge vs Firefox

Chrome a Edge jsou oba postavené na Chromiu, takže si jsou blízké, ale liší se od Firefoxu v několika věcech:

**1. User-Agent formát**
- Chrome: `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/XXX.0.0.0 Safari/537.36`
- Edge: stejné jako Chrome + navíc `Edg/XXX.0.0.0` na konci
- Firefox: `Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:XXX) Gecko/20100101 Firefox/XXX`

**2. Client Hints (Sec-CH-UA-*) – zásadní rozdíl**
Chrome a Edge navíc posílají tzv. **User-Agent Client Hints**, které Firefox vůbec nepodporuje:
```
Sec-CH-UA: "Not/A)Brand";v="8", "Chromium";v="XXX", "Microsoft Edge";v="XXX"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "Windows"
```
Toto je moderní náhrada za parsování User-Agent stringu (méně fingerprintingu, granulárnější info na vyžádání serveru přes hlavičku `Accept-CH`).

**3. Sec-GPC**
Firefox a Brave ho posílají nativně po zapnutí v nastavení. **Chrome a Edge Sec-GPC nepodporují vůbec** (bez rozšíření typu Privacy Badger/GPC extension ho neuvidíte).

**4. Accept-Encoding – zstd**
Firefox i Chrome/Edge dnes zstd podporují, ale Chrome ho přidal o něco dříve (Chrome 123+, cca začátek 2024), Firefox jej doháněl.

**5. Priority hlavička**
Chrome/Edge používají stejný formát (`Priority: u=X, i`), protože se řídí stejnou specifikací HTTP Extensible Priorities, ale hodnoty priorit se občas liší podle interní scheduling logiky prohlížeče.

**6. Sec-Fetch-* hlavičky**
Tady jsou si všechny tři prohlížeče prakticky totožné – je to standard (Fetch Metadata), který implementují stejně.

Shrnutí: pokud chcete experimentovat s hlavičkami systematicky napříč prohlížeči, doporučuji nástroj jako **Wireshark** (zachytí syrový provoz) nebo jednodušeji **DevTools → Network tab** v každém z prohlížečů a hlavičky si přímo porovnat na stejné URL.

# Popiš mi blíže standard Fetch Metadata. Kdy vznikl, kdo ho navrhl, co je jeho smyslem, jak funguje?

Standard **Fetch Metadata Request Headers** je specifikace popisující skupinu HTTP hlaviček `Sec-Fetch-*`, kterou prohlížeč automaticky přidává ke každému požadavku a která serveru sděluje kontext, **odkud, jak a proč** byl požadavek vytvořen.

## Kdo ho vytvořil a kdy

Specifikaci vede **Web Application Security Working Group** při W3C, jako editor je uveden zástupce **Google**. První veřejný pracovní návrh (First Public Working Draft) byl publikován 27. června 2019. Dokument prošel od té doby dalšími revizemi – například verze z 31. října 2023 – a stále je vedený jako W3C Working Draft, který má směřovat k plnému W3C Recommendation.

Zajímavý je vývoj samotného designu: první verze dokumentu definovala jedinou hlavičku Sec-Metadata, jejíž obsah byl strukturovaný jako slovník (dictionary). Po diskuzi s odborníky na HTTP (mj. Mark Nottingham, autor doporučení pro návrh HTTP hlaviček) se ale návrh přesunul od jedné komplexní hlavičky k sadě jednoduchých hlaviček, z nichž každá nese jen jeden token – tento přístup se lépe komprimuje pomocí HPACK algoritmu používaného v HTTP/2. Na návrhu spolupracovali i další inženýři z bezpečnostního prostředí webu (Anne van Kesteren, Artur Janc, Dan Veditz a další).

## Proč standard vznikl (smysl)

Základní motivace je popsaná přímo v abstraktu specifikace: dokument definuje sadu fetch metadata hlaviček, jejichž cílem je poskytnout serverům dostatek informací pro rozhodnutí ještě před obsloužením požadavku – rozhodnutí založené na tom, jakým způsobem byl požadavek vytvořen a v jakém kontextu bude použit.

Prakticky to znamená: dřív musel server hádat nebo se spoléhat jen na nespolehlivé signály (např. `Referer`, který lze snadno potlačit nebo zfalšovat), aby poznal, jestli požadavek přišel z legitimní navigace uživatele, nebo třeba z útočné cross-site stránky. Fetch Metadata dává serveru přímou, prohlížečem garantovanou informaci.

Konkrétní bezpečnostní cíle, které standard řeší, zahrnují:
- **CSRF** (Cross-Site Request Forgery)
- **XSSI** (cross-site script inclusion)
- **Clickjacking a reflected XSS**
- **Timing side-channel útoky** a **exfiltraci dat přes spekulativní vykonávání** (typu Spectre)

Podle doprovodného vysvětlujícího materiálu k specifikaci existují dvě typické serverové politiky postavené na těchto hlavičkách:

- **Resource Isolation Policy** – chrání proti CSRF, XSSI, timing side-channels a exfiltraci dat přes spekulativní vykonávání; na vysoké úrovni odmítá požadavky, kde Sec-Fetch-Site == 'cross-site' A ZÁROVEŇ (Sec-Fetch-Mode není 'navigate'/'nested-navigate' NEBO metoda není GET/HEAD)
- **Navigation Isolation Policy** – chrání proti clickjackingu a reflected XSS; odmítá požadavky, kde Sec-Fetch-Site == 'cross-site' A ZÁROVEŇ Sec-Fetch-Mode == 'navigate'/'nested-navigate'

## Jak funguje – jednotlivé hlavičky

**Sec-Fetch-Site** – udává vztah mezi originem, který požadavek inicioval, a originem požadovaného zdroje – tedy říká serveru, zda požadavek přichází ze stejného originu, ze stejného webu (site), z jiného webu, nebo jde o požadavek vyvolaný přímo uživatelem. Server pak na základě této informace může rozhodnout, zda požadavek povolit – požadavky ze stejného originu bývají povoleny automaticky, zatímco u požadavků z jiných originů závisí rozhodnutí na typu požadovaného zdroje nebo na dalších fetch metadata hlavičkách. Ve výchozím nastavení by nepřijaté požadavky měly být odmítnuty s kódem 403.

**Sec-Fetch-Mode** – udává režim požadavku a umožňuje serveru rozlišit mezi požadavky vzniklými navigací uživatele mezi HTML stránkami a požadavky na načtení obrázků či jiných zdrojů. Například hodnota "navigate" se použije pro požadavky nejvyšší úrovně navigace, zatímco "no-cors" se používá třeba při načítání obrázku.

**Sec-Fetch-Dest** – udává cíl (destination) požadavku, tedy kde a jak budou stažená data použita. To umožňuje serveru rozhodnout, zda požadavek obsloužit podle toho, jestli odpovídá očekávanému způsobu použití – například požadavek s destinací "audio" by měl žádat o audio data, ne o jiný typ zdroje, třeba dokument obsahující citlivé údaje uživatele.

**Sec-Fetch-User** – posílá se jen u navigací a hodnotou `?1` potvrzuje, že požadavek vznikl přímou akcí uživatele (klik, zadání adresy), nikoli automaticky skriptem.

## Praktické příklady z návrhu specifikace

Autoři přímo v GitHub explaineru uvádí typické kombinace, které ilustrují, jak kombinace hlaviček popisuje kontext požadavku:

Načítání obrázku přes tag: Sec-Fetch-Dest: image, Sec-Fetch-Mode: no-cors, Sec-Fetch-Site: cross-site. Navigace nejvyšší úrovně na stejné doméně vyvolaná kliknutím uživatele na odkaz: Sec-Fetch-Dest: document, Sec-Fetch-Mode: navigate, Sec-Fetch-Site: same-origin, Sec-Fetch-User: ?1. Navigace nejvyšší úrovně na jinou doménu vyvolaná JavaScriptem nebo meta tagem: Sec-Fetch-Dest: document, Sec-Fetch-Mode: navigate, Sec-Fetch-Site: cross-site.

Přesně to koresponduje s hlavičkami, které jsme rozebírali u vaší ukázky z 19fortyfive.com: `Sec-Fetch-Site: same-origin` + `Sec-Fetch-Mode: navigate` + `Sec-Fetch-User: ?1` odpovídá druhému příkladu výše – tedy skutečnému kliknutí uživatele na odkaz vedoucí na stejnou doménu.

## Shrnutí principu fungování

1. Prohlížeč **při každém požadavku sám** (bez možnosti zásahu ze strany webové stránky nebo JavaScriptu) vyhodnotí kontext požadavku.
2. Přidá odpovídající `Sec-Fetch-*` hlavičky.
3. Server na straně backendu tyto hlavičky **čte a rozhoduje** – např. odmítne POST požadavek, který tvrdí, že jde o `cross-site` a zároveň nejde o navigaci (typický vzorec CSRF útoku), nebo odmítne vykreslit stránku v iframe z cizí domény (ochrana proti clickjackingu).

Protože jde o bezpečnostní mechanismus vynucený samotným prohlížečem (podobně jako `Origin` hlavička u CORS), **nelze tyto hodnoty z JavaScriptu ani uživatelsky přepsat** – to je i důvod, proč jsme si u předchozích otázek řekli, že v about:config Firefoxu nejsou k nastavení.
