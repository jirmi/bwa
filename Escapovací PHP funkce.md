# Bezpečnostní důsledky PHP escapovacích funkcí

## Přehled funkcí a jejich zamýšlené použití

### `htmlspecialchars()` a `htmlentities()`
Slouží k **ochraně před XSS** (Cross-Site Scripting) — převádějí speciální HTML znaky na entity.

**Rozdíl:**
- `htmlspecialchars()` převádí pouze: `& < > " '`
- `htmlentities()` převádí *všechny* znaky s HTML ekvivalentem (včetně diakritiky apod.)

**Správné použití:**
```php
echo htmlspecialchars($input, ENT_QUOTES | ENT_HTML5, 'UTF-8');
```

**Bezpečnostní úskalí:**

| Problém | Popis |
|---|---|
| Chybějící `ENT_QUOTES` | Bez tohoto flagu se neescapují jednoduché uvozovky `'`, útok v atributech jako `<input value='$x'>` je stále možný |
| Špatné kódování | Bez explicitního `'UTF-8'` hrozí bypassy přes vícebajtové znaky (UTF-7 útoky ve starších prohlížečích) |
| Špatný kontext | Funkce chrání pouze v **HTML kontextu**. V JS, CSS nebo URL kontextu nestačí — `<script>var x = "<?= htmlspecialchars($x) ?>"</script>` je stále zranitelné |
| Dvojité escapování | Při nesprávném použití vede k nefunkčnímu zobrazení (`&amp;amp;`) |

---

### `addslashes()`
Přidává zpětné lomítko před `' " \ NULL`.

**Nikdy nepoužívat pro SQL!** Tato funkce **není náhrada za prepared statements** ani PDO escapování.

**Proč je nebezpečná:**

```php
// ŠPATNĚ — stále zranitelné
$query = "SELECT * FROM users WHERE name = '" . addslashes($input) . "'";
```

- Závisí na kódování databáze — v GBK/Big5 lze lomítko "pohltit" vícebajtovým znakem (tzv. *multibyte bypass*)
- Nechrání před SQL injection ve všech kontextech (čísla, identifikátory, `LIKE` wildcards `%` `_`)
- Původně navržena pro Magic Quotes (odstraněno v PHP 5.4) — nemá spolehlivé bezpečnostní využití

---

### `mysqli_real_escape_string()` / `PDO::quote()`

Lepší než `addslashes()`, ale stále **nedostatečné** jako primární ochrana.

```php
// Stále špatně — integer kontext, escapování nepomůže
$query = "SELECT * FROM users WHERE id = " . mysqli_real_escape_string($conn, $id);
// $id = "1 OR 1=1" projde bez uvozovek
```

**Jediná správná ochrana před SQL Injection:**
```php
// Prepared statements
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ? AND name = ?");
$stmt->execute([$id, $name]);
```

---

### `escapeshellcmd()` a `escapeshellarg()`

Slouží k ochraně při předávání dat do shellu (`exec()`, `shell_exec()`, `system()` atd.).

**Rozdíl:**

| Funkce | Co dělá | Použití |
|---|---|---|
| `escapeshellarg()` | Obalí celý argument do uvozovek, escapuje speciální znaky | Jednotlivý argument |
| `escapeshellcmd()` | Escapuje speciální znaky shellu v celém příkazu | Celý příkaz |

**Bezpečnostní úskalí `escapeshellcmd()`:**

```php
// Zdánlivě bezpečné, ale...
$file = "file.txt -ofile.php"; // option injection
system(escapeshellcmd("convert " . $file));
// escapeshellcmd NEOCHRÁNÍ před přidáváním parametrů/přepínačů!
```

- **Nechrání před option/argument injection** — útočník může přidat přepínače jako `--output=evil.php`
- Vhodná pouze tehdy, když skutečně sestavujete celý příkaz dynamicky (vzácné a riskantní)

**Správný postup:**
```php
// escapeshellarg() pro každý argument zvlášť
$file = escapeshellarg($userInput);
system("convert input.jpg " . $file);

// Ještě lépe — allowlist
if (!preg_match('/^[a-zA-Z0-9_\-]+$/', $userInput)) {
    throw new InvalidArgumentException();
}
```

---

## Souhrnná tabulka: Co chrání co

| Funkce | XSS | SQL Injection | Command Injection | Poznámka |
|---|:---:|:---:|:---:|---|
| `htmlspecialchars()` | ✅ (s ENT_QUOTES) | ❌ | ❌ | Pouze HTML kontext |
| `htmlentities()` | ✅ (s ENT_QUOTES) | ❌ | ❌ | Agresivnější, stejná omezení |
| `addslashes()` | ❌ | ⚠️ Nestačí | ❌ | Multibyte bypass, nedůvěryhodná |
| `mysqli_real_escape_string()` | ❌ | ⚠️ Nestačí | ❌ | Jen v uvozovkovém kontextu |
| `escapeshellarg()` | ❌ | ❌ | ✅ | Preferovaná pro argumenty |
| `escapeshellcmd()` | ❌ | ❌ | ⚠️ Nestačí | Nechrání před option injection |
| **Prepared statements** | ❌ | ✅ | ❌ | Správné řešení pro SQL |

---

## Klíčové principy

1. **Escapování je kontextové** — funkce platná v HTML kontextu neplatí v SQL, URL nebo JS kontextu
2. **Preferujte parametrizaci před escapováním** — prepared statements jsou spolehlivější než jakékoliv escapování pro SQL
3. **Allowlist > Denylist** — povolte jen povolené znaky, místo blokování zakázaných
4. **Nikdy nevolat shell zbytečně** — `exec()` a spol. by měly být krajním řešením
5. **Defense in depth** — CSP hlavičky, WAF, validace vstupu na více vrstvách — escapování je jen jedna vrstva
