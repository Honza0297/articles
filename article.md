# Jak konvertovat RPM balíček do flatpaku

## flatpaku obecně

Poslední dobou se stále více mluví o nové edici Fedory - Silverblue. Ta, kromě jiného, nepoužívá klasický balíčkovací systém jako většina linuxových distribucí (třeba rpm v klasické Fedoře), ale spoléhá na distribuci aplikací přes flatpak. 

Flatpak, stejně jako rpm, je způsob nasazování aplikací. Od klasických balíčkovacích systémů se ovšem liší hned v několika věcech:
* Aplikace ve flatpaku jsou sandboxované -  běží ve svém vlastním kontejeneru - a pro přístup ke zdrojům (jako je třeba přístup k souborům nebo k síti) potřebují povolení. 
* Flatpakované aplikace si s sebou nesou všechny závislosti - odpadá tím problém s kompatibilitou různých verzí knihoven - všechno už za vás vyřešili ti, co aplikaci do flatpaku zabalili. 
* Díky předchozím dvěma bodům jsou aplikace ve flatpaku velmi snadno přenositelné mezi systémy - stačí, aby daný systém flatpak podporoval. 

Bohužel, jelikož je flatpak relativně nová technologie (prvotní myšlenka vznikla v roce 2013 a Silverblue, která spoléhá výhradně na flatpak, je tu s námi zhruba dva roky), nejsou tak distribuované zdaleka všechny aplikace. Většinu flatpakovaných aplikací najdete na flathub.org, který se stal *de facto standardem* pro hostování aplikací ve flatpaku. Poslední dobou má ale konkurenci v podobě repozitáře flatpakových aplikací spravovaných Fedorou, kd ejsou flatpaky vytvořené z rpm balíčků, o kterých je i celý článek.

## Jak se liší normální flatpaky od těch pro Silverblue?

Hlavním rozdílem je zdroj dat. "Normální" flatpaky jsou sestavovány přímo ze zdrojových kódů, kdežto "fedoří" flatpaky využívají jako zdroj dat rpm balíčky, což má za následek několik výhod. Díky rpm je třeba předem známý závislostní strom a aplikaci stačí "jen" zabalit. Díky rpm je také jasné, co a kam se instaluje a do jisté míry odpadá procházení stotisícových logů při zťjišování "proč tam ten soubor není". Celkově funguje rpm jako jakási kontrolní a přehledová vrstva.
Jako vše, i tohle řešení má svoje nevýhody. Jelikož se k sestavování flatpaků používají naprosto normální balíčky bez žádných úprav (ideálně se používá balíček z právě aktuální větve), jakákoli nutnost změny rpm vede k pull - requestu, jelikož spec soubor nejde nijak opatchovat.

## Co všechno je pro převod potřeba? 

Obecný postup je nejdříve vytvořit modul a z něj následně kontejner. V kontextu Fedory jsou flatpaky jen jinou formou kontejnerů a zachází se s nimi podobně jako s kontejnery pro serverové použití. Detailně je postup popsán dále:

Pokud používáme Silverblue, vytvořme si nový toolbox kontejner a vstupme do něj pomocí příkazů (pokud ne, rovnou pokračujme dál)
~~~
$ toolbox create -c jmeno-kontejneru
$ toolbox enter -c jmeno-kontejneru
~~~
Dále je nutné nainstalovat všechny potřebné nástroje...
~~~
$ sudo dnf install flatpak-module-tools fedmod
~~~
...přidat se do skupiny mock...
~~~
$ sudo usermod -a -G mock $USER
~~~
...a nainstalovat Fedora Flatpak Runtime
~~~
$ flatpak remote-add fedora-testing oci+https://registry.fedoraproject.org#testing
$ flatpak install fedora-testing org.fedoraproject.Platform/x86_64/f30
~~~

Pokud se vše podařilo, můžeme přistoupit k samotnému převodu aplikace. Pro ukázku nám bude sloužit hra Supertux (ta už je v repozitáři Fedory jako flatpak[0], ale je jednoduchá a proto vhodná na ukázku). Přejděte tedy do složky, kde plánujete kovertovat své aplikace (doporučuji si udělat například složku "flatpaks" a v ní potom podsložku pro každou aplikaci), vytvořte složku pro Supertuxe a přesuňte se do ní - název musí být stejný jako název budoucího flatpaku a ten by měl být stejný jako název rpm balíčku
~~~
$ mkdir supertux && cd supertux
~~~
Nyní si necháme vygenerovat vstupní soubory pro náš flatpak:
~~~
$ fedmod fetch-metadata
$ fedmod rpm2flatpak --flatpak-common --flathub=supertux supertux
~~~
Prvním příkazem získáme metadata pro RPM balíčky a druhým příkazem si vygenerujeme prvotní verze dvou souborů, které si popíšeme dále. Volba *--flatpak-common* přidává závislost vygenerovaného modulu na modullu flatpak-common. To nám často pomáhá dělat aplikace menší a snadněji sestavitelné. Druhá volba ( *--flathub=supertux* ) znamená, že se použije flathubový manifest pro inicializaci souboru container.yaml. Pokud na flathubu aplikace není, tuhle volbu můžeme vynechat a jestli zadané jméno způsobí více shod, jsou všechny zobrazeny a musíme zopakovat příkaz s konkrétnějším pojmenováním - nově zadané jméno je kompletní jméno flathubové verze Supertuxe. 
~~~
$ fedmod rpm2flatpak --flatpak-common --flathub=org.supertuxproject.SuperTux  supertux
~~~

Nyní se podíváme na vygenerované soubory:
*supertux.yaml*
~~~
1 ---
2 document: modulemd
3 version: 2
4 data:
5   summary: Jump'n run like game
6   description: >-
7     SuperTux is a jump'n run like game, Run and jump through multiple worlds, fighting
8     off enemies by jumping on them or bumping them from below. Grabbing power-ups
9     and other stuff on the way.
10   license:
11     module:
12     - MIT
13   dependencies:
14   - buildrequires: 
15       flatpak-common: [f30]
16       flatpak-runtime: [f30]
17       platform: [f30]
18     requires:
19       flatpak-common: [f30]
20       flatpak-runtime: [f30]
21       platform: [f30]
22   profiles:
23     default:
24       rpms:
25       - supertux
26   components:
27     rpms:
28       supertux:
29         rationale: Application package
30         ref: f30
31         buildorder: 10
32 ...
~~~

Nejvíce nás zajímá část *components*, která v našem případě obsahuje jedinou položku supertux, běžně tu ale najdeme jednotky až desítky závislostních balíčků. "rationale" je krátký popis dané komponenty, "ref:" nás odkazuje na větev dané komponenty v https://src.fedoraproject.org/, která bude použitá pro sestavení - větev jde použít libovolná, ale většinou se používá aktuální nebo master větev. Poslední je buildorder, který udává, v jakém pořadí se budou aplikace sestavovat - začíná se s těmi s nějnižším číslem. Později si ukážeme ještě další části, ty ale nyní nebudou potřeba. 


*container.yaml*
~~~
1 compose:
2     modules:
3     - supertux:master
4 flatpak:
5     id: org.supertuxproject.SuperTux
6     branch: stable
7     command: supertux2
8     rename-appdata-file: supertux2.appdata.xml
9     rename-desktop-file: supertux2.desktop
10     rename-icon: supertux2
11     finish-args: |-
12         --socket=wayland
13         --socket=x11
14         --share=ipc
15         --socket=pulseaudio
16         --share=network
17         --device=all                                
~~~

V téhle části tutoriálu si vysvětlíme jen část "finish-args" - víc stejně nebudeme potřebovat. V této části totiž aplikaci povolujeme jednotlivé interakce se systémem. Jelikož supertux je jen jednoduchá hra a nepotřebuje komunikovat po síti, --share=network můžeme smazat. Co přesně můžeme naší aplikaci povolit, můžeme najít například tady: http://docs.flatpak.org/en/latest/sandbox-permissions.html. 

Nyní se dostáváme k samotné konverzi. Tu provedeme příkazem
~~~
$ flatpak-module local-build --install
~~~
cože je zkratka pro sadu tří příkazů - sestavení modulu, sestavení kontejneru a nainstalování OCI obrazu kontejneru - jak už bylo zmíněno, flatpaky ve Fedoře jsou považovány jen za zvláštní druh kontejneru. Jednotlivé příkazy se mohou spouštět i postupně - třeba ve chvíli, kdy jen testujete různé povolování v kontejneru není nutné sestavovat celý modul. 
~~~
$ flatpak-module build-module
$ flatpak-module build-container --from-local
$ flatpak-module install <application>-master-<version>.oci.tar.gz
~~~

Pokud všechno proběhne bez problémů, můžeme otestovat náš první vytvoření flatpak - pokud se vyskytly jakékoli problémy, podívejte se na konec článku, kde budou probrané základní problémy nebo napište na foru TODO odkaz.
~~~
$ flatpak run org.supertuxproject.SuperTux 
~~~

TODO jak si založit FAS, pagure a podobně 

Nyní máme lokálně sestavený funkční flatpak. My ho ovšem cheme dát k dispozici i ostatním, takže ho musíme nahrá do infrastruktury Fedora - dál už je článek pouze teoretický, protože supertux už tam jako flatpak je. Pro jakoukoli jinou aplikaci je ale postup stejný. Prvním krokem je požadavek na vytvoření repozitáře - to může trvat i několik desítek hodin, jelikož se o to starají živí lidé, kteří kontrolují validitu požadavku a podobně:
~~~
$ fedpkg request-repo --namespace=flatpaks <jmeno_aplikace>
~~~ 

Jakmile máme náš repozitář, naklonujeme si ho a přesuneme do něj naše dva soubory - doporučuji ještě do souboru README přidat krátký popis aplikace
~~~
$ mv <jmeno_aplikace> <jmeno_aplikace>.old
$ fedpkg clone flatpaks/<jmeno_aplikace>
$ cd <jmeno_aplikace>
$ cp ../jmeno_aplikace.old/{<jmeno_aplikace>.yaml,container.yaml} .
$ git add <jmeno_aplikace>.yaml container.yaml
$ git commit -m "Initial import"
$ git push origin master
~~~

Nyní musíme nechat Fedoru, aby náš flatpak ověřila sestavením v Koji TODO odkaz - opět nejdříve modul
~~~
$ fedpkg module-build
~~~
a poté kontejner
~~~
$ fedpkg flatpak-build
~~~
Teď bychom měli otestovat flatpak vytvořený na Koji, abychom ověřili, že se nic nerozbilo
~~~
$ flatpak-module install --koji <jmeno_aplikace>:master
~~~


A poslední krok je na Bodhi[] TODO odkaz a založit nový update. Do políčka Candidate Builds vložíme NVR	našeho flatpaku - pokud ho nenajdeme v terminálu v logu předchozích kroků, můžeme ho najít i v koji. Stačí na https://bodhi.fedoraproject.org/updates/new vyhledat název aplikace. Hledané NVR bude jedno z vrchních a bude vypadat nějak takto: mojeaplikace-20b180601144429.2. Do políčka "Update notes" stačí napsat něco jako "Initial flatpak of <jmeno_aplikace>", Type vyberte "newpackage" a zmáčkněte "Submit". nyní stačí počkat, až flatpak projde testováním. A to je vše! 

# Troubleshooting aneb co se může rozbít




https://bodhi.fedoraproject.org/updates/new
https://docs.fedoraproject.org/en-US/flatpak/tutorial/



 