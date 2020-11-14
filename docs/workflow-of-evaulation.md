---
id: workflow-of-evaulation
title: Pontozás menete
sidebar_label: Pontozás menete
---
# Áttekintés 

A pontozás tekintetében három időszakra osztjuk a félévet. Ezek optimális esetben az alábbi sorrendben követik egymást, de a rendszer tudja kezelni, ha a sorrend felcserélődik. Amikor az időszakot átállítják a következő félév nyugalmi időszakára, akkor automatikusan kiszámoljuk a PointHistoy-t a felhasználóknak.


## Implementációs részletek

- A [Pundit](https://github.com/varvet/pundit) gem-et használtuk a jogosultság kezeléshez.
  - Ha az **authorize** függvényt használjuk, akkor a kontroller és action neve alapján lesz meghatározva, hogy melyik policy és annak melyik metódusa lesz használva a jogosultság eldöntésére.
  - Az EvaluationController-ből leszármazó kontollerek, a JustificationsController és a PrinciplesController a szülő kontolleren keresztül autorizálódnak. Nem kell külön authorizálni őket.
  - EntryRequestsController és a  PointDetailsController, mivel nem származnak le más kontrollerből, ezért új EvaluationPolicy példány létrehozásával authorizáljuk.
  - A view-kban is policy osztályok példányait használjuk, hogy eldöntsük mely UI elemeket kell megjeleníteni. Például egy gombot az alapján jelenítűnk meg, hogy az oldal amire átirányítja a felhasználót, az elérhető a felhasználó számára. Igy ha a jogosultságok változnak, akkor elég csak a policy-ben frissíteni, a UI elemek automatikusan követni fogják a frissítést.

## Jó tudni

### A pontok és a belépőigények külön vannak kezelve

A körvezető külön adhatja le a pontozást és a belépőigényeket. Emellett az RVT is külön bírálhatja ezeket. Például lehet olyan helyzet, hogy a belépők elfogadásra kerülnek, míg a pontozást elutasítják.

### Pontozási időszakban a körvezető visszavonhatja a leadott értékeléseket

A körvezetőnek van lehetősége a pontozást és belépőigényeket, egymástól függetlenül, leadni és a leadást visszavonni. Itt a leadott státusz jelzés értékű. Látható legyen, melyik körök vannak készen a pontozással.

### Módosítható pontozások

Bírálási időben csak az elutasított pontozások módosíthatók. Pontozási időszakban az elfogadott értékeléseken kívül bármelyik értékelés módosítható, ha nincs leadva. A leadott értékeléseket vissza kell vonni, hogy lehessen őket szerkeszteni.

## Jogosultsági szintek

3 darab jogosultsági szint van jelenleg az értékeléseknél. Egy felhasználó jelenlegi jogosultságának meghatározásához a szerep köreit és az aktuális időszakot is figyelembe vesszük.

Ezek a szintek az alábbiak:

### Megtekintés

Az adott felhasználó megnézheti a kör pontozását és belépőigényeit, de nem szerkesztheti.

A körvezető, reszort vezeztő és a pék admin számára érhető, ha nem nyugalmi időszak van.

### Pontozás és belépő igénylési jog

A felhasználónak van joga az adott pontozást és belépőigénylést szerkeszteni, leadni.

A körvezető számára lehetséges, ha nincs leadva a pontozás és még módosítható.

A reszortvezető számára lehetséges a bírálási időszak alatt bármikor.

Implementáció szinten külön van kezelve a pontozások és belépőigények leadása. Emellett a leadás és visszavonás is külön lett implementálva.

### Értékelés szerkesztői jog

A felhasználó tud pontozási elveket, beszámolókat és belépőigény indoklásokat felvenni és szerkeszteni.

A körvezetőnek és a pék adminnak van jogosultsága erre, ha nincs leadva a pontozás és módosítható.

## Nyugalmi időszak - off season

Ebben az időszakban kezdődik a félév. Egyetlen pontozással kapcsolatos funkció sem elérhető.

## Pontozási időszak - application season

Ebben az időszakban állítják össze a körvezetők a körök pontozását a félévre. A pontozások bármikor leadhatók és visszavonhatók. Ebben az időszakban a leadott pontozási státusz csak jelzés értékű. Bármikor vissza lehet vonni és további módosításokat végrehajtani a pontozásokban.

## Bírálási időszak - evaluation season

Ebben az időszakban bírálja el az RVT a leadott pontozásokat. Ha megfelelő a pontozás, akkor elfogadására kerül. Ellentétes esetben a pontozást elutasítják. Ekkor a körvezető el tudja végezni a kért módosításokat. Elutasított pontozásoknál ebben az időszakbab a körvezető leadás után már nem tudja módosítani az értékelést, tehát figyelni kell arra, hogy elsőre helyes legyen a pontozás. Ha a körvezető nem végzi el a kért módosításokat, akkor a megfelelő reszort vezetője is elvégezheti azokat.

## Tippek
- Az összetett logikai kifejezéseknél figyelni kell a precedenciára! Érdemes zárójelekkel garantálni a megfelelő kiértékelési sorrendet.
