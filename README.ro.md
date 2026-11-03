*[English](README.md) · **Română***

# Joc 2D în Java — motor de joc pe tile-uri

Un proiect personal de învățare: un joc 2D văzut de sus, scris de la zero în **Java**, fără motor de
joc și fără biblioteci externe — doar `Swing` și `AWT` din JDK. Scopul proiectului a fost să înțeleg
ce face de fapt un motor de joc în spate: bucla de joc, sincronizarea cadrelor, animația sprite-urilor,
hărțile din tile-uri și citirea tastaturii.

| | |
|---|---|
| **Autor** | Ștefan Horia-Eusebiu |
| **Limbaj** | Java (Swing / AWT, fără dependențe externe) |
| **Tip** | Proiect personal, în lucru |
| **Început** | Octombrie 2024 |

---

## Ce este implementat

**Buclă de joc cu pas fix.** `GamePanel` implementează `Runnable` și rulează pe propriul fir de
execuție. Fiecare iterație apelează `update()` și `repaint()`, apoi așteaptă exact cât a mai rămas
până la următorul cadru:

```java
double drawInterval = 1000000000 / FPS;          // 60 FPS → 16,67 ms pe cadru
double nextDrawTime = System.nanoTime() + drawInterval;
// ... update(); repaint();
Thread.sleep((long) remainingTime);
nextDrawTime += drawInterval;
```

Acumularea lui `nextDrawTime`, în loc de o pauză de durată fixă, împiedică frecvența cadrelor să
deriveze atunci când un cadru durează mai mult decât era de așteptat.

**Randare cu dublu buffer.** `setDoubleBuffered(true)` pe panou, desenare prin `Graphics2D` în
`paintComponent` suprascris: întâi tile-urile, apoi jucătorul deasupra.

**Sistemul de tile-uri.** `TileManager` încarcă texturile tile-urilor și citește harta dintr-un fișier
text în care fiecare număr este un index în vectorul de tile-uri:

```
4 4 4 4 4 4 4 4 4 4 4 4 4 4 4 4 …
4 4 4 4 4 4 4 4 4 4 0 2 2 2 2 0 …
```

**Animația sprite-urilor.** Jucătorul are 8 sprite-uri (2 cadre de animație × 4 direcții). Un contor
schimbă cadrul la fiecare 12 tick-uri de joc, adică de aproximativ 5 ori pe secundă la 60 FPS —
destul de repede ca să pară mers, destul de încet ca să nu pâlpâie.

**Citirea tastaturii.** `KeyHandler` implementează `KeyListener` și păstrează câte un boolean pentru
fiecare direcție (`W`, `A`, `S`, `D`), setat la `keyPressed` și resetat la `keyReleased`. Citirea
stării în loc de reacția la evenimente face mișcarea lină și continuă cât timp tasta e apăsată.

**Geometria ecranului.** Tile-uri de 16×16 px scalate de 3× → 48 px pe ecran; o fereastră de 16×12
tile-uri → o fereastră de 768×576.

## Structura proiectului

```
.
├── src/main/java/
│   ├── joc2d/
│   │   ├── Main.java          punctul de intrare — construiește JFrame-ul și pornește firul
│   │   ├── GamePanel.java     bucla de joc, geometria ecranului, randarea
│   │   └── KeyHandler.java    citirea tastaturii (WASD)
│   ├── entity/
│   │   ├── Entity.java        clasa de bază: poziție în lume, viteză, sprite-uri, direcție
│   │   └── Player.java        jucătorul — mișcare, animație, desenare
│   └── tile/
│       ├── Tile.java          un tile: textură + indicator de coliziune
│       └── TileManager.java   încărcarea texturilor, citirea hărții, desenarea
└── res/
    ├── player/                8 sprite-uri ale jucătorului (2 cadre × 4 direcții)
    ├── tiles/                 38 de texturi de tile-uri
    └── maps/                  hărțile, ca fișiere text cu indici de tile-uri
```

## Rulare

Încă nu există un fișier de build în repository, așa că proiectul se compilează direct cu JDK-ul
(17 sau mai nou). Folderul `res/` este rădăcina resurselor și trebuie să fie în classpath, pentru că
codul își încarcă fișierele prin `getResourceAsStream`:

```bash
javac -d out $(find src/main/java -name "*.java")
java -cp out:res joc2d.Main
```

Pe Windows, folosește `;` în loc de `:` ca separator de classpath:

```bash
java -cp "out;res" joc2d.Main
```

Alternativ, deschide folderul în IntelliJ IDEA, marchează `src/main/java` ca *Sources Root* și `res`
ca *Resources Root*, apoi rulează `Main`.

**Comenzi:** `W` `A` `S` `D` pentru deplasare.

## Stadiul actual și ce urmează

Este un proiect în lucru, iar codul arată asta cinstit. Ce nu este încă implementat:

- **Coliziunile.** `Tile` are deja un boolean `collision`, dar încă nu îl citește nimic — jucătorul
  trece prin orice. Acesta e pasul următor.
- **Camera.** Jucătorul este desenat într-un punct fix pe ecran, iar tile-urile la coordonate fixe de
  ecran, fără decalajul unei camere. Coordonatele jucătorului în lume se schimbă, dar imaginea încă nu
  le urmărește, deci lumea nu derulează.
- **Încărcarea hărții complete.** Lumea este declarată de 50×50 de tile-uri, dar `loadMap()` și
  `draw()` sunt limitate de `maxScreenCol` / `maxScreenRow` (16×12), așa că se citește doar colțul
  vizibil din `world01.txt`. Fișierul hărții conține deja toate cele 50 de coloane.
- **`worldHeight`** este calculat ca `tileSize * maxScreenRow` în loc de `maxWorldRow` — va trebui
  corectat odată cu camera.

## Ce am învățat

- O buclă de joc nu e doar „un `while` infinit” — fără o sincronizare controlată, jocul rulează cu
  altă viteză pe fiecare calculator.
- Citirea tastaturii ca *stare* (câte un boolean pe tastă), nu ca *evenimente*, este ce face
  mișcarea să pară continuă în loc să fie sacadată.
- Separarea lui `update()` de `draw()` este distincția care păstrează logica jocului independentă de
  randare.
- O hartă de tile-uri ca simplu fișier text cu indici este un format surprinzător de eficient: poate fi
  editată în orice editor de text și citită cu câteva linii de cod.

---

Lucrare academică și personală, publicată în scop de portofoliu.
