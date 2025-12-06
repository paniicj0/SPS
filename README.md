
# SMART PARKING SERVICE

### Namena aplikacije

Smart Parking System je aplikacija koja omogućava korisnicima da pronadju, rezervišu I započnu parkiranje na privatnom parkingu sa kontrolisanim ulazom. Sistem podržava pregled raspoloživih parking mesta u okviru privatnog parkinga, upravljanje rezervacijama, skeniranje QR kodova za identifikaciju parking mesta i ulaza, kao i asinhronu komunikaciju sa simuliranim senzorima o zauzetosti parking mesta.

### Uloge u aplikaciji

•	Neautentifikovani korisnik:

-	Može da se registruje
  
•	Autentifikovani korisnik – Vozač

-	Ima svoj nalog, može da se prijavi na sistem
-	Može da vidi sve informacije o parkingu
-	Može da vidi svoj profil, menja podatke
-	Može da rezerviše/otkaže mesto
  
•	Admin

-	Ima svoj nalog, može da se prijavi na sistem
-	Može da upravlja parking mestima
-	Može da pregleda analitiku

### Funkcionalni zahtevi

1.	Upravljanje korisnicima u aplikaciji

  1.1	 Registracija korisnika
  
  Ukoliko korisnik nema nalog na sistemu, potrebno je da napravi nalog, na početnoj stranici putem forme za registraciju. Registracija obuhvata unos email adrese, lozinke, imena, prezimena i broja telefona. Lozinka se unosi dva puta da bi se otežalo pojavljivanje grešaka prlikom odabira lozinke.
Takodje korisnik mora da potvrdi svoj identitet nakon registracije, klikom na aktivacioni link iz email-a. Korisnik ne može da se prijavi na aplikaciju dok se njegov nalog ne aktivira posećivanjem tog linka, koji je takodje vremenski ograničen na 24h. 

  1.2	Prijava na sistem
  
  Ukoliko korisnik ima nalog na sistemu i ako je aktivan, ima mogućnost prijave na sistem. Prijava se izvršava unošenjem email-a i lozinke. Ukoliko je korisnik uspešno prijavljen, generisaće se JWT token, u suprotnom ispisaće se poruka da kredencijali nisu dobri.

  1.3	Upravljanje nalogom / Profil korisnika
  
  Korisnik ima mogućnost da pregleda svoj nalog i da upravlja njime. Može da pogleda sve informacije o sebi i da ih menja ih osim emaila. Izmena lozinke je moguća, uz unošenje stare lozinke i željene nove.
Na profilu korisnika se nalazi deo za unos tablica svog vozila kako bi pri samoj rezervaciji mesta imao mogućnost da odabere za koje vozilo rezerviše. 
Na profilu se nalazi i deo za pregled istorije parkiranja.

  Admin ima mogućnost da radi CRUD operacije nad parkingom I parking mestima. Takodje može da pregleda zauzetost mesta i da pregleda analitike (grafikoni: dnevna zauzetost, mesečna statistka, itd..)

2.	Upravljanje rezervacijama

  2.1	Početna stranica korisnika

  Na početnoj strani korisnik vidi šemu privatnog parkinga. Na vrhu ekrana s desne strane nalazi se opcija mogućnost pregledanja korisnikovog profila, kao I deo za logout sa aplikacije. Sa desne strane ekrana (na sredini), nalaze se opcije koje korisnik bira prilikom same rezervacije. 

  Kompletan način rezervacije: Korisniku se prikazuje mogućnost za odabir vremena, pocetnog i krajnjeg za vreme parkiranja, dok se racuna cena za odredjen period. Ukoliko korisnik odabere vreme koje se preklapa sa svim rezervacijama, kada ni jedno mesto nije slobodno, dobiće povratnu poruku sa obaveštenjem da su sva mesta zauzeta odredjenog perioda. U suprotnom, nakon odabira jednih od svojih tablica moćiće da rezerviše mesto, sa povratnom porukom o uspešnosti rezervacije.
*Rezervacija korisniku garantuje pravo na jedno mesto u izabranom terminu, ali ne i na konkretan broj mesta. Sistem omogućava ulazak ako postoji makar jedno fizički slobodno mesto.

  2.2	Evidencija parkiranja

  Kada korisnik dodje do parkinga, da bi evidentirao dolazak skeniraće QR kod koji se nalazi na stubu rampe. Sistem zatim proverava preko korisnikovog JWT tokena i gateId-a, da li u tom periodu (+/- 20 min.), za tog korisnika ima zabeležena rezervacija. Ukoliko postoji, rampa se otvara, u suprotnom korisnik dobija poruku da ne postoji rezervacija za to vozilo u tom periodu. Ukoliko se ne validira u roku od max. 20min od početnog sata rezervacije, smatraće se da je odustao i mesto će ponovo biti slobodno. Nakon isteka vremena, korisnik će imati evidenciju u delu istorije parkiranja.

  2.3	Naplata parkinga i evidentiranje izlaska

  Pri izlasku sa parkinga, korisnik skenira opet QR kod za izlazak. U tom trenutku postavlja se vreme zavrsetka na trenutni datum I vreme izračunava trajanje sesije i na osnovu tarifnog modela (npr. naplate po započetom satu) računa ukupnu cenu parkinga. Nakon toga sistem generiše digitalni račun koji se čuva u bazi i šalje korisniku na e-mail adresu. Račun sadrži jedinstveni identifikator (u formi QR koda) i iznos za uplatu. U produkcionom okruženju, ovakav račun bi korisnik mogao da plati na različitim prodajnim mestima (trafike, prodavnice), pri čemu bi eksterni payment sistem nakon uspešne uplate obavestio aplikaciju da označi račun kao plaćen. U okviru ovog rada, ovaj proces je simuliran – status računa se menja pozivom posebnog endpoint-a ili putem admina.

  2.4	Istorija parkiranja

  Korisnik ce imati tabelu u kojoj će se nalaziti sve rezervacije. Sistem omogućava korisnicima otkazivanje prethodno napravljenih rezervacija pod određenim uslovima. Nakon uspešnog kreiranja rezervacije, korisnik ima pravo da je otkaže u ograničenom vremenskom periodu (npr. u roku od 10 minuta od trenutka kreiranja). 

  Nakon isteka ovog vremenskog perioda, otkazivanje rezervacije više nije dozvoljeno. Ukoliko korisnik ne iskoristi rezervaciju i ne pojavi se na parkingu u definisanom vremenskom prozoru (npr. do 20 minuta nakon planiranog vremena početka), rezervacija se automatski označava kao istekla i slobodna je opet za rezervaciju.

### Nefunkcionalni zahtevi

  3.	Arhitektura sistema

  Sistem mora biti zasnovan na mikroservisnoj arhitekturi i sadržaće najverovatnije 4-5 nezavisnih servisa. Mikroservisi za komunikaciju: REST API-ja za sinhronu i Message broker-a za asinhronu komunikaciju.
  Autentifikacija mora koristiti JWT token, sa vremenskim ograničenjem važenja. Lozinke moraju biti čuvane uz korišćenje hash funkcija. API rute podržavaju role-based acess control i validaciju ulaznih podataka. Aktivacioni link ima ograničeni rok trajanja od 24h.
  Servisi (otprilike): User Service, Reservation Service, Parking/Gate Service, Billing/Invoice Service, Notification Service (opciono)

4.	Tehnologije
   
  Backend: Rust, PostgreSQL baze, Python (opciono), RabbitMQ, JWT autentifikacija
  Frontend: Angular

##### Dodatno za diplomski bih odabrala nešto od ovoga: Obaveštavanje korisnika nakon 15 dana (ili pri kraju meseca) da odredjeni računi nisu plaćeni; Analiza ponašanja korisnika: prosečno trajanje parkiranja, najčešće vreme rezervacije...; Rekomendacioni sistem za personalizovane predloge rezervacija; Blokiranje korisnika koji je više puta rezervisao, ali se nije pojavio...ili nešto slično
