### Laboratorijska vaja: Moderni koncepti AAA in storitve v oblaku (aaS)

### 1. Uvod in cilji vaje

Tradicionalna računalniška omrežja so delovala po načelu "grajskega zidu": vse, kar je bilo znotraj lokalne mreže (LAN), je bilo varno, vse zunaj pa nevarno. V današnjem svetu dela na daljavo in storitev v oblaku ta model ne deluje več. Prešli smo na model **Zero Trust (Nihče ni vnaprej vreden zaupanja)**. 

V tej vaji bomo spoznali ključni varnostni steber **AAA** in kako ga s pomočjo platforme **Cloudflare Zero Trust** uporabljamo kot storitev v oblaku (**aaS - as-a-Service**), ne da bi za to morali napisati eno vrstico programske kode. 

### Cilji vaje:

* Razumeti razliko med **Authentication, Authorization in Accounting (AAA)**.
* Spoznati koncepte **SaaS, IDaaS in SECaaS** v praksi.
* Brez programiranja zaščititi spletno aplikacijo pred celotnim internetom.

### 2. Teoretična osnova: Kaj je AAA in kaj so te "aaS" kratico?

Da si bomo lažje predstavljali, si poglejmo analogijo z **nočnim klubom**. 

### Koncept AAA:

* **Authentication (Avtentikacija) – Kdo si?** 

  * *Analogija:* Varnostnik na vhodu vas prosi za osebno izkaznico, da preveri vašo identiteto.
  * *V računalništvu:* Vnos uporabniškega imena in gesla ali prijava prek Google/GitHub računa.
* **Authorization (Avtorizacija) – Kaj smeš delati?** 

  * *Analogija:* Ko varnostnik ve, kdo ste, preveri VIP seznam. Če ste na seznamu, vas spusti v VIP ložo, sicer ostanete v navadnem delu kluba.
  * *V računalništvu:* Pravilo, ki določa, da lahko študenti z e-pošto @student.uni-lj.si dostopajo do ocen, ne morejo pa jih spreminjati (to lahko počnejo le profesorji).
* **Accounting (Revizija / Sledljivost) – Kaj si počel?** 

  * *Analogija:* Varnostnik si v zvezek zapisuje uro vašega prihoda in morebitne kršitve kršenja reda.
  * *V računalništvu:* Dnevnik logov (zgodovina), kjer natančno piše, kdo se je prijavil, kdaj in katere strani je obiskal.

### Kako se to povezuje z modeli "aaS" (Kot storitev)?

Včasih so podjetja za vsako od teh funkcij kupovala drago strojno opremo (strežnike). Danes vse to najamemo v oblaku: 

```
+-----------------------------------------------------------------+
|                      TRADICIONALNI DOSTOP                       |
|  Uporabnik ----> [ Lokalni požarni zid ] ----> [ Lokalna baza ] |
+-----------------------------------------------------------------+

+-----------------------------------------------------------------+
|                    MODERNI OBLAČNI DOSTOP                       |
| Uporabnik ----> [ SECaaS (Cloudflare) ] ----> [ IDaaS (Google) ]|
+-----------------------------------------------------------------+
```

* **SaaS (Software-as-a-Service):** Spletna aplikacija ali portal, ki ga želimo zaščititi.
* **IDaaS (Identity-as-a-Service):** Storitev, ki preverja, kdo smo (npr. ko kliknete "Prijavi se z Google računom", Google nastopa kot IDaaS).
* **SECaaS (Security-as-a-Service):** Varnostne rešitve v oblaku. Cloudflare prestreže ves promet in blokira nepooblaščene uporabnike, preden sploh pridejo do naše spletne strani.

### 3. Arhitektura vaje (Kako poteka komunikacija)

Preden začnemo s klikanjem, si poglejmo spodnjo skico. Naša spletna stran bo javno dostopna na internetu. Cloudflare bo deloval kot "ščit" oz. posrednik (Proxy). 

```
                  +--------------------------+
                  |    Nepooblaščen obisk    |
                  +--------------------------+
                               |
                               v
+------------+      +--------------------+      +--------------------+
|  Študent   | ---> |  CLOUDFLARE EDGE   | <--- |  Heker / Napadalec |
| (Uporabnik)|      |  (Požarni zid &    |      |                    |
+------------+      |   Avtorizacija)    |      +--------------------+
      |             +--------------------+
      |                       |
      v                       v
+------------+      +--------------------+
|   IDaaS    |      |  SPLETNA STRAN     |
|  (Google)  |      |  (Cilj vaje)       |
+------------+      +--------------------+
```

1. Uporabnik želi obiskati spletno stran.
2. Cloudflare prestreže zahtevo in reče: *"Stoj! Najprej se izkaži."*
3. Uporabnik se overi prek zunanje storitve (**IDaaS - Google**).
4. Cloudflare preveri pravila (**Authorization**). Če študent ustreza pogojem, ga spusti naprej do spletne strani.
5. Vse skupaj se zapiše v dnevnik (**Accounting**).

### 4. Navodila za izvedbo vaje (Korak za korakom)

### Korak 1: Spoznavanje aplikacije, ki jo varujemo

1. Profesor vam bo dodelil URL naslov testne spletne strani (npr. http://student-vaja.com ali vašo unikatno poddomeno).
2. Odprite to stran v brskalniku. Ugotovili boste, da se stran naloži takoj in da nanjo lahko pride kdorkoli na svetu.
3. *Vprašanje za razmislek:* Če bi bila na tej strani objavljena izpitna vprašanja, bi bila trenutno varna? (Odgovor: Ne).

### Korak 2: Registracija in vstop v varnostni oblak (SECaaS)

1. Pojdite na spletno stran https://dash.cloudflare.com/ in se registrirajte za brezplačen račun.
2. Po uspešni prijavi v levem meniju poiščite in kliknite na zavihek **Zero Trust**.
3. Cloudflare vas bo prosil, da si izberete unikatno ime vaše organizacije (npr. matic-uni-vaja). Vpišite ga in potrdite.

### Korak 3: Nastavitev avtentikacije (Authentication -> IDaaS)

Sedaj bomo določili, KAKO se bodo uporabniki sploh lahko prijavili. Namesto ustvarjanja lastnih baz podatkov, bomo uporabili Google. 

1. V nadzorni plošči Zero Trust kliknite na **Settings** (Nastavitve) v spodnjem levem kotu.
2. Izberite zavihek **Authentication** (Avtentikacija).
3. Pod delom *Login methods* kliknite gumb **Add new** (Dodaj novo).
4. Izberite ponudnika **Google** (ali drug ponudnik, ki vam ga določi profesor).
5. Za namene te vaje bomo uporabili Cloudov privzeti testni ključ (kliknite *Save* brez spreminjanja zapletenih nastavitev API ključev, če je na voljo možnost "Test mode", sicer sledite navodilom na zaslonu za vpis začasnih podatkov).

*Spoznanje:* Pravkar ste integrirali **Identity-as-a-Service**. Brez ene vrstice programske kode ima vaša bodoča stran možnost Google prijave! 

### Korak 4: Ustvarjanje pravila za dostop (Authorization)

Zdaj bomo postavili našega "varnostnika pred vrata" in določili VIP seznam. 

1. V levem meniju kliknite na **Access** -> **Applications** (Aplikacije).
2. Kliknite gumb **Add an application** (Dodaj aplikacijo) in izberite možnost **Self-hosted**.
3. Izpolnite osnovne podatke: 

  * **Application name:** Zaščiteni študentski portal
  * **Application URL:** Vpišite URL naslov ali domeno, ki vam jo je v 1. koraku dodelil profesor.
4. Kliknite gumb **Next** (Naprej), da pridete do nastavitve pravil (**Policies**).
5. Nastavite pravilo: 

  * **Policy name:** Dovoljeno samo študentom
  * **Action:** Allow (Dovoli)
  * Pod delom **Configure a rule** (Nastavi pravilo) izberite: 

    * *Selector:* Emails ending in (E-pošta, ki se konča z)
    * *Value:* @student.uni-lj.si (ali vaš osebni e-poštni naslov za namene testiranja).
6. Kliknite **Next** in na dnu strani potrdite z gumbom **Add application**.

*Spoznanje:* Nastavili ste pravilo **avtorizacije**. Cloudflare bo sedaj preveril vsakega obiskovalca. 

### 5. Testiranje in revizija (Accounting)

### Korak 5: Preverjanje delovanja v praksi

1. Odprite nov zavihek v brskalniku v **brezhibnem načinu (Incognito mode)**.
2. Ponovno obiščite URL naslov vaše spletne strani iz 1. koraka.
3. **Kaj se zgodi?** Stran se ne naloži neposredno! Pojavi se elegantno prijavno okno Cloudflare, ki zahteva vašo identiteto.
4. Kliknite na gumb za prijavo z Googlom.
5. Poskusite se prijaviti z napačnim e-poštnim naslovom (ki nima končnice, ki ste jo določili v pravilu). 

  * *Rezultat:* Cloudflare vas zavrne z obvestilom *"Access Denied"*.
6. Sedaj poskusite s pravim e-poštnim naslovom. 

  * *Rezultat:* Uspešno ste preusmerjeni na vašo spletno stran.

### Korak 6: Sledljivost dogodkov (Accounting)

Kot administratorji moramo vedeti, kdo je poskušal vstopiti v naš sistem. 

1. Vrnite se v Cloudflare Zero Trust nadzorno ploščo, kjer ste prijavljeni kot skrbnik.
2. V levem meniju poiščite zavihek **Logs** (Dnevniki) -> **Access**.
3. Pred vami se bo izrisala tabela dogodkov.
4. Poiščite svoja zadnja dva poskusa prijave.

```
+---------------------+-------------------+----------+--------+
| ČAS POSKUSA         | UPORABNIK (EMAIL) | IP NASLOV| UKREP  |
+---------------------+-------------------+----------+--------+
| 03.11.2026 14:10:22 | heker@gmail.com   | 193.2... | BLOCKED|
| 03.11.2026 14:12:05 | jan@student.uni...| 84.22... | ALLOWED|
+---------------------+-------------------+----------+--------+
```

*Spoznanje:* Brez zapletenih baz podatkov ali branja tekstovnih JSON/XML datotek imate popoln grafični pregled nad tem, kdo, kdaj in od kod je dostopal do vaše aplikacije. To je **Accounting**. 

## 6. Zaključna vprašanja za poročilo o vaji

Študenti, na spodnja vprašanja odgovorite z lastnimi besedami na podlagi opravljenih korakov in opažanj med vajo.

### Glavna vprašanja (Ključni koncepti)

1. S svojimi besedami opiši razliko med Avtentikacijo in Avtorizacijo na primeru te vaje.
2. Katero storitev v tej vaji smo prepoznali kot **IDaaS** in katero kot **SECaaS**?
3. Zakaj je model Zero Trust varnejši od klasičnega gesla, ki bi ga preprosto programirali neposredno na spletno stran?

### Dodatna vprašanja za razmislek

#### Sklop A: Delovanje AAA v praksi
4. Ko ste v nadzorni plošči Zero Trust odprli zavihek "Logs" (Dnevniki), kateri del koncepta AAA ste s tem pregledovali in zakaj je ta del pomemben za varnostnega inženirja?
5. Predstavljajte si, da napadalec pozna vaše pravilno študentsko geslo, vendar poskuša do strani dostopati iz države, ki ste jo v požarnem zidu blokirali. Kateri del AAA sistema ga bo ustavil – avtentikacija ali avtorizacija? Utemeljite odgovor.
6. Zakaj je beleženje dogodkov (Accounting) pomembno tudi takrat, ko so vsi poskusi prijav uspešni in v sistemu ni zaznanih nobenih napadov?

#### Sklop B: Razumevanje modelov "as-a-Service"
7. Na vajah niste programirali lastnega obrazca za registracijo, prav tako niste ustvarili baze podatkov za shranjevanje študentskih gesel. Pojasnite, kako ste ta problem rešili z uporabo storitev v oblaku.
8. Pojasnite razliko med ponudnikom identitete (Identity Provider - npr. Google) in ponudnikom varnostne platforme (npr. Cloudflare). Kako ti dve vlogi sodelujeta pri zaščiti spletne strani?
9. Kakšna je glavna prednost uporabe modelov "aaS" (kot sta IDaaS in SECaaS) za razvijalce spletnih strani v primerjavi s tem, da bi celoten varnostni sistem programirali sami od začetka?

#### Sklop C: Varnostni scenariji in Zero Trust
10. Med vajo ste poskusili dostopati do strani z napačnim e-poštnim naslovom. Opišite, kaj se je izpisalo na zaslonu in kaj se je v istem trenutku zabeležilo v administratorjevem dnevniku (Logs).
11. Kaj bi se zgodilo, če bi napadalec uspešno vdrl v vaš osebni profil na Cloudflaru? Do katerih podatkov in sistemov bi s tem pridobil dostop?
12. Predlagajte dodatno varnostno pravilo (poleg preverjanja končnice e-poštnega naslova), ki bi ga lahko dodali v Cloudflare vmesnik, da bi še bolj zmanjšali možnost nepooblaščenega dostopa (npr. omejitev glede na lokacijo ali čas).
