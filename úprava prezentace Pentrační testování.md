Prezentace má velmi solidní teoretický základ a dobře vysvětluje principy webové bezpečnosti. Pro ročník 2025 a pro úroveň akademické/odborné prezentace však **chybí několik zásadních moderních témat, standardů a praktických souvislostí**.

Zde je přehled toho, co byste měli do prezentace doplnit, rozdělené podle témat, aby dávalo smysl, kam jednotlivé slidy zařadit:

### 1. Chybějící metodiky a standardy (Zásadní doplnění)
Prezentace popisuje fáze obecně, ale v praxi se pentesty neřádí "od stolu", ale podle mezinárodních standardů.
*   **OWASP (Open Worldwide Application Security Project):** U webových aplikací je naprostý základ. Zmiňte **OWASP Top 10** (seznam 10 nejčastějších rizik) a **OWASP Testing Guide** (průvodce jak testovat).
*   **PTES (Penetration Testing Execution Standard):** Standardizuje přesný postup pentestu (od předvýzkumu po report).
*   **OSSTMM:** Další metodika zaměřená více na analytické a procesní stránku.

### 2. Úprava fáze "Vymazání stop" (Kritická chyba v definici)
*   **Problém:** Prezentace uvádí fázi "Vymazání stop" jako součást etického hackingu/pentestu. To je chyba. Vymazání stop dělají *černí hackeři*. Etický hacker **nikdy** nemá za úkol mazat stopy, naopak je musí pečlivě zdokumentovat.
*   **Oprava:** Fázi "Vymazání stop" přesuňte jako poznámku pod "Typy hackerů (Černý)". U pentestu tuto fázi nahraďte fází **Post-exploitation (Následná exploatace)** – tedy co útočník může dělat *poté*, co získal přístup (např. pivotování do dalších sítí, krádež dat) a **Obnova (Cleanup)** – odstranění backdoorů a úprav po skončení testu, aby systém zůstal funkční.

### 3. Chybějící typy útoků na webové aplikace (Doplnění k SQLi a XSS)
Kromě SQLi a XSS chybí z dnešního pohledu kritické útoky, které tvoří jádro OWASP Top 10:
*   **CSRF (Cross-Site Request Forgery):** Vynucení nechtěné akce u přihlášeného uživatele. (Logicky navazuje na slide o Cookies a SOP – pokud cookie nemá `SameSite` atribut, je útok snadný).
*   **Broken Access Control / IDOR (Insecure Direct Object Reference):** Útok změnou ID v URL (např. změníte `id=123` na `id=124` a vidíte cizí fakturu).
*   **SSRF (Server-Side Request Forgery):** Velmi aktuální útok v roce 2025. Vnutíme serveru, aby komunikoval s vnitřní sítí (např. `http://localhost/admin`), čímž obejdeme firewall.
*   **Authentizační útoky:** Brute-force, Credential stuffing (podvržení ukradených hesel), zneužití chybné obnovy hesla.

### 4. Rozšíření stávajících slideů (Důležité technické detaily)
*   **U slideu SQL Injection:** Místo "šablon nebo uložených procedur" používejte profesionální termín **Prepared Statements (Parametrizované dotazy)**. Dále zmiňte **ORM (Object-Relational Mapping)** frameworky, které SQL injection dnes v podstatě znemožňují, pokud se nepoužívají surové dotazy.
*   **U slideu XSS:** Rozdělte XSS na tři typy, protože se řeší různě:
    *   *Reflected XSS* (odražené – v URL nebo error zprávě).
    *   *Stored XSS* (uložené – nejnebezpečnější, uloží se do DB a infikuje všechny uživatele, např. fórum).
    *   *DOM-based XSS* (problém je čistě v kódu prohlížeče, server ho nevidí).
*   **U slideu Cookies (Krádež relace):** Chybí zmínka o atributu **`Secure`**. Když cookie nemá `Secure`, lze ji zachytit na nešifrované WiFi síti (přesně tam, kde zmínka o HTTP, že propouští firewall). Dnes se musí používat `HttpOnly`, `Secure` a `SameSite=Strict/Lax`.

### 5. Chybějící oblasti penetračního testování (Moderní doba)
V sekci "Možné oblasti" chybí technologie, které dnes tvoří většinu firemní infrastruktury:
*   **Cloudové prostředí (AWS, Azure, GCP):** Testování špatně nastavených oprávnění (IAM), otevřených bucketů (S3), API gateway.
*   **Mobilní aplikace (iOS/Android):** Revize kódu, dekompilace, testování lokálního úložiště, komunikace s API. (Poznámka: "Testy na straně klienta" zní spíše jako desktop/sw testování).
*   **IoT (Internet of Things):** Často slabý článek, testování firmware, rozhraní.
*   **Fyzický pentest:** Přístup do budovy, lockpicking, tailgating (následování někoho za dveře). Často se dělá v kombinaci se sociálním inženýrstvím.

### 6. Právní a smluvní rámec (Etický hacking)
Úvod říká, že je to "legální a se souhlasem", ale to je v praxi složitější. Přidejte slide o:
*   **NDA (Smlouva o mlčenlivosti):** Tester nesmí zveřejnit zjištění.
*   **RoE (Rules of Engagement):** Pravidla-engagementu. Co smí tester dělat a co už ne (např. "Nemáte smazat data", "Testování DoS útoku je zakázáno", "Pentest probíhá jen v sobotu od 22:00 do 6:00").
*   **Odpovědnost:** Co se stane, když tester omylem shodí produkční server.

### 7. Automatizace vs. Manuální testování
*   Zmiňte, že nástroje jako ZAP nebo Nikto najdou jen "nízko visící plody" (low-hanging fruits). Skutečný pentester musí kombinovat automatizované skenování s **manuálním testováním** (logické chyby, složité obcházení bezpečnosti, komplexní řetězce útoků).

### 8. Závěr prezentace (Chybí obranná strategie)
Prezentace je velmi útočná. Na konci by měl být slide, který shrne, jak se firmy brání (Defense in depth):
*   **WAF (Web Application Firewall):** Filtruje HTTP požadavky.
*   **SIEM (Security Information and Event Management):** Centralizované logování a detekce anomálií.
*   **Šifrování:** HTTPS everywhere (TLS 1.3).
*   **Zero Trust architektura:** "Nikdy nevěř, vždy ověřuj" – přístup na základě neustálé autorizace.

---

### Návrh, jak to do prezentace fyzicky vložit:
1.  **Za slide "Typy penetračních testů"** vložte slide **"Standardy a metodiky (OWASP, PTES)"**.
2.  **V sekci "Fáze"** upravte bod 5 na **"Post-exploitation a obnova systému"** a do poznámky pod slide napište, že vymazání stop dělají jen útočníci (black hats).
3.  **Rozdělte slide "Útok typu XSS"** na dva – jeden popisující princip, druhý s typy (Stored, Reflected, DOM).
4.  **Za slide "XSS"** přidejte slide **"CSRF (Cross-Site Request Forgery)"** (skvěle to naváže na Cookies).
5.  **V sekci "Možné oblasti"** přidejte odrážky: **Cloudové služby**, **Mobilní aplikace**, **IoT zařízení**.
6.  **Před závěrečnou částí** (nebo na začátku u etického hackingu) přidejte slide **"Pravidla hry (Rules of Engagement a právní rámec)"**.
7.  **Na úplný konec** přidejte slide **"Komplexní obrana (Defense in Depth)"**, aby prezentace neskončila jen u popisu toho, jak věci rozbít, ale i jak je chránit.
