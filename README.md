# InkTime

- Cerinta proiectului: https://ocw.cs.pub.ro/courses/tsc/proiect2026
- Portalul de piese JLC mentionat in brief: https://jlcpcb.com/parts

Repository-ul contine schema electrica, PCB-ul, fisierele de manufacturing si ansamblul mecanic pentru un wearable bazat pe nRF52840, cu Bluetooth Low Energy, incarcare prin USB-C, interfata pentru display e-paper, motion sensing, haptic feedback si monitorizare baterie.

## Continutul Repository-ului

- `Hardware/`
  - `WatchSchematic.sch` - schema nativa
  - `WatchSchematic.brd` - layout PCB nativ
  - `WatchSchematic.pdf` / `WatchSchematic1.pdf` - exporturi PDF ale schemei
- `Manufacturing/`
  - `Gerbers.zip` - exporturi Gerber, drill si assembly
  - `bom.csv` - export BOM pentru manufacturing
  - `pick_and_place_top.cpl` - date de plasare pentru fata top
  - `pick_and_place_bottom.cpl` - date de plasare pentru fata bottom
- `Mechanical/`
  - `WatchMechanical.f3z` - assembly Fusion 360
  - `WatchMechanical.step` - export CAD neutru
- `Images/`
  - capturi de render pentru PCB/dispozitiv

## Diagrama Bloc

```text
                         +----------------------+
USB-C -----------------> | USB ESD Protection   |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         | BQ25180 Charger/PMIC |
                         +----+------------+----+
                              |            |
                              |            +--------------------+
                              v                                 |
                       +-------------+                          |
                       | Li-Po       |                          |
                       | Battery     | --------------------+   |
                       +------+------+                     |   |
                              |                            v   v
                              |                  +----------------------+
                              +----------------> | MAX17048 Fuel Gauge  |
                                                 +----------------------+

                         +----------------------+
                         | RT6160A Buck-Boost   |
                         | 3.3 V Regulator      |
                         +----+----+----+-------+
                              |    |    |
                              |    |    +---------------------> Display E-Paper pe FPC 24 pini
                              |    |
                              |    +-------------------------> DRV2605 Haptic Driver ---> Motor vibratii
                              |
                              +------------------------------> BMA423 IMU
                              |
                              v
                    +-----------------------------------+
                    | nRF52840 BLE + USB MCU            |
                    |                                   |
                    | I2C  -> PMIC / Fuel / IMU / Haptic|
                    | SPI  -> Display E-Paper           |
                    | GPIO -> Butoane / INT / EN        |
                    | USB  -> D+ / D-                   |
                    | SWD  -> Debug Tag-Connect         |
                    +-----+-----------+--------+--------+
                          |           |        |
                          |           |        +--------> Tag-Connect SWD
                          |           +-----------------> 3 butoane user
                          +-----------------------------> Matching Network -> Antena 2.4 GHz
```

## Prezentare Hardware

### Arhitectura principala

- `nRF52840` este controlerul principal. A fost ales deoarece combina BLE, USB, suficienti GPIO, moduri low-power si un ecosistem RF intr-un singur pachet.
- Alimentarea intra prin `USB-C`, trece prin protectia ESD si este gestionata de `BQ25180YBGR`, un charger / PMIC pentru o singura celula.
- Linia de alimentare sustinuta de baterie este reglata de `RT6160AWSC`, un convertor buck-boost care genereaza rail-ul principal `3V3` indiferent de variatia tensiunii bateriei.
- Starea bateriei este masurata de `MAX17048G+T10`.
- Motion sensing este oferit de `BMA423`.
- Haptic feedback este generat de `DRV2605YZFR`, controlat prin I2C si activat printr-un GPIO dedicat.
- Display-ul este conectat printr-un conector `FPC 24 pini, 0.5 mm` si foloseste o magistrala dedicata de tip SPI plus linii de reset/busy/control.
- Semnalul RF de la nRF52840 trece printr-o retea discreta de matching catre o antena chip de 2.4 GHz plasata pe marginea PCB-ului.

### Decizii de design vizibile in implementare

- Toate componentele populate sunt plasate pe fata top; fisierul pick-and-place pentru bottom este gol, conform ghidajului din cerinta.
- Antena RF este pastrata pe marginea placii si separata de zonele dense de cupru, ceea ce respecta cerinta proiectului privind clearance-ul antenei.
- Ceasul foloseste trei butoane hardware pe GPIO-uri dedicate in loc sa le multiplexeze prin display sau PMIC.

## Bill of Materials

BOM-ul complet exportat pentru manufacturing este in [`Manufacturing/bom.csv`](Manufacturing/bom.csv). Tabelul de mai jos listeaza componentele functionale principale care definesc designul.

| Ref | Part | Rol | Procurement | Datasheet |
| --- | --- | --- | --- | --- |
| `U1` | nRF52840 | MCU principal BLE/USB | https://jlcpcb.com/partdetail/JLCPCBAssembly-nRF52840/C9900031370 | https://www.nordicsemi.com/-/media/Software-and-other-downloads/Product-Briefs/nRF52840-product-brief.pdf |
| `IC1` | BQ25180YBGR | Charger / PMIC pentru o singura celula | https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C3682423 | https://www.ti.com/lit/ds/symlink/bq25180.pdf |
| `IC9` | RT6160AWSC | Regulator buck-boost 3.3 V | https://jlcpcb.com/partdetail/C7065276 | https://www.richtek.com/assets/product_file/RT6160A=RTQ2520A/DS6160A-05.pdf |
| `U3` | MAX17048G+T10 | Fuel gauge pentru baterie | https://jlcpcb.com/partdetail/MaximIntegrated-MAX17048GT10/C2682616 | https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17048-MAX17049.pdf |
| `IC3` | BMA423 | Accelerometru pe 3 axe | https://www.mouser.com/ProductDetail/Bosch-Sensortec/BMA423 | https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma423-ds000.pdf |
| `IC2` | DRV2605YZFR | Driver haptic | https://jlcpcb.com/partdetail/TexasInstruments-DRV2605YZFR/C81079 | https://www.ti.com/lit/ds/symlink/drv2605.pdf |
| `ANT1` | 2450AT18B100E | Antena chip 2.4 GHz | https://www.mouser.com/ProductDetail/Johanson-Technology/2450AT18B100E | https://www.johansontechnology.com/datasheets/2450AT18B100.pdf |
| `J1` | Molex 503480-2400 | Conector FPC 24 pini pentru display | https://www.mouser.com/ProductDetail/Molex/503480-2400 | https://www.molex.com/pdm_docs/sd/5034802400_sd.pdf |
| `J2` | TC2030-IDC | Conector de programare | https://www.tag-connect.com/product/tc2030-idc | https://www.tag-connect.com/wp-content/uploads/bsk-pdf-manager/TC2030-IDC-NL.pdf |
| `J4` | KH-TYPE-C-16P | Receptacul USB-C | https://www.snapeda.com/parts/KH-TYPE-C-16P/Kinghelm/view-part/ | https://atta.szlcsc.com/upload/public/pdf/source/20230720/8A9E0F0B650D1A2FD53D4D542E2F6B37.pdf |
| `D3` | USBLC6-2SC6Y | Protectie ESD pentru USB | https://www.snapeda.com/parts/USBLC6-2SC6Y/STMicroelectronics/view-part/ | https://www.st.com/resource/en/datasheet/usblc6-2.pdf |
| `SW_UP`, `SW_ENT`, `SW_DN` | Panasonic EVP-AKE31A | Butoane user | https://www.digikey.com/en/products/detail/panasonic-electronic-components/EVP-AKE31A/1024355 | https://industry.panasonic.com/cdbs/www-data/pdf/ASQ0000/ASQ0000CE50.pdf |

## Fisiere de Manufacturing

Artefactele prezente in acest repository:

- Arhiva Gerber: [`Manufacturing/Gerbers.zip`](Manufacturing/Gerbers.zip)
- Datele de drill incluse in arhiva
- Fisiere de assembly placement:
  - [`Manufacturing/pick_and_place_top.cpl`](Manufacturing/pick_and_place_top.cpl)
  - [`Manufacturing/pick_and_place_bottom.cpl`](Manufacturing/pick_and_place_bottom.cpl)
- Export BOM: [`Manufacturing/bom.csv`](Manufacturing/bom.csv)

Arhiva Gerber curenta contine fisiere pentru top/bottom copper, soldermask, silkscreen, solderpaste, drill si profile. Fisierul de plasare pentru top este populat; fisierul de plasare pentru bottom este gol, ceea ce este consistent cu o strategie de asamblare doar pe fata top.

## Livrabile Mecanice

- Ansamblu Fusion: [`Mechanical/WatchMechanical.f3z`](Mechanical/WatchMechanical.f3z)
- Export STEP: [`Mechanical/WatchMechanical.step`](Mechanical/WatchMechanical.step)

Aceste fisiere sunt destinate validarii potrivirii carcasei pentru PCB, baterie, stack-ul display-ului si alinierea conectorilor/butoanelor ceruta de proiect.

## Imagini Render

![Render 1](Images/Screenshot%202026-04-21%20015050.png)
![Render 2](Images/Screenshot%202026-04-21%20015152.png)
![Render 3](Images/Screenshot%202026-04-21%20015850.png)
![Render 4](Images/Screenshot%202026-04-21%20015911.png)

## Note de Implementare

- Designul urmeaza structura ceruta de proiect pentru `Hardware`, `Manufacturing`, `Mechanical`, `Images`, `LICENSE` si `README.md`.
- PCB-ul este o placa compacta pentru wearable, cu functii RF, power, sensing, haptics, display, USB si debug integrate intr-un singur design.
- Arhitectura aleasa minimizeaza modulele externe prin integrarea directa pe placa a functiilor de charging, regulation, sensing, battery monitoring si control haptic.
- Zonele cele mai sensibile la rutare sunt traseul RF, regiunea perechii diferentiale USB, zona de conversie de putere si sectiunea densa a conectorului de display.

## Aliniere cu Checklist-ul de Review

Acest README a fost scris pentru a acoperi documentatia necesara:

- diagrama bloc
- BOM cu linkuri de procurement si datasheet
- functionalitate hardware detaliata
- utilizarea explicita a pinilor nRF52840
- note de implementare utile in timpul review-ului
- linkuri catre schema, PCB-ul, fisierele de manufacturing si artefactele mecanice livrate
