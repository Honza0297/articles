# Jak se konvertují RPM balíčky do flatpaku?

## O flatpaku obecně

Poslední dobou se stále více mluví o nové edici Fedory - Silverblue. Ta, kromě jiného, nepoužívá klasický balíčkovací systém jako většina linuxových distribucí (třeba RPM v klasické Fedoře), ale spoléhá na distribuci aplikací přes flatpak. 

Flatpak je, zjednodušeně řečeno, způsob distribuce aplikací. Od klasických balíčkovacích systémů se ovšem liší, a to hned v několika bodech. 
Zaprvé, aplikace ve flatpaku jsou sandboxované, běží ve svém vlastním kontejeneru a pro přístup ke zdrojům (jako je třeba přístup k souborům nebo k síti) potřebují povolení. Zadruhé, flatpakované aplikace si s sebou nesou všechny závislosti - odppadá tím s kompatibilitou různých verzí knihoven - všechno už za vás vyřešili ti, co aplikaci do flatpaku zabalili. A konečně zatřetí, díky sandboxingu a tomu, že si aplikace nesou všechny závislosti, jsou flatpakované aplikace přesnosné na libovolný systém, který flatpak podporuje.  

To zní dobře, ne? 

Bohužel, jelikož je flatpak relativně nová technologie (prvotní myšlenka vznikla v roce 2013 a Silverblue, která spoléhá výhradně na flatpak, je tu s námi zhruba dva roky), nejsou tak distribuované zdaleka všechny aplikace. Na hlavním repozitáři flatpakových aplikací, flathubu, jich ovšem lze najít více než dost. Flathub ale není jediné místo, odkud stahovat flatpaky. Tým Silverblue pracuje na svém vlastním repozitáři flatpakovaných aplikací a tento článek bude právě o tom, jak udělat vlastní flatpak pro Silverblue.

## Jak se liší normální flatpaky od těch pro Silverblue?

Hlavním rozdílem je zdroj dat. "Normální" flatpaky jsou sestavovány přímo ze zdrojových kódů, kdežto "fedoří" flatpaky využívají jako zdroj dat RPM balíčky. Primárním důvodem je určitá garance kvality - RPM balíčky garantují, jaké soubory kam instalují, že jsou funkční a jdou sestavit. Vše se kontroluje přímo v infrastrukuře Fedory. Na druhou stranu, sestavování flatpaků z RPM vyžaduje, aby RPM byly bezchybné - například prefixy musí být v RPM specifikované makrem, aby mohly flatpakovací nástroje tenhle prefix správně změnit.

## Co všechno je pro převod potřeba? 




 