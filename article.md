#Jak předělat RPM balíčky do flatpaku

## Co je to ten flatpak?

Poslední dobou se stále více mluví o nové edici Fedory - Silverblue. Ta, kormě jiného, nepoužívá klasický balíčkovací systém jako většina linuxových distribucí, ale spoléhá na distribuci aplikací přes flatpak. Co že to ten flatpak je? Zjednodušeně řečeno, flatpak je způsob, jak dodávat plikaci uživatelům. Od klasických balíčkovacích systémů, které dělají to samé, se ovšem liší, a to hned v několika bodech. 
Zaprvé, aplikace ve flatpaku jsou sandboxované a běží ve svém vlastním kontejeneru a pro přístup ke zdrojům (jako je třeba přístup k souborům nebo k síti) potřebují povolení. Zadruhé, flatpakované aplikace si s s ebou nesou všechny závislosti - odppadá tím problém, že aplikace X nefunguje s knihovnou Y ve verzi Z. A konečně zatřetí, díky sandboxingu a tomu, že si aplikace nesou všechny závislosti, jsou flatpakované aplikace přesnosné na libovolný systém, který flatpak podporuje.  

To zní dobře, ne? Jelikož je flatpak relativně nová technologie (prvotní myšlenka vznikla v roce 2013 a Silverblue, která spoléhá výhradně na flatpak, je tu s námi zhruba dva roky), nejsou tak distribuované zdaleka všechny aplikace. Na hlavním repozitíři flatpakových aplikací, flathubu, jich ovšem lze najít více než dost. Ovšem flathub není jediné místo, odkud stahovat flatpaky. Silverblue pracuje na svém vlastním repozitáři flatpakovaných aplikací a tento článek bude právě o tom, jak udělat vlastní flatpak pro Silverblue.

## Jak se liší normální flatpaky od těch pro Silverblue?

Hlavním rozdílem je zdroj dat: "normální" flatpaky jsou sestavovány přímo ze zdrojových kódů, kdežto "fedoří" flatpaky využívají RPM balíčky. Primárním důvodem je určitá garance kvality - RPM balíčky garantují, jaké soubory kam instalují, jsou funkční a jdou sestavit. Vše se kontroluje přímo v infrastrukuře Fedory. Na druhou stranu, sestavování flatpaků z RPm vyžaduje, aby RPM byly skutečně bezchybné - například prefixy musí být v RPM specifikované makrem, aby mohly flatpakovací nástroje tenhle prefix správně změnit (nebojte se, vše vysvětlím dále). 
 