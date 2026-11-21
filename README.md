# Adware Scanner

Ingyenes, nyílt forráskódú Android-alkalmazás a reklámokkal teli, kéretlen alkalmazások (adware) felismerésére és eltávolítására, beépített reklámfigyelővel és reklámszűrővel.

## Funkciók

- **Tiltólista-ellenőrzés:** a telepített alkalmazásokat összeveti egy tiltólistával (kamu „frissítők”, „tisztítók”, „vírusirtók”, reklámos kamerák, PDF/QR-olvasó klónok stb.). Minden találatnál megmutatja, miért gyanús az app, és egy gombnyomással eltávolítható. Az eltávolítást mindig a rendszer saját megerősítő ablaka kéri.
- **Háttérfigyelés:** kb. 15 percenként megnézi, települt-e új alkalmazás, és ha tiltólistás, értesítést küld. Naponta frissíti a listát.
- **Bejelentés:** bármelyik appot be lehet jelenteni gyanúsként.
- **Reklámfigyelő:** a használati statisztikából felismeri a teljes képernyős reklámokat (AdMob, AppLovin, Unity, ironSource, Meta, Pangle stb.), appok szerint számolja őket, és figyelmeztet, ha egy app túl sok reklámot mutat, vagy *más app használata közben* dob fel reklámot.
- **Reklámszűrő (bekapcsolható):** telefonon belüli, csak DNS-kéréseket szűrő VPN. A reklám- és követőcímeket nyilvános listák (StevenBlack, opcionálisan AdGuard DNS-szűrő) alapján blokkolja. Nincs külső VPN-szerver, nincs fiók vagy feliratkozás. Az appon kívül reklámozó appok internetét automatikusan elvághatja (visszaállítható).
- **Kivételek:** a szűrésből kizárható appok (pl. banki appok) és engedélyezett címek, valamint a legutóbb blokkolt címek naplója.

## Adatvédelem

- Nem gyűjt személyes adatot, nincs benne reklám, analitika vagy fiók.
- A telepített appok listája és a reklámstatisztika a telefonon marad.
- Hálózati forgalom: a tiltólista és a reklámlisták letöltése, valamint a felhasználó által kezdeményezett bejelentés (az app neve, csomagneve, telepítési forrása és egy opcionális megjegyzés).

## Követelmények

- Android 8.0 (API 26) vagy újabb. Az appok szerinti reklámstatisztikához Android 10 (API 29) kell.
- Fordításhoz: Android Studio (Ladybug vagy újabb), JDK 17+, Android SDK 35.

## Fordítás

1. Klónozd a repót, majd az Android Studióban válaszd az **Open** lehetőséget, és nyisd meg az `android` mappát.
2. Várd meg a Gradle-szinkronizálást. Ez letölti a függőségeket.
3. Állítsd be a saját értékeidet az `android/app/build.gradle.kts` fájl `defaultConfig` részében:
   ```kotlin
   buildConfigField("String", "API_BASE", "\"https://…/api/\"")   // a tiltólista-szolgáltatás címe
   buildConfigField("String", "PUBLIC_URL", "\"https://…/\"")     // tájékoztató oldal (Névjegy)
   buildConfigField("String", "GITHUB_URL", "\"https://github.com/…\"")
   buildConfigField("String", "CONTACT_EMAIL", "\"…\"")
   ```
   Ha a tiltólista-szolgáltatás nem érhető el, az app az APK-ba épített listával működik tovább.
4. Futtatás telefonon: **Run ▶** (USB-hibakeresés bekapcsolva).
5. Kiadható APK: **Build → Generate Signed App Bundle / APK → APK**, saját aláírókulccsal.

Parancssorból:

```bash
cd android
./gradlew assembleDebug          # hibakereső APK: app/build/outputs/apk/debug/
./gradlew assembleRelease        # kiadási APK (aláírás beállítása szükséges)
./gradlew testDebugUnitTest      # egységtesztek
```

### Reklámlista fordításkor

A fordítás automatikusan letölti a teljes StevenBlack-reklámlistát az `app/src/main/assets/hosts_stevenblack.txt` fájlba, és hetente frissíti. Ha nincs internet, a fordítás ettől nem áll le: az app a beépített alaplistával (`assets/hosts_base.txt`) indul, és futás közben tölti le a teljes listát. A letöltött fájl nincs verziókezelve (`.gitignore`).

### Beépített tiltólista

Az `app/src/main/assets/blacklist.json` az első indításkor és offline használatkor érvényes tiltólista. Formátuma:

```json
{
  "version": 1794060000,
  "categories": { "fake_updater": "Kamu frissítő" },
  "ad_activity_prefixes": [],
  "apps": [
    { "package": "com.updatesoftware.updateallapps", "name": "Update All Apps",
      "risk": "high", "category": "fake_updater",
      "reason": "Kamu „szoftverfrissítő”: frissíteni nem tud, csak reklámokat jelenít meg." }
  ]
}
```

## Engedélyek

| Engedély | Mire kell |
|---|---|
| `QUERY_ALL_PACKAGES` | a telepített alkalmazások listázása az ellenőrzéshez |
| `REQUEST_DELETE_PACKAGES` | az eltávolító ablak megnyitása (a rendszer mindig rákérdez) |
| `POST_NOTIFICATIONS` | értesítés tiltott app telepítésekor, reklámos appokról |
| `INTERNET`, `ACCESS_NETWORK_STATE` | listák letöltése, bejelentés |
| `PACKAGE_USAGE_STATS` | reklámfigyelő; a felhasználó adja meg a beállításokban |
| `BIND_VPN_SERVICE` | reklámszűrő (helyi, csak DNS-t szűrő VPN) |
| `RECEIVE_BOOT_COMPLETED` | a reklámszűrő újraindítása a telefon bekapcsolása után |
| `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` | kérés, hogy a rendszer ne állítsa le a háttérellenőrzést |

A `QUERY_ALL_PACKAGES` engedély miatt az app nem a Google Play Áruházba, hanem APK-ként (pl. GitHub Releases) terjeszthető.

## Felépítés

```
android/app/src/main/java/hu/csakoattila/adwarescanner/
  data/   tiltólista, telepített appok, beállítások, hálózati hívások
  ads/    reklámfigyelő: használati események, reklám-aláírások, helyi adatbázis
  vpn/    reklámszűrő: VpnService, DNS-csomagkezelés, reklámlisták
  work/   háttérellenőrzés (WorkManager), értesítések
  ui/     Jetpack Compose képernyők
android/app/src/test/   egységtesztek (DNS-csomagkezelés, listafeldolgozás)
```

Technológia: Kotlin, Jetpack Compose (Material 3), WorkManager, SQLite. A DNS-csomagkezelés tiszta Java, Android-függőség nélkül, így egységtesztelhető.

## Ismert korlátok

- A reklámszűrő mellett nem futhat más VPN.
- A YouTube reklámjai nem szűrhetők, mert ugyanarról a címről jönnek, mint a videók.
- Ha a telefonon egyéni Privát DNS van beállítva, vagy a böngészőben „Biztonságos DNS”, ezek megkerülik a szűrőt. Az app figyelmeztet erre.
- A lebegő (más appok felett megjelenő) reklámok nem látszanak a használati statisztikában. Az ilyen engedéllyel rendelkező appokat az app külön jelöli.

## Licenc

MIT – lásd a [LICENSE](LICENSE) fájlt.

Fejlesztő: Csákó Attila
