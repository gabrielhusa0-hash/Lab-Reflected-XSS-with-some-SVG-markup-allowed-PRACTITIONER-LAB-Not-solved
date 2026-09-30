# Lab: Reflected XSS with some SVG markup allowed

## 📚 Studijní materiál
Tento repozitář slouží jako můj osobní studijní zápisník a přehled řešení laboratoří z platformy **PortSwigger Web Security Academy**. Dokumentuji zde postupy, testování filtrů a využívání netradičních tagů pro budoucí reference.

---

## 🛠️ Postup řešení laboratoře (Krok za krokem)

1. **Základní test a příprava v Burp Suite:**
   * Do hlavního vyhledávacího políčka na stránce jsem zapsal testovací payload: `<img src=1 onerror=alert(1)>` a odeslal ho.
   * V Burp Suite jsem v záložce **HTTP History** našel odpovídající požadavek (`GET /?search=...`), označil jsem text a poslal ho do **Intruderu**.

2. **Hledání povolených tagů přes Intruder (1. fáze):**
   * V Intruderu jsem vymazal původní kód a napsal tam jen prázdné zobáčky: `<>`.
   * Tyto zobáčky jsem označil a proměnil na pozici pro payload (tlačítko *Add §*), takže to vypadalo jako `<§§>`.
   * Šel jsem na oficiální [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet), zkopíroval odtamtud sadu tagů a vložil je do Intruderu jako payloady.
   * Po spuštění útoků (Attack) jsem hledal, které tagy filtr nevyblokuje, a takto jsem našel, že funguje SVG markup (např. tag `<svg>`).

3. **Hledání správného eventu / atributu v SVG (2. fáze):**
   * Vzal jsem další tag nebo strukturu z Cheat Sheetu a v Intruderu jsem zkoušel hledat konkrétní spouštěče.
   * Nalezl jsem tvar, který vypadal takto: `<svg><animatetransform=1>`.
   * Později jsem zkoumáním a testováním v Intruderu (případně skrze další odkazy na Cheat Sheetu) narazil na správnou událost **`onbegin`**, takže struktura začínala jako `<svg><animatetransform%20onbegin=1>`.
   * V Burp Intruderu jsem mezi `%20` (mezeru) a zbytek textu vložil pozici pro payload (pomocí *Add §*), abych otestoval přesné chování filtru.

4. **Sestavení finálního payloadu a exploitu:**
   * Výsledný funkční požadavek vypadal takto:
     ```http
     GET /?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E HTTP/1.1
     ```
   * Sestavil jsem finální URL adresu a vložil ji do prohlížeče (každý si zde nechá svou adresu laboratoře):
     ```text
     [https://0ab800c6032b98cb80b585d600c70003.h1-web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E](https://0ab800c6032b98cb80b585d600c70003.h1-web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E)
     ```

5. **Dokončení:**
   * Po načtení takto upravené URL se SVG animace spustila, aktivoval se `onbegin=alert(1)` a laboratoř byla úspěšně vyřešena.

---

## 🔍 Technické vysvětlení
V této laboratoři web blokoval běžné skripty, ale povoloval SVG značky. Využitím elementu `<animatetransform>` a události `onbegin` jsme obešli filtr bez použití klasického tagu `<script>`, což vedlo k úspěšnému provedení XSS.