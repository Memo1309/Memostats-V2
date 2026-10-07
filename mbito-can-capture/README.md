# MBito CAN Capture

Aplicație web autonomă pentru iPhone + Bluefy, compatibilă cu firmware-ul `MBito-Sniffer` din `main.c` (ESP32-C3, CAN în mod listen-only 500 kbps).

## Pornire

Site-ul trebuie servit prin **HTTPS** și deschis în **Bluefy** pe iPhone, nu în Safari, care nu oferă API-ul necesar Web Bluetooth.

1. Apasă **CONECTEAZĂ DONGLE** și selectează `MBito-Sniffer`.
2. Alege durata scurtă, preferabil 1 secundă la trafic intens.
3. Apasă **START**. Aplicația trimite comanda `01` la FFF1.
4. La expirarea duratei sau la butonul **STOP + DESCARCĂ**, aplicația trimite `02` apoi `03` la FFF1.
5. Așteaptă mesajul **DUMP complet** (`A3`) și exportă **JSON** sau **CSV**.

Datele sunt decodificate din notificările FFF2: `A1` conține timestamp (uint32 LE), CAN ID (uint32 LE), flags, DLC, 8 octeți date și numărul de secvență pe 8 biți. `A2` este status, `A3` conține numărul așteptat de cadre și suprascrierile.

## Limitări reale

- Firmware-ul actual are doar **4096 cadre** în buffer circular. La 36k cadre în ~10s, poate păstra doar ultima secundă sau două. Pentru o animație de 25s, **nu poate salva totul** fără firmware nou sau filtrare dinamică.
- În timpul capturii nu se transmit cadrele în timp real prin BLE. Transmisia începe numai la DUMP.
- Când numărul recepționat diferă de A3 `count`, descărcarea a fost incompletă. Chiar dacă acestea coincid, `overwritten > 0` înseamnă că începutul capturii a fost pierdut înainte de descărcare.
- Captura poate surprinde doar traficul care este vizibil prin OBD. Nu este demonstrat încă faptul că evenimentul Blue Welcome este vizibil pe acea magistrală.
- Nu transmite cadre CAN în vehicul și nu controlează luminile sau ECU-urile.
- Exportul este generat local. Nu închide aplicația în timp ce se descarcă, nici înainte de export.

## Publicare pe Vercel

Proiect static, fără dependințe. Descarcă fișierele și publică folderul `mbito-can-capture` ca proiect static (Root Directory). Dacă este necesar, configurează `Framework Preset: Other`, `Output Directory: .`. Pentru deploy prin CLI din folder, se poate folosi `vercel --prod` după autentificare.