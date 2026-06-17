Tyto direktivy jsou z konfigurace webového serveru **Apache HTTP Server**. Patří do sekce řízení přístupu a konfigurace adresářů.

Typicky se objeví například:

```apache
<Directory /var/www/html>
    AllowOverride All
    Require all granted
</Directory>
```

Pojďme je rozebrat.

---

## `AllowOverride All`

### K čemu slouží

`AllowOverride` říká Apache:

> "Dovolím, aby tento adresář měnil svou konfiguraci pomocí souboru `.htaccess`?"

Apache má hlavní konfiguraci:

```text
/etc/apache2/apache2.conf
/etc/httpd/httpd.conf
```

ale také může načítat lokální konfiguraci z:

```text
.htaccess
```

například:

```
/var/www/html/.htaccess
```

---

### Příklad

V konfiguraci:

```apache
<Directory /var/www/html>
    AllowOverride All
</Directory>
```

může být v:

```
/var/www/html/.htaccess
```

například:

```apache
RewriteEngine On

RewriteRule ^login$ login.php
```

Apache to použije.

---

## Co znamená `All`

`All` povoluje všechny typy přepisování konfigurace:

* `AuthConfig`
* `FileInfo`
* `Indexes`
* `Limit`
* `Options`

tedy například:

### AuthConfig

autentizace:

```apache
AuthType Basic
AuthName "Private"
Require valid-user
```

---

### FileInfo

změny URL:

```apache
RewriteEngine On
```

nebo:

```apache
AddType application/x-httpd-php .txt
```

---

### Options

například:

```apache
Options Indexes
```

povolí výpis adresáře.

---

### Limit

řízení přístupu:

```apache
Require ip 192.168.1.10
```

---

# Bezpečnostní důsledky `AllowOverride All`

Je to velmi důležitá volba.

## Riziko 1: uživatel může měnit chování webu

Pokud někdo může zapisovat do:

```
/var/www/html
```

může vytvořit:

```
.htaccess
```

a změnit konfiguraci.

Například:

```apache
AddHandler application/x-httpd-php .jpg
```

a najednou může být:

```
obrazek.jpg
```

spuštěn jako PHP.

---

## Riziko 2: vypnutí bezpečnostních omezení

Například administrátor nastaví:

```apache
Options -Indexes
```

ale `.htaccess` to změní:

```apache
Options Indexes
```

a zobrazí se:

```
/uploads/
    secret.txt
    backup.sql
```

---

## Riziko 3: obejití ochrany

Například:

```apache
Require all denied
```

může být přepsáno.

---

# `Require all granted`

Toto je z Apache 2.4.

Znamená:

> všichni klienti mají povolen přístup

Jinými slovy:

```
ALLOW EVERYONE
```

---

Příklad:

```apache
<Directory /var/www/html>
    Require all granted
</Directory>
```

znamená:

```
Apache:
"Můžeš zobrazit soubory z tohoto adresáře."
```

---

Opak:

```apache
Require all denied
```

znamená:

```
Nikdo nemá přístup.
```

---

## Omezení podle IP

Například:

```apache
Require ip 192.168.1.0/24
```

povolí jen LAN.

---

## Kombinace pravidel

Apache používá logiku:

```apache
<RequireAll>
    Require all granted
    Require not ip 10.0.0.5
</RequireAll>
```

význam:

* všichni mohou
* kromě 10.0.0.5

---

# Typická bezpečná konfigurace

Veřejný web:

```apache
<Directory /var/www/html>
    AllowOverride None
    Require all granted
</Directory>
```

Proč?

* žádné `.htaccess` změny
* rychlejší
* předvídatelnější
* menší attack surface

---

# Kdy se používá `AllowOverride All`

Například:

WordPress:

```
/var/www/html/.htaccess
```

potřebuje:

```apache
RewriteEngine On
```

takže často:

```apache
AllowOverride All
```

---

Lepší varianta:

```apache
AllowOverride FileInfo
```

protože povolíš jen:

* RewriteRule
* AddType

ale ne:

* Options
* Auth
* Access control

---

# Z pohledu bezpečnosti (např. DVWA)

U aplikací typu DVWA, starších PHP aplikací apod. často uvidíš:

```apache
AllowOverride All
Require all granted
```

protože:

* aplikace očekává `.htaccess`
* má povolený přístup z laboratoře
* není to produkční nastavení

V produkci by typicky bylo:

```apache
<Directory /var/www/html>
    AllowOverride None
    Options -Indexes
    Require all granted
</Directory>
```

tedy:

* žádné náhodné `.htaccess`
* žádný directory listing
* jasná konfigurace v Apache.
