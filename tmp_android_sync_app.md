# Software - DualEmSync (Android Sync-App.) 
[Retur til README](../../README.md)

## Beskrivelse:
Setup, ATV trækker en Slæde der opsamler data om undergrunden, på slæden er der en Raspberry Pi (RPi) som opsamler data og gemmer disse lokalt, 
på ATV er der en Andoid Tablet med en App. der henter data fra RPi og også gemmer disse lokalt, derudover skal Android App give info til chaffør om aktuel status.  

RPi opsamler data fra en DualEM geologi sensor (10 hz) og GPS (2 hz) og gemmer disse data i en ".nema" fil. ".nema" filer har fortløbende nummere, "1.nmea" & "2.nmea" osv.  
RPi laver en ny fil ved Start Log, og starter derefter automatisk data opsamling.  
RPi`s Power-Supply er en Powerbank på 10.000 mAh, hele systemet skal optimeres så RPi bruger mindst mulig strøm.  
RPi har 2 modes, en mode til måling og en mode til test/simulering.  
Android App. skal hente ".nmea" filer fra RPi, med faste mellemrum.  
Android App. skal give info om aktuel status.  
Android App. har 2 modes, LIVE der hentes data fra RPi, DEMO der hentes ingen data fra til RPi men istedet anvendes interne data.  

## SW - Facts og Krav:
* **Tablet Model:** Oukitel RT9 anvendes på ATV. Lenovo TAP M9 til test/udvikling.  
* **Tablet Display:** App. skal automatisk tilpasse sig størelse på Tablet display (flydende layout).
* **Tablet Display:** App. skal "tvinge" display til altid at være tændt.
* **Tablet Display:** App. skal kunne indeholde flere sprog. Default = Engelsk.
* **Tablet Display:** High Contrast tema til bl.a. direkte sollys, Android systemet styrer dette.
* **Tablet Display:** Farve sammensætning der er bedst til Outdoor-brug.
* **Tablet Mode:** Tablet anvendes i Landscape mode. App. skal kun programmes til denne.
* **Tablet Forbindelse:** Tablet er forbundet til Raspberry Pi via Wi-fi. Raspberry Pi er Hotspot.
* **Tablet Betjening:** Tablet skal kunne betjenes med handsker, i det omfang det er muligt.  
* **Tablet Power Supply:** Tablet anvender indbygget 11.000 mah batterie.  

## SW - Generelt:
* **Data Netværk:** RPi fungerer som Hotspot. Tablet logger paa som gaest. Dette giver den mest stabile genforbindelse i marken.
* **Data Model:**  Passiv HTTP-Pull (Incremental Sync).
* **Data Server:**  RPi koerer en letvaegts-webserver. Den er passiv og belaster kun RPi-batteriet minimalt.
* **Data Full Synkronisering:**  Ved App. opstart kigger i RPi datamappen og laver en Full spejling af datafiler (kan tage flere minutter).
* **Data Update Synkronisering:**  Ved App. loop kigger i RPi datamappen hvert 1. til 120. sekund (setup værdi).
* **Data Update Metode:**  App. tjekker filstoerrelsen paa den aktive fil. Hvis filen er vokset, henter App. kun de nye bytes og tilfoejer dem til sin lokale tekstfil.
* **Data sikkerhed:**  Ved WiFi-udfald mistes ingen data. RPi skriver ufortroedent til sin egen disk, og App. indhenter automatisk efterslaebet ved genforbindelse.
* **Data Effektivitet:**  Da App. styrer traekket, minimeres WiFi-trafikken, hvilket sparer stroem paa RPi batteri-forsyningen.
* **Data Sletning:**  Ingen filer slettes automatisk på hverken Raspberry Pi og Android App. Dette gøres manuaelt.

## SW - Opbygning - Version 1:
> * **App. Navn:** DualEmSync.
> * **App. Icon:** Fra Bargheer Geophysics homepage.  

> * **App. Program:** Opstart.  
**App. Initialisering:** Ved opstart er `activeLogFile` altid `offline.log`. Alle systemhændelser logges her.  
**App. Valg af Mode:** Brugeren vælger LIVE eller DEMO. Logning fortsætter i `offline.log` indtil data er verificeret.  
**App. Data-verifikation:** Sync (HTTP-Pull eller Simulator) leder efter første gyldige filnavn og datalinje.  
**App. Skift af Log:** Når første data er verificeret opdateres `activeLogFile` til det matchende nummer (f.eks. `14.log`).  
**App. Dynamisk Rulning:** Hvis RPi skifter filnummer (f.eks. fra `14.nmea` til `15.nmea`) under en session, opdaterer Sync `activeLogFile` til `15.log`. Appen følger automatisk med.  

> * **Display sider:** Skift mellem sider 1-5 vha. menu (Drawer)
```text
* Side, 0: "Opstart"
    * Diverse checks (se Appendix A)  

* Side Header: Side 1-5  
    * Understående sider har sammen Header (se Appendix B)   

* Side 1: "Dashboard", Logning STOPPET  
    * Kolonne 1: Start LIVE knap.  
    * Kolonne 2: Start DEMO Knap.  
* Side 1: "Dashboard", Logning STARTET  
    * Kolonne 1: Offline Log eller Online Log. (se Appendix C)     
    * Kolonne 2: Status felt RESULTAT, RØD = Alarm, GUL = Warning, GRØN = Ok
                 Status felt PROGRAM, GRÅ = Venter, BLÅ = Igang
    * Kolonne 3: Sidste hentede data fra Raspberry Pi. (se Appendix D) 

* Side 2: "Setup"  
    * Kolonne 1: Projekt data. (se Appendix E) 
    * Kolonne 2: System/Globale data. (se Appendix F)  

* Side 3: "Stop & Exit"  
    * Kolonne 1: Stop LIVE/DEMO knap.  
    * Kolonne 2: Exit App. Knap.  

* Side 4: "Info"  
    * Sektion 1: Wifi.
    * Sektion 2: Betjening.
    * Sektion 3: Download data.
    * Sektion 4: Slet filer.
    * Sektion 5: Fil typer.
    * Sektion 6: Data filer.

* Side 5: "About"  
    * Kolonne 1: Ejer/kunde navn og kontakt.   
    * Kolonne 2: Programmør navn og kontakt. App. og RPi version, samt hvilke versioner der passer sammen
```
> * **Sekvens:** STARTUP og SEQUENCE (Loop) i LIVE eller DEMO mode
```text
        Start LIVE knap
        Start DEMO Knap 
            |   
            | STARTUP (INIT)
            ├ StartUp(1): Reset     
    ┌─────>─┼ StartUp(2): Set Init  
    |       ├ StartUp(3): Connect Data (Full)  
    |   ┌─<─┼ StartUp(4): Connect RPi (Full)   
    |   ├─<─┼ StartUp(5): Full Mirror (RPi -> Tablet)
    |   |   |
    |   └─>─┼ StartUp(7): Result
    └─────<─┼ StartUp(8): Wait (Retry)
            ├ StartUp(9): Finish (Start Sequence) 
            |
            | SEQUENCE (ACTIVE) - Loop
    ┌─────>─┼ Seq(10): Set Active
    |       ├ Seq(11): Connect Data (Update)
    |   ┌─<─┼ Seq(12): Connect RPi (Update)
    |   ├─<─┼ Seq(13): Update Mirror (RPi -> Tablet)
    |   ├─<─┼ Seq(14): Find Valid datafile number in folder
    |   ├─<─┼ Seq(15): Last data to buffer & Dashboard
    |   ├─<─┼ Seq(16): All Data Check: New Lines
    |   |   |
    |   ├─<─┼ Seq(18): RPi Data Check: Version
    |   |   ├ Seq(19): RPi Data Check: Other
    |   |   |
    |   |   ├ Seq(20): GPS Data Check: Log Frequency
    |   |   ├ Seq(21): GPS Data Check: Data Frozen
    |   |   ├ Seq(22): GPS Data Check: Data Quality
    |   |   ├ Seq(23): GPS Data Check: ATV max Speed
    |   |   |
    |   |   ├ Seq(30): DualEM Data Check: Log Frequency
    |   |   ├ Seq(31): DualEM Data Check: Data Frozen
    |   |   ├ Seq(32): DualEM Data Check: Power (Volt)
    |   |   ├ Seq(33): DualEM Data Check: Roll
    |   |   |
    |   └─>─┼ Seq(40): Result
    |       |
    |     <─┼ Seq(45): Stop LIVE/DEMO knap 
    |       |
    |       ├ Seq(49): Finish & Wait (Wait or Retry)
    └─────<─┘ 
```
> * **Sekvens:** WARNING og ALARM (RESULTAT)
```text
STARTUP (INIT)
* App. data checks:
StartUp(4): Connect RPi (Full)                  Retry: WARNING/ALARM = forgæves forsøg tæl op, tæller> 1/3
StartUp(5): Full Mirror (RPi -> Tablet)         Retry: WARNING/ALARM = forgæves forsøg tæl op, tæller> 1/3

SEQUENCE (ACTIVE) - Loop
* App. data checks:
Seq(12): Connect RPi (Update)                   Retry: WARNING/ALARM = forgæves forsøg tæl op, tæller > 1/3
Seq(13): Update Mirror (RPi -> Tablet)          Retry: WARNING/ALARM = forgæves forsøg tæl op, tæller> 1/3
Seq(14): Find Valid datafile number in folder   Retry: ALARM = Ingen valid datafil fundet / Manglende/slettet datafil
Seq(15): Last data to buffer & Dashboard        Retry: WARNING/ALARM = For få liner / Fejl ved læsning 
Seq(16): All Data Check: New Lines              Retry: WARNING/ALARM = Hvis ingen nye data liner tæl op, tæller > 1/2

* RPi data checks:
Seq(18): RPi Data Check: Version                Set: ALARM = version passer ikke til App. version.
Seq(19): RPi Data Check: Other - Disk           Set: WARNING/ALARM = Anvendt Disk plads > 85/95 %
Seq(19): RPi Data Check: Other - Volt           Set: WARNING/ALARM = Hvis voltHist=true / Hvis voltAlarm=true 
Seq(19): RPi Data Check: Other - Throttle       Set: WARNING/ALARM = Hvis throttleHist=true / Hvis throttleAlarm=true
Seq(19): RPi Data Check: Other - Soft Temp.     Set: WARNING/ALARM = Hvis softTempHistv / Hvis softTempAlarm=true

* GPS data checks:
Seq(20): GPS Data Check: Log Frequency          Set: WARNING/ALARM = Hvis Hz < 1.9 eller Hz > 2.1 tæl op, tæller > 1/3
Seq(21): GPS Data Check: Data Frozen            Set: WARNING/ALARM = Hvis line $GNGGA er ens tæl op, tæller > 1/3
Seq(22): GPS Data Check: Data Quality           Set: WARNING/ALARM = Find quality < 1, beregn PV %, Hvis PV > SP tæl op, tæller > 1/3 
Seq(23): GPS Data Check: ATV max Speed          Set: WARNING/ALARM = Hvis speed > SP tæl op, tæller > 1/3 

* DualEM data checks:
Seq(30): DualEM Data Check: Log Frequency       Set: WARNING/ALARM = Hvis Hz < 9.9 eller Hz > 10.1 tæl op, tæller > 1/3
Seq(31): DualEM Data Check: Data Frozen         Set: WARNING/ALARM = Hvis line $PDLM1/$PDLM2/$PDLM4 er ens tæl op, tæller > 1/3
Seq(32): DualEM Data Check: Power (Volt)        Set: WARNING/ALARM = Hvis gennemsnit < Warning SP / Hvis gennemsnit < Alarm SP
Seq(33): DualEM Data Check: Roll                Set: WARNING/ALARM = Hvis gennemsnit > SP i, Warning tid SP / Alarm tid SP
```
----------------------------------------------------------------------------------------------------
## SW - fremtidige funktioner - Version 2:
Der skal, i det omfang der muligt, tages hensyn til at understående funktioner (i prioteret rækkefølge) kan implemteres.  
* **Plot map:** Map der viser hvor der opsamlet data, live (Qfield?).
* **Visualisering:** Graf der viser geologi-sensorens udsving live (oscilloskop-stil).
* **Speedometer:** Visuel advarsel hvis ATV kører for stærkt til optimal dataindsamling (f.eks. > 15 km/t).
* **Audio-feedback:** En tone der ændrer frekvens alt efter geologi-data (så chaufføren kan "høre" jorden).
----------------------------------------------------------------------------------------------------

## Appendix
----------------------------------------------------------------------------------------------------
> * **A: Opstart side (SplashPage)**
```text
* Sekvens med feedback:
    [1] "Checking permissions..."               → `[checking/OK/ERROR]` (MANAGE_EXTERNAL_STORAGE).
    [2] "Verifying folders..."                  → `[checking/OK/ERROR]` (Root, RPi_data, Demo_data).
    [3] "Starting log..."                       → `[checking/OK/ERROR]` (Skrivning til log verificeret).
    [4] "Configuring setup values..."           → `[checking/OK/ERROR]` (Ændringer verificeret).
    [5] "System Ready. Starting Dashboard..."   → `[checking/OK/ERROR]` (pause 2 sekunder).
* Feedback fejl (1-4):**
    Ved fejl i Trin 1-4 blokeres skærmen af en *Popup*, hvor bruger kan aktivere "Prøv igen" eller "Afslut app".

* [1] "Checking permissions..."
    Appen kræver rettigheder til at oprette mapper og skrive data i disse.

* [2] "Verifying folders..."
    Appen tjekker og opretter automatisk mappestrukturen ved hver opstart, hvis den mangler.
    / (root)
    └── Documents/ 
        └── DualEnSync/         <-- offline.log
            ├── Demo_data/      <-- Simulerede data og tilhørende x.log
            └── RPi_data/       <-- Live data fra RPi og tilhørende x.log

* [3] "Starting log..."
    Skriver til offline-log
```
----------------------------------------------------------------------------------------------------
> * **B: - Side Header** Global Status
```text
VENSTRE SIDE
* DualEmSync - [aktuel side]
    [aktuel side] = Dashboard     (Menu)  
    [aktuel side] = Setup         (<-)  
    [aktuel side] = Stop & Exit   (<-)  
    [aktuel side] = Info          (<-)
    [aktuel side] = About         (<-)

HØJRE SIDE
* RPi: (RPi mode).
    `PENDING`    : Ingen forbindelse til RPi. (grå)
    `LIVE`       : Forbindele til RPi og RPi i Live mode. (blå)
    `SIMULATION` : Forbindele til RPi og RPi i Simulerings mode. (orange)

* APP: (App mode).
    `PENDING`    : Ingen mode valgt (grå)
    `LIVE`       : LIVE mode aktiveret, data fra RPi. (blå)
    `DEMO`       : DEMO mode aktiveret, data fra simulator. (orange)

* SYNC(n): Sync Step.
    `PENDING(n)` : Ingen mode valgt (grå)
    `INIT(n)`    : LIVE/DEMO mode aktiveret, opstart af data synkronisering med RPi. (cyan)
    `ACTIVE(n)`  : LIVE/DEMO mode aktiveret, første data er synkroniseret med RPi. Loop synkronisering kører. (grøn)
    `STOP(n)`    : STOP LIVE/DEMO mode aktiveret, Loop Synkronisering stoppes kontrolleret. (rød)

* FILE`(log file navn).
    `offline.log`: Ingen forbindelse til RPi (blå).
    `x.log`      : Forbindelse til RPi, x er aktuel RPi data fil nr. (grøn)
```
----------------------------------------------------------------------------------------------------
> * **C: - log fil** eksempler i offline.log & x.log
```text
* App. logs
2026-04-27 09:29:46, "App. Startup/Init OK --------------------"
2026-04-27 09:31:46, "App. Mode = LIVE"
2026-04-27 09:33:45, "App. Mode = DEMO"

* Demo logs
2026-04-27 09:29:48, "Demo: INIT - Reset all"
2026-04-27 09:29:48, "Demo: OFF - Reset Index & Time"
2026-04-27 09:29:48, "Demo: ON - Cold Start, Delayed start file:"
2026-04-27 09:29:48, "Demo: ON - Hot Start, Resuming from file/index:"
2026-04-27 09:29:48, "Demo: PAUSE - Position saved at file/index:"

* StartUp logs
2026-04-27 09:31:46, "Sync: StartUp steps -- INIT --"
2026-04-27 09:31:52, "Sync: StartUp steps -- OK --"

* Sequence new file logs (første liner i log)
2026-04-27 09:31:52, "Sync: Sequence loop -- ACTIVE --"
2026-04-27 13:51:32, "Sync: New Valid datafile Found, no: 174"
2026-04-27 13:51:33, "Sync: Project Data: Project Name / 1 / Project Date / Operator Initials"
2026-04-27 13:51:33, "Sync: Settings, DualEM Flight height:  10.0 [cm]"

* Sequence OK logs
2026-04-27 13:51:33, "Sync: RESULT to: 5.752 -- OK --"
2026-04-27 13:51:33, "Sync: Waittime, sec: 10  -- NEXT --"

* Sequence WARNING logs
2026-04-27 14:02:10, "Sync: Connect to RPi: 192.168.20.1/data/ -- WARNING --"
2026-04-27 14:02:10, "Sync: RESULT to: 632.772 -- WARNING --"
2026-04-27 14:02:10, "Sync: Waittime, sec: 20  -- RETRY --"

* Sequence ALARM logs
2026-04-27 14:22:51, "Sync: Connect to RPi: 192.168.20.1/data/ -- ALARM --"
2026-04-27 14:22:51, "Sync: RESULT to: 1833.770 -- ALARM --"
2026-04-27 14:22:51, "Sync: Waittime, sec: 20  -- RETRY --"
```
----------------------------------------------------------------------------------------------------
> * **D: Data liner** RPi, GPS og DualEM.
```text
* **RPI, 1 sæt fra RPi består af 1 line type (log frekvens 10-30 sek.)**
    * Type R1:  30.115, $RPI, 100, 1, 16, 40.9, 0, 1, 0, 1, 0, 1
                (Timestamp, Id, Version, Mode, Disk, Temp, VoltAlarm, VoltHist, ThrottleAlarm, ThrottleHist, SoftTempAlarm, SoftTempHist)

* **GPS, 1 sæt fra sensor består af 1 line type (frekvens 2Hz)**
    * Type G1:  0.936514,$GNGGA,145343.00, 5506.24691,N,01014.87678,E,1,12,0.57,8.2,M,43.6,M,,*40  
                (Timestamp, Id, UTC-Time, Latitude, N/S, Longitude, E/W, Quality, Satellites, HDOP, Height, M, Height correction, M, , *Checksum)

* **DualEM, 1 sæt fra geologi sensor består af et sæt på 4 liner typer (frekevens 10Hz)**  
    * Type D1:  0.515154, $PDLM1, 001156,+0.0,+1.65,+2.7,+0.15*08  
    * Type D2:  0.528715,$PDLM2,001156,+10.4,+1.51,+3.8,-3.67*37  
    * Type D3:  0.540237,$PDLM4,001156,+15.1,+3.71,+8.6,-9.31*3D  
                (Timestamp, Id, Time, HCP conductivity, HCP inphase, PRP conductivity, PRP inphase *hex checksum)
    * Type D4:  0.549025,$PDLMA,+13.20,+5.5,-0.1,+0.2*51  
                ('Timestamp, Id, Volt, Temp, Pitch, Roll *hex checksum)
```
----------------------------------------------------------------------------------------------------
> * **E: Setup** Projekt data.
```text
* Projekt Navn [tekst]                  [default: Input Project Name]           kan ændres
* Projekt Målenr 1-9999 [tal]           [default: 1]                            kan ændres
* Projekt Dato [tekst]                  [default: Input Project Date]           kan ændres
* Projekt Initaler [tekst]              [default: Input Project Initials]       kan ændres
* RPi, Disk Warning [%]                 [default: 85]  
* RPi, Disk Alarm [%]                   [default: 95]
* GPS, Frekvens [Hz]                    [default: 2]
* GPS, Kvalitet [%]                     [default: 95]
* GPS, ATV Hastighed max. 5-50 [km/t]   [default: 40]                           kan ændres
* DualEM Flyhøjde 3-300 [cm]            [default: 10]                           kan ændres
* DualEM Frekvens [Hz]                  [default: 10]
* DualEM Power Warning [V]              [default: 12.8]
* DualEM Power Alarm [V]                [default: 12.2]
* DualEM Roll +/- max. [Gr.]            [default: 10.0]
* DualEM Roll Warning [sek]             [default: 20.0]
* DualEM Roll Alarm [sek]               [default: 30.0]
```
----------------------------------------------------------------------------------------------------
> * **F: Setup** System data (global).
```text
* App. Demo Mode [Dropdown]             [default: Init]                         kan ændres
* App. Opdaterings tid 5-120 [sek]      [default: 10]                           kan ændres
* RPi. IP Adresse [tekst]               [default: 192.168.20.1]                 kan ændres
* RPi. Data mappe [tekst]               [default: /data/]
* RPi. Genopret forbindelse 5-60 [sek]  [default: 20]                           kan ændres
* RPi. Inkluder .info fil [bit]         [default: true]
* App. Skriv Ok-linjer i Log [bit]      [default: false]                        kan ændres
* App. Eksporter Log fejldatafil [bit]  [default: false]                        kan ændres
* App. Sprog [Dropdown]                 [default: En]                           kan ændres
```
----------------------------------------------------------------------------------------------------
> * **G: Opøsning.**  
```text
* ATV hastighed 10 km/t = 2,8 m/sek. Opøsning: DualEM = 0,28 m, GPS = 1,4 m  
* ATV hastighed 20 km/t = 5,6 m/sek. Opøsning: DualEM = 0,56 m, GPS = 2,8 m  
* ATV hastighed 30 km/t = 8,3 m/sek. Opøsning: DualEM = 0,83 m, GPS = 4,8 m  
* ATV hastighed 40 km/t = 11,1 m/sek. Opøsning: DualEM = 1,11 m, GPS = 5,6 m  
* ATV hastighed 50 km/t = 13,9 m/sek. Opøsning: DualEM = 1,39 m, GPS = 6,9 m  
    * Den optimale DualEM/GPS opløsning er afhængig af opgaven.  
```
----------------------------------------------------------------------------------------------------
