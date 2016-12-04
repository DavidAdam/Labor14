# Labor14

## Bevezető

Android 6.0 (API level 23) verziótól kezdve a felhasználó futásidőben adhatja meg az alkalmazás engedélyeit, és nem az alkalmazás telepítésekor vagy frissítésekor. Dönthet úgy, hogy bizonyos engedélyeket nem ad meg egy alkalmazásnak, így nagyobb fokú irányítás kerül a kezébe. Az alkalmazásengedélyeket később bármikor módosíthatja az alkalmazásbeállítások képernyőn.

Az engedélyek két kategóriába vannak sorolva: normal/dangerous.
A normal kategóriába tartozó engedélyek nem jelentenek közvetlen kockázatot a felhasználó érzékeny adataira, ezeket az engedélyeket a rendszer automatikusan megadja.

A dangerous kategóriába tartozó engedélyek lehetőséget adhatnak az alkalmazásnak, hogy a felhasználó érzékeny adataihoz hozzáférjen. Ebben az esetben a felhasználónak kell megadni az engedélyt az alkalmazás számára.

https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous

Minden esetben meg kell adni a normal és a dangerous engedélyeket a manifest fájlban, de ennek hatása eltér a rendszer verziójától és a target sdk szinttől függően:

Ha az eszköz Android 5.1 vagy alacsonyabb verziót futtat, VAGY az alkalmazás target SDK szintje 22 vagy kisebb, akkor a rendszer telepítéskor kéri el az összes engedélyt, és ha a felhasználó nem fogadja el, akkor a telepítés leáll.

Ha az eszköz Android 6.0 verzióval rendelkezik ÉS az alkalmazás target SDK szintje 23 vagy nagyobb, akkor az alkalmazás a futása során fogja elkérni a dangerous kategóriába tartozó engedélyeket, a normal engedélyeket pedig a rendszer automatikusan megadja. A felhasználó bármely engedélyt megadhat, vagy letilthat, emiatt az alkalmazás limitált funkcionalitással futhat tovább, melyet megfelelően kell kezelni.

## Jogosultság ellenőrzése
Amennyiben az alkalmazás funkciójának egy veszélyes engedélyre van szüksége, akkor minden esetben ellenőrízni kell még a funkció indítása előtt, hogy rendelkezik-e az engedéllyel, hiszen az engedélyeket a felhasználó bármikor módosíthatja.
ContextCompat.checkSelfPermission()

## Jogosultság elkérése
Az Android rendszer számos metódust biztosít egy jogosultság elkérésére. Ezeket meghívva egy nem testreszabható dialógust dob fel a rendszer.
ActivityCompat.requestPermissions()

## Jogosultság magyarázata
Egyes esetekben szükséges lehet a felhasználót tájékoztatni, hogy miért kér az alkalmazás veszélyes engedélyeket.
ActivityCompat.shouldShowRequestPermissionRationale()

## Feladat

## Kezdő lépések

A labor során egy egyszerű telefonkönyv alkalmazást kell elkészíteni. Az alkalmazás listázni tudja a telefonon tárolt névjegyeket, majd egy adott elemre kattintva hívást lehet kezdeményezni.

Hozzunk létre egy új Android Studio Projektet PermissionsLabor néven. A Company Domain mező tartalmát töröljük ki és hagyjuk is üresen.

A packagename legyen hu.bme.aut.amorg.examples.permissionslabor A támogatott céleszközök a Telefon és Tablet, valamint a minimum SDK szint az API15: Android 4.0.3

A kezdő projekthez adjuk hozzá egy Empty Activity-t, melynek neve legyen ContactsActivity.

Vegyük fel a RecyclerView komponens függőségét, illetve állítsuk a targetSDK-t 23 vagy nagyobbra a build.graddle(module:app) fájlban, majd nyomjuk meg a Sync Now gombot. Amennyiben nincs 23, vagy magasabb SDK telepítve a gépre, akkor frissítsük az SDK Manager segítségével a szükséges komponenseket.

## Felhasználói felület

Első lépésként készítsük el az alkalmazás felhasználói felületét XML erőforrásból. A felület egyetlen RecyclerView komponensből fog állni.

```xml
<?xml version="1.0" encoding="utf-8"?>
```


