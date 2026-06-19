# Moderní zabezpečení webových aplikací: Technický přehled pro rok 2026

Zabezpečení webových aplikací se do roku 2026 posunulo od reaktivního patchování k **proaktivní, hluboké obraně (Defense-in-Depth)** založené na striktních standardech prohlížečů. S nástupem pokročilých útoků na renderovací jádra (Spectre, Meltdown a jejich varianty), sofistikovaného XSS (Cross-Site Scripting) a krádeží relací přes SDN (Supply Chain Attacks) je reliance na samotný aplikační kód nedostačující. 

Tento dokument slouží jako komplexní návod pro vývojáře a bezpečnostní experty k návrhu odolných webových architektur využívajících moderní standardy W3C a OWASP.

---

## 1. Bezpečnostní HTTP hlavičky: První linie obrany

Hlavičky HTTP tvoří základní kámen obrany prohlížeče. V roce 2026 již není jejich použití volitelné, ale vyžadováno moderními standardy (např. PCI DSS 4.1, ISO/IEC 27001:2025).

### Content Security Policy (CSP) level 3 a 4
CSP je kritickou obranou proti XSS a injekcím. Doba `unsafe-inline` a `unsafe-eval` je minulostí. Moderní CSP se opírá o **nonces** (jednorázové tokeny) a **hashes**.

*   **Strict-dynamic:** Povoluje spouštění skriptů, které mají platný nonce, a skripty, které tyto dynamicky generují, čímž odpadá nutnost udržovat obrovské whitelisty (které historicky selhávaly kvůli bypassům přes JSONP).
*   **Trusted Types:** Klíčová funkce pro rok 2026. Zabraňuje DOM-based XSS tím, že zakazuje vkládání řetězců do DOM sinků (např. `innerHTML`). Vyžaduje definici explicitních "policy" objektů.

**Konfigurace (Příklad pro moderní SPA):**
```http
Content-Security-Policy: script-src 'strict-dynamic' 'nonce-EXAMPLE123456' 'unsafe-inline' https:; object-src 'none'; base-uri 'self'; report-uri /api/csp-report;
```
*(Pozn.: `unsafe-inline` je ignorováno moderními prohlížeči, pokud je přítomen nonce, slouží jako fallback pro starší klienty).*

### HTTP Strict Transport Security (HSTS)
Zabraňuje útokům typu SSL stripping a Man-in-the-Middle (MitM).
```http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
*   **max-age:** 2 roky (standard 2026).
*   **preload:** Součástí HSTS preload listu prohlížečů (nelze jej zneplatnit první návštěvou přes HTTP).

### COOP, COEP a CORP: Izolace původu (Origin Isolation)
Tyto hlavičky jsou v roce 2026 povinné pro mitigaci úniků dat přes spekulativní exekuci CPU a SharedArrayBuffer.

1.  **Cross-Origin Opener Policy (COOP):** Izoluje okno prohlížeče od jiných cross-origin dokumentů. Zabraňuje útokům, kdy škodlivá stránka otevře vaši aplikaci v novém okně a přistupuje k `window.opener`.
    ```http
    Cross-Origin-Opener-Policy: same-origin
    ```
2.  **Cross-Origin Embedder Policy (COEP):** Zajišťuje, že aplikace může načítat pouze zdroje, které explicitně povolují jejich načtení pomocí hlavičky CORP nebo CORS.
    ```http
    Cross-Origin-Embedder-Policy: require-corp
    ```
3.  **Cross-Origin Resource Policy (CORP):** Definuje, kdo může načítat daný zdroj (obrázek, font, skript).
    ```http
    Cross-Origin-Resource-Policy: same-site
    ```

### Další kritické hlavičky
*   **X-Frame-Options** / **CSP `frame-ancestors`**: Obrana proti Clickjackingu. V roce 2026 se preferuje CSP: `frame-ancestors 'self'`.
*   **Permissions-Policy (dříve Feature Policy):** Restrikce přístupu k hardwarovým a softwarovým API prohlížeče (kamera, mikrofon, geolokace, USB).
    ```http
    Permissions-Policy: camera=(), microphone=(), geolocation=(self "https://trusted.example.com")
    ```
*   **X-Content-Type-Options:** `nosniff` (zabraňuje MIME-sniffing útokům).

---

## 2. Správa souborů Cookie a ochrana relací

Krádež relace (Session Hijacking) a CSRF (Cross-Site Request Forgery) patří mezi nejzávažnější hrozby. V roce 2026 je platný standard **Cookies with Independent Partitioned State (CHIPS)** a přísná pravidla třetích stran.

### Atributy Cookie pro rok 2026
Každá autentizační cookie musí obsahovat následující sadu atributů:
```http
Set-Cookie: session_id=xyz; Secure; HttpOnly; SameSite=Strict; Path=/; Partitioned; Max-Age=3600
```
*   **`Secure`**: Přenos pouze přes HTTPS.
*   **`HttpOnly`**: Zamezení přístupu z JavaScriptu (`document.cookie`), primární obrana proti krádeži relace pomocí XSS.
*   **`SameSite=Strict` (nebo `Lax`):** Zabraňuje odeslání cookie při cross-site požadavcích. `Strict` je pro autentizaci preferován, `Lax` pro navigační GET požadavky.
*   **`Partitioned` (CHIPS):** Tzv. "third-party cookie partitioning". Cookie je vázána na doménu nejvyšší úrovně (eTLD+1), ve které byla vytvořena. Zabraňuje sledování uživatelů napříč weby a snižuje riziko cross-site leaků.

### Pokročilá správa relací
*   **Device Binding & Token Binding:** Vazba JWT relace na otisk zařízení (TPM/Secure Enclave) nebo kryptografický klíč prohlížeče (WebAuthn).
*   **Rotace relací:** Při změně stavu (přihlášení, eskalace privilegií) musí být vždy vygenerováno nové ID relace a staré zneplatněno (prevence Session Fixation).

---

## 3. Strategie pro bezpečné vkládání obsahu třetích stran

Vkládání widgetů (Analytics, chatboty, platební brány) představuje masivní riziko, neboť únik u dodavatele znamená kompromitaci vaší aplikace (Supply Chain Attack). V roce 2026 se uplatňuje **Zero-Trust přístup k DOMu**.

### 1. Sandboxing pomocí iframů a atributu `sandbox`
Nikdy nevkládejte skripty třetích stran přímo do hlavního DOMu. Používejte izolované iframes s striktním sandboxem.
```html
<iframe src="https://third-party.com/widget"
        sandbox="allow-scripts allow-forms"
        csp="script-src 'self'">
</iframe>
```
Atribut `sandbox` bez `allow-same-origin` zajistí, že iframe běží v jiném původu (origin) a nemůže přistupovat k `localStorage` nebo cookies vaší hlavní domény.

### 2. Subresource Integrity (SRI)
Pokud je načítání skriptu z CDN nezbytné, SRI zaručuje, že kód nebyl cestou pozměněn.
```html
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
        crossorigin="anonymous"></script>
```

### 3. Vzdálené izolace komponent (např. Web Components a Shadow DOM)
Pro izolaci UI komponent od stylů a skriptů hostitelské stránky lze využít Shadow DOM, avšak pro plnou bezpečnostních izolaci JavaScript logiky je nutné použít Web Workers nebo port-based messaging k iframům.

---

## 4. Architektura Vrstvené Obrany (Defense-in-Depth) - Návod pro experty

Obrana nesmí spoléhat na jeden mechanismus. Pokud útočník průlomí jednu vrstvu, další jej musí zastavit.

### Vrstva 1: Síťová a Infrastrukturální
*   **WAF (Web Application Firewall) s AI detekcí:** Moderní WAF v roce 2026 nevyužívá jen regex, ale modely strojového učení, které detekují anomálie v requestech (frekvence, payload patterns, bizarní HTTP metody).
*   **DDoS Protection a Rate Limiting na Edge:** Použití CDN s těžbou PoW (Proof of Work) pro boty k eliminaci L7 útoků (HTTP Flood).
*   **Zero Trust Network Access (ZTNA):** API servery nekomunikují mezi sebou na základě IP adres, ale na základě mTLS (Mutual TLS) a krátkodobých certifikátů (SPIFFE/SPIRE).

### Vrstva 2: Aplikační Logika
*   **Framework-Driven Encoding:** Zásada "nikdy nekonstruovat HTML pomocí zřetězení řetězců". Frameworky (React, Vue, Angular) automaticky escapují kontextově (HTML, atribut, JS). Pro vyjímky (vložení raw HTML) se využívá striktní sanitizace (např. DOMPurify s Trusted Types).
*   **Input Validation (Schema-First):** Veškerá vstupní data (JSON, query parametry) musí projít validací striktně definovaným schématem (např. Zod, JSON Schema) dříž, než dosáhnou byznys logiky.
*   **CSRF Defense:** Ačkoli `SameSite=Lax/Strict`Cookies výrazně omezily CSRF, pro kritické operace se vyžaduje Double Submit Cookie nebo synchronizérní token pattern. 

### Vrstva 3: Autentizace a Autorizace
*   **Phishing-Resistant Authentication:** Zrušení hesel ve prospěch **WebAuthn / Passkeys** (FIDO2). Kombinace "něco, co uživatel má" (kryptografický klíč v zařízení) a "něco, co uživatel je" (biometrie).
*   **OAuth 2.1 / OIDC Best Practices:** Pro API autorizaci výhradně Authorization Code Flow s PKCE. Žádné Implicit nebo Password flow.

### Vrstva 4: Prezentační vrstva (Prohlížeč)
Jak detailně rozebráno v sekcích 1 a 2, zde se uplatňuje:
*   Komplexní CSP s Trusted Types.
*   COOP/COEP/CORP pro paměťovou izolaci.
*   Sandboxing iframe komponent.

### Vrstva 5: Sledování, Detekce a Reakce (Observability)
*   **Security Headers Reporting:** Využití direktiv `report-to` v CSP, COOP a dalších pro real-time monitorování pokusů o průnik. Posílání reportů na endpoints analyzované bezpečnostním týmem.
*   **Real User Monitoring (RUM) pro bezpečnost:** Sledování chování na klientské straně pro detekci anomálií (např. detekce ad-injecting malwaru na zařízení klienta pomocí kontrol integrity DOMu).

---

## Závěr

Webové prostředí roku 2026 vyžaduje holistický přístup. Aplikace postavená na moderním frameworku, která spoléhá na izolaci DOMu, ale postrádá striktní HTTP hlavičky a správnou konfiguraci cookies (např. chybějící `Partitioned` nebo `SameSite`), je stále triviálně zneužitelná. 

Klíčem k odolnosti je **omezit prostor pro útok na minimum**. Odstranění `unsafe-inline` z CSP, implementace Trusted Types a migrace na izolaci původu (COOP/COEP) jsou kroky, které pasivně eliminují celé třídy zranitelností (XSS, Clickjacking, Spekulativní útoky) bez ohledu na to, kolik chyb vývojáři do kódu zanesou. Bezpečnostní experti musí tyto standardy integrovat přímo do CI/CD pipelin a vyžadovat je jako pevnou součást definice "Done" pro každý nový feature.
