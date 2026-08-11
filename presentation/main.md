---
marp: true
theme: default
title: Emulacija računalnika ZX Spectrum z vpogledom v stanje sistema
author: Klemen Remec
---

<style>
header {
    font-size: 120%;
}
img[alt~="center"] {
  display: block;
  margin: 0 auto;
}
section.columns p {
    display: flex;
    align-items: center;
    justify-content: space-around;
    margin: 0;
}
section.columns br {
    display: none;
}
ul, ol {
    margin-block-start: -15px;
    margin-block-end: -10px;
}
</style>

<!-- footer: 'Klemen Remec, 17.7.2026' -->

# Emulacija računalnika ZX Spectrum z vpogledom v stanje sistema

Avtor: Klemen Remec
Mentor: izr. prof. dr. Jurij Mihelič

Fakulteta za računalništvo in informatiko, Univerza v Ljubljani

![bg right:40% w:500](./image/wikipediazxspectrum_pic1.png)

<!--
Pozdravljeni, predstavil vam bom svoje delo na diplomskem delu, v katerem bom govoril o ...
-->

---

<!-- footer: '' -->

# Povzetek

- ZX Spectrum 48K: zgodovina, zgradba
- cilj dela
- razvoj in končna implementacija
- pregled in ovrednotenje končnih rezultatov

![bg right:40% w:500](./image/wikipediazxspectrum_pic1.png)

<!--
- računalniku ZX Spectrum 48K
- ciljih diplomskega dela
- implementaciji izdelka in
- ovrednotenju končnega rezultata
-->

---

<!-- header: '' --->

# ZX Spectrum 48K

<!--
Podjetje Sinclair Research je ZX Spectrum ...
-->

---

<!-- header: '**Zgodovina**' -->

![center w:900](./image/i4cysinclairomputers_pic1.png)

<!--
... predstavilo leta 1982 kot zmogljivejši naslednik nizkocenovnih domačih računalnikov - ZX80, ZX81

Ime Spectrum je dobil po novosti: barvni grafiki
-->

---

<!-- header: '**Zgradba**' -->
<!-- _class: columns -->

![w:500](./image/zxspectrumintroduction_pic2.png)
![w:500](./image/zxspectrumintroduction_pic1.png)

<!--
Računalnik gradijo
- procesor Z80
- bralni pomnilnik oz. ROM
- bralno-pisalni pomnilnik oz. RAM
- ULA, ki predstavlja komunikacijski center sistema med procesorjem in:
    - tipkovnico
    - UHF zaslonom
    - zvočnikom
    - kasetofonom

Vse te dele bomo podrobneje pogledali pri implementaciji posameznih komponent.

Povedal bi še, da model 48K predstavlja najbolj razširjeno različico Spectruma, zato se
nadaljnja obravnava emulatorja osredotoča le na ta model.
-->

---

<!-- header: '' --->

# Cilj dela

<!--
Preden začnem govoriti o ciljih dela, naj razložim še pojem emulacija
Ta predstavlja prevod ukazov in delovanja strojne opreme gosta (v našem primeru ZX Spectruma)
v ukaze arhitekture gostitelja (sodobni računalniki)
-->

---

<!-- header: '**Cilji dela**' -->

![center w:800](./image/subspectruminterface_pic1.png)

<!--
Cilj dela je razvoj delujočega emulatorja in razhroščevalnika, ki omogoča
- ne le poganjanje programov, temveč tudi
- razhroščevanje z vpogledom v notranje stanje sistema

Slednje omogoča:
- boljše razumevanje emuliranega računalnika in
- ohranjanje znanja o njegovem delovanju
-->

---

<!-- header: '' --->

# Razvoj

<!--
Emulator sem razvil v ogrodju za razvoj večplatformnih aplikacij - Kotlin Multiplatform.

...
-->

---

<!-- header: 'Arhitektura' -->

Moduli funkcionalnosti:

- `Processor`
- `Memory`
- `ULAScreen`, `ULAKeyboard`, `ULATapeDeck`, `ULABeeper`

![bg right:55% w:650](./image/subspectruminterface_pic1.png)

<!--
...

Abstraktni model emuliranega sistema pa sem zasnoval tako, da povezuje module funkcionalnosti
- procesorja
- pomnilnika
- zaslona, tipkovnice, kasetofona oz. zvočnih kaset, in zvočnika
-->

---

<!-- header: '' -->

# Pomnilnik

<!--
Pomnilnik Spectruma je logično razdeljen na 2 dela:
- ROM, ki vsebuje nadzorni program za zagon in razne podprograme za:
    - izrisovanje slike
    - spremljanje pritiskov tipk in
    - tolmačenje BASIC programov, ter
- RAM, ki vsebuje:
    - zaslonsko datoteko
    - sistemske spremenljivke
    - prosti pomnilnik in
    - sklad
-->

---

<!-- header: '**Pomnilnik**' -->
<!-- _class: columns -->

![w:500](./image/subspectruminterface_pic3.png)
![w:500](./image/subspectruminterface_pic4.png)

<!--
Implementiran je s 64 kilobajtov velikim poljem zlogov, in funkcijami za branje in pisanje zlogov ali bitov

Podokno razhroščevalnika
- prikazuje vrednosti pomnilniških celic in označi posamezne odseke in njihove namene
- omogoča ročno nastavljanje vrednosti posameznih pomnilniških celic
-->

---

<!-- header: '' -->

# Centralna procesna enota Z80

<!--
Procesor Z80 ...
-->

---

<!-- header: '**Registri in zastavice**' -->

![center w:800](./image/subspectruminterface_pic6.png)

<!--
... vsebuje 18 8-bitnih in 4 16-bitne registre v dveh registrskih naborih, vsak vsebuje:
- akumulator (A) za 8-bitne aritmetično-logične operacije
- register zastavic (F) značilnosti zadnje izvedene operacije
- 6 splošnonamenskih registrov, ki jih lahko uporabimo posamično, ali kot registrske pare (npr. BC)

Register zastavic vsebuje:
    - 6 dokumentiranih in
    - 2 nedokumentirani zastavici - slednji se v literaturi pogosto obravnavata kot neuporabljeni, čeprav jih dejanski procesor vseeno nastavlja in uporablja

Ta dva registrska nabora (od katerih procesor uporablja le enega naenkrat) omogočata hitro menjavo izvajalnega konteksta, uporabno za npr. hiter odziv na prekinitve

6 posebnonamenskih registrov:
- programski števec
- kazalec sklada
- indeksna registra
- osvežitveni register
- register naslova strani prekinitev

-->

---

<!-- header: '**Ukazi**' -->

CPE Z80:

- 158 ukazov
- 10 tipov naslavljanja
- 3 načini delovanja prekinitev

<br/>

`00rrr110 nnnnnnnn` → `LD r,n`

![bg right:46% w:540](./image/subspectruminterface_pic5.png)

<!--
ZX Spectrum omogoča uporabo jezika BASIC (preko tolmača v ROMu), in strojnega jezika neposredno na procesorju.

Pri slednjem procesor Z80 podpira:
- 158 različnih dokumentiranih in mnogo nedokumentiranih ukazov
- 10 tipov naslavljanja
- 3 načine delovanja maskirnih in nemaskirnih prekinitev

Vsaka definicija strojnega ukaza ima določen:
- bitni vzorec, ki definira ukaz in operande
- funkcijo izvedbe ukaza
- časovno trajanje izvedbe ukaza
-->

---

<!-- header: '**Izvajanje**' -->

`Processor.step`:

- prebere ukaz
- posodobi programski števec
- ukaz izvede
- poveča števec izvedenih ukazov
- po potrebi proži prekinitve

![bg right:46% w:540](./image/subspectruminterface_pic5.png)

<!--
Delovanje emuliranega procesorja je organizirano okrog izvedbe posameznega ukaza, kar nam omogoči korakanje skozi izvajajoči-se program

Poleg korakanja skozi program razhroščevalnik podpira tudi tekoči in pospešeni tek programa, ter prekinitvene točke
-->

---

<!-- header: '' -->

# ULA

<!--
ULA deluje kot komunikacijski center sistema

Preko V/I vrat skrbi za določene operacije, ki se dotikajo:
- zaslona
- tipkovnice
- zvočnika (piskača)
- kasetofona in posledično kaset

Podpira pa tudi serijsko vodilo za poljubne druge zunanje naprave
-->

---

<!-- header: '**Zaslon**' -->
<!-- _class: columns -->

![w:450](./image/subspectruminterface_pic7.png)
![w:650](./image/subspectruminterface_pic13.png)

<!--
Zaslon se ob periodičnih prekinitvah izrisuje na podlagi sekcij RAMa:
- zaslonska slika določa aktivnost pik: papir kot ozadje ali črnilo kot ospredje
- atributi določajo barvo papirja in črnila, svetlost in utripanje

Posebnost zaslonske slike je nelinearen razpored shranjevanja vrednosti pik
-->

---

<!-- header: '**Tipkovnica**' -->

![center h:360](./image/softspectrum48_pic1.png)

<!--
Tipkovnica je razdeljena na 8 polvrstic po 5 tipk
Branje tipkovnice iz strojnega programa nam da vedeti, katere tipke izbranih polvrstic so pritisnjene
-->

---

<!-- header: '**Kasete**' -->

![center h:360](./image/howtapeloadingword_pic1.png)

<!--
Navadne zvočne kasete predstavljajo trajno shrambo uporabniških programov

Emulator implementira dekodiranje in nalaganje najpogostejših formatov zapisov kaset s programi

Nalaganje kasete je implementirano kot "hitro nalaganje", torej namesto emulacije zvočnih impulzov posnema učinek standardne rutine nalaganja podatkov, kar prinese
- hitrejše in zanesljivejše delovanje, a obstaja
- možnost, da programi s posebnimi nalagalniki in zaščitami, ki neposredno merijo dolžine impulzov, niso polno podprti
-->

---

<!-- header: '**Zvok**' -->

![center h:260](./image/zxspectrumbasicprogramming_pic3.png)

<!--
ZX Spectrum ima vgrajen en piskač, ki predstavlja enobitni zvočni izhod (v praksi osnovni C fiksne glasnosti)

Ker lahko v strojnem programu z enim bitom določamo le stanje proizvajanja zvoka (zvok ali tišina),
lahko igran ton spremenimo z različnimi frekvencami preklapljanja stanja bita
-->

---

<!-- header: '' -->

# Evalvacija

<!--
Emulator, ki je nastal v zadnjem letu in obsega več kot 70.000 vrstic kode, lahko ovrednotimo z vidikov:
- pravilnosti izvajanja programov, razvitih za ZX Spectrum, in
- uporabnosti razhroščevalnika
-->

---

<!-- header: '**Pravilnost emulacije** - Manic Miner' -->

![center w:900](./image/subspectruminterface_pic8.png)

<!--
Pravilnost emulacije smo preverili z zagonom treh izbranih programov

Preizkušeni so bili:
- popularna arkadna igra Manic Miner
...
-->

---

<!-- header: '**Pravilnost emulacije** - INES' -->

![center w:900](./image/subspectruminterface_pic9.png)

<!--
...
- slovenski urejevalnik besedil INES, ki med drugim podpira uporabo slovenskih črk
...
-->

---

<!-- header: '**Pravilnost emulacije** - Kontrabant' -->

![center w:900](./image/subspectruminterface_pic10.png)

<!--
...
- ter slovenska besedilna pustolovska igra Kontrabant

Vsi ti programi
- se pravilno naložijo iz datotek posnetkov kaset
- delujejo brez opaznih napak z delujočo sliko, zvokom in upravljanjem preko tipkovnice

Razhroščevalnik pri tem pravilno podpira:
- prikaz stanja sistema in takojšnji prikaz sprememb
- delujoče uporabniške posege v delovanje sistema
-->

---

<!-- header: '' -->
<!-- _class: columns -->

![w:450](./image/wikipediazxspectrum_pic1.png)
![w:650](./image/subspectruminterface_pic8.png)

<!--
Emulator tako ni le način poganjanja stare programske opreme,
temveč služi tudi kot učno in predstavitveno orodje za razumevanje in digitalno ohranjanje
delovanja sistema in
zanj napisanih programov


Hvala za vašo pozornost!
-->
