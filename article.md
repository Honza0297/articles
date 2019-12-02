# Jak z RPM vytvořit Flatpak


## O Flatpaku obecně

Poslední dobou se stále více mluví o nové edici Fedory - Silverblue. Ta, kromě jiného, nepoužívá klasický balíčkovací systém jako většina linuxových distribucí (třeba RPM v klasické Fedoře), ale spoléhá na distribuci aplikací přes Flatpak. 

Flatpak, stejně jako RPM, je způsob nasazování aplikací. Od klasických balíčkovacích systémů se ovšem liší hned v několika ohledech. Aplikace ve Flatpaku jsou sandboxované -  běží ve svém vlastním kontejneru - a pro přístup ke zdrojům jako soubory či síť, potřebují povolení. Flatpakované aplikace si s sebou také nesou všechny závislosti - odpadá tím problém s kompatibilitou různých verzí knihoven - vše už za vás vyřešili ti, co aplikaci do Flatpaku zabalili. Díky těmto vlastnostem jsou aplikace ve Flatpaku velmi snadno přenositelné mezi systémy - stačí, aby daný systém Flatpak podporoval.

Bohužel, jelikož je Flatpak relativně nová technologie (prvotní myšlenka vznikla v roce 2013 a Silverblue, která spoléhá (téměř) výhradně na Flatpak, je tu s námi zhruba dva roky), nejsou tak distribuované zdaleka všechny aplikace. Většinu Flatpakovaných aplikací najdete na [flathub.org](https://www.flathub.org), který se stal *de facto standardem* pro hostování aplikací ve Flatpaku. Poslední dobou přibila nová alternativa v podobě repozitáře Flatpakových aplikací spravovaných Fedorou. Tam najdete Flatpaky vytvořené z RPM balíčků, o kterých bude zbytek článku.


## Jak se liší "normální" Flatpaky od těch pro Fedoru?

Hlavním rozdílem je zdroj dat. "Normální" Flatpaky jsou sestavovány přímo ze zdrojových kódů, kdežto Flatpaky od Fedory využívají jako zdroj dat RPM balíčky, což má za následek několik výhod. RPM balíčky jsou vytvářeny transparentním způsobem pod dohledem distribuce a jsou tedy důvěryhodné, což u Flathubu, kde jsou Flatpaky vytvářeny dobrovolníky nemusí být vždy pravdou. Dále je předem známý strom závislostí a aplikaci stačí "jen" zabalit. Díky RPM je také jasné, co a kam se instaluje - ve Flatpaku se sice data aplikace nacházejí ve složce /app, ale stejně je dobré mít možnost dohledat, ke které části aplikace ten či onen soubor patří.
Jako vše, i toto řešení má svoje nevýhody. Jelikož se k sestavování Flatpaků používají normální balíčky bez možnosti dodatečného patche, v jejich spec souborech se občas nachází speciální úpravy kvůli Flatpakům, které do nich přidají několik dalších řádků.


## Jak samotné převádění funguje? 

Obecný postup je nejdříve vytvořit modul a z něj následně kontejner. V kontextu Fedory jsou Flatpaky jen jinou formou kontejnerů a zachází se s nimi podobně jako s kontejnery pro serverové použití.
Teď, když už víme vše potřebné, se můžeme pustit do praktické ukázky. Pokud používáme Silverblue, vytvořme si nový toolbox kontejner a vstupme do něj pomocí následujících příkazů.
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
...a nainstalovat Fedora Flatpak Runtime.
~~~
$ flatpak remote-add fedora-testing oci+https://registry.fedoraproject.org#testing
$ flatpak install fedora-testing org.fedoraproject.Platform/x86_64/f31
~~~

Pokud se vše podařilo, můžeme přistoupit k samotnému převodu aplikace. Pro ukázku nám bude sloužit hra Supertux - ta už v repozitáři Fedory jako Flatpak je [0](https://src.fedoraproject.org/flatpaks/supertux) a navíc je jednoduchá a proto vhodná na ukázku. Přejděme tedy do složky, kde plánujeme konvertovat své aplikace - doporučuji si udělat například složku "flatpaks" a v ní potom podsložky pro každou aplikaci - vytvořme složku pro Supertuxe a přesuňme se do ní - název musí být stejný jako název budoucího Flatpaku a ten by měl být stejný jako název RPM balíčku.
~~~
$ mkdir supertux && cd supertux
~~~
Nyní si necháme vygenerovat vstupní soubory pro náš Flatpak.
~~~
$ fedmod fetch-metadata
$ fedmod rpm2flatpak --flatpak-common --flathub=supertux supertux
~~~
Prvním příkazem získáme metadata pro RPM balíčky a druhým příkazem si vygenerujeme prvotní verze dvou souborů, které si popíšeme dále. Volba *--flatpak-common* přidává závislost vygenerovaného modulu na modulu flatpak-common a tím nám často pomáhá dělat aplikace menší a snadněji sestavitelné. Druhá volba - *--flathub=supertux* - znamená, že se použije flathubový manifest pro inicializaci souboru container.yaml. Pokud na Flathubu aplikace není, tuhle volbu můžeme vynechat a jestli zadané jméno způsobí více shod, jsou všechny zobrazeny a musíme zopakovat příkaz s konkrétnějším pojmenováním - pro ukázku jsem upravil předchozí příkaz na kompletní jméno flathubové verze Supertuxe. 
~~~
$ fedmod rpm2flatpak --flatpak-common --flathub=org.supertuxproject.SuperTux  supertux
~~~


## Soubory <aplikace>.yaml a container.yaml 
Nyní se podíváme na vygenerované soubory.

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
15       flatpak-common: [f31]
16       flatpak-runtime: [f31]
17       platform: [f31]
18     requires:
19       flatpak-common: [f31]
20       flatpak-runtime: [f31]
21       platform: [f31]
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

Nejvíce nás zajímá část *components*, která v našem případě obsahuje jedinou položku supertux, běžně tu ale najdeme jednotky až desítky závislostních balíčků. Položka "supertux" má v sobě vnořených několik dalších položek: "rationale" je krátký popis dané komponenty, "ref:" nás odkazuje na větev dané komponenty v [1](https://src.fedoraproject.org/), která bude použitá pro sestavení - větev lze použít libovolnou, ale většinou se používá aktuální nebo master. Poslední je "buildorder", který udává, v jakém pořadí se budou aplikace sestavovat - začíná se s těmi s nějnižším číslem a není-li buildorder specifikovaný, je defaultně nastavený na 0. Později si ukážeme ještě další části, ty ale nyní nebudou potřeba. 


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
10    rename-icon: supertux2
11    finish-args: |-
12         --socket=wayland
13         --socket=x11
14         --share=ipc
15         --socket=pulseaudio
16         --share=network
17         --device=all                                
~~~

Dále si vysvětlíme část "finish-args" - více nebudeme nyní potřebovat. Zde aplikaci povolujeme jednotlivé interakce se systémem. Jelikož supertux je jen jednoduchá hra a nepotřebuje komunikovat po síti, --share=network můžeme smazat. Co přesně můžeme naší aplikaci povolit, můžeme najít například tady: http://docs.flatpak.org/en/latest/sandbox-permissions.html. 


## Konverze začíná!

Nyní se dostáváme k samotné konverzi. Tu provedeme příkazem níže.
~~~
$ flatpak-module local-build --install
~~~
Tento příkaz je zkratka pro sadu tří příkazů - sestavení modulu, sestavení kontejneru a nainstalování OCI obrazu kontejneru - jak už bylo zmíněno, Flatpaky ve Fedoře jsou považovány jen za zvláštní druh kontejneru. Jednotlivé příkazy se mohou spouštět i postupně - třeba ve chvíli, kdy jen testujete různé povolování v kontejneru není nutné sestavovat celý modul. 
~~~
$ flatpak-module build-module
$ flatpak-module build-container --from-local
$ flatpak-module install <application>-master-<version>.oci.tar.gz
~~~

Pokud vše proběhlo bez problémů, můžeme otestovat náš první vytvořený Flatpak - pokud se vyskytly jakékoli problémy, podívejte se na konec článku, kde budou probrané základní problémy.
~~~
$ flatpak run org.supertuxproject.SuperTux 
~~~

## Flatpak a infrastruktura distribuce Fedora

Nyní máme lokálně sestavený funkční Flatpak. My ho ovšem chceme dát k dispozici i ostatním, takže jej musíme nahrát do infrastruktury Fedora. Jelikož už Supertux jako Flatpak existuje, prakticky si další kroky můžete vyzkoušet, až budete převádět vlastní aplikaci. Předtím ale potřebujeme několik věcí. Tou první je Fedora account, který si můžeme vytvořit zde: https://admin.fedoraproject.org/accounts/. Až bude účet vytvořen, vyplníme potřebné informace a naimportujeme svůj veřejný ssh klíč. Dále musíte být přidání do skupiny packager, kam vás musí pozvat někdo, kdo už v této skupině je a kdo má požadovaná práva. Pokud takového člověka neznáte, zeptejte se na https://ask.fedoraproject.org/, určitě vám někdo pomůže. 

Kromě FAS (Fedora Account System) účtu budete potřebovat ještě účet na https://pagure.io/. Ten bude propojený s vaším FAS účtem a je nutné mít na obou účtech stejné jméno. Nyní potřebujeme provést i nějaká lokální nastavení. Na Pagure si vygenerujeme API klíč a vložíme ho do `.config/rpkg/fedpkg.conf` v níže popsaném formátu, kde `<APIkey>` nahradíme za náš API klíč:
~~~
[fedpkg.pagure]
token = <APIkey>
~~~

Během vytváření účtů a nastavování občas vzniknou problémy, proto se nebojte poprosit o radu na https://ask.fedoraproject.org/, kde vám někdo pomůže.

Nyní se můžeme vrátit zpět k převodu RPM do Flatpaku. Skončili jsme ve stavu dvou připravených .yaml souborů a chystali jsme se je uploadovat do infrastruktury discribuce Fedora. 
Prvním krokem je požadavek na vytvoření repozitáře - to může trvat i několik desítek hodin, jelikož se o to starají živí lidé, kteří kontrolují validitu požadavku a podobně.
~~~
$ fedpkg request-repo --namespace=flatpaks <jmeno_aplikace>
~~~ 

Jakmile máme náš repozitář, naklonujeme si jej a přesuneme do něj naše dva soubory - doporučuji ještě do souboru README přidat krátký popis aplikace.
~~~
$ mv <jmeno_aplikace> <jmeno_aplikace>.old
$ fedpkg clone flatpaks/<jmeno_aplikace>
$ cd <jmeno_aplikace>
$ cp ../jmeno_aplikace.old/{<jmeno_aplikace>.yaml,container.yaml} .
$ git add <jmeno_aplikace>.yaml container.yaml
$ git commit -m "Initial import"
$ git push origin master
~~~

Nyní musíme nechat Fedoru, aby náš Flatpak ověřila sestavením v Koji[2](https://koji.fedoraproject.org/koji/) - opět nejdříve modul...
~~~
$ fedpkg module-build
~~~
...a poté kontejner.
~~~
$ fedpkg flatpak-build
~~~
Teď bychom měli otestovat Flatpak vytvořený na Koji, abychom ověřili, že se nic nerozbilo.
~~~
$ flatpak-module install --koji <jmeno_aplikace>:master
~~~

Posledním krokem je v Bodhi[3](https://bodhi.fedoraproject.org/updates/new) vytvořit nový update. Do pole *Candidate Builds* vložíme NVR (popsáno níže) našeho Flatpaku - pokud ho nenajdeme v terminálu v logu předchozích kroků, můžeme ho najít i v Koji. Stačí na [2](https://koji.fedoraproject.org/koji/) vyhledat název aplikace. Hledané NVR bude jedno z vrchních a bude vypadat nějak takto: mojeaplikace-20b180601144429.2. Do políčka "Update notes" stačí napsat něco jako "Initial Flatpak of <jmeno_aplikace>", *Type* zvolíme "newpackage" a stiskneme "Submit". Nyní stačí počkat, až Flatpak projde testováním. A to je vše! 

## Troubleshooting aneb co se může rozbít

Během samotné konverze nebo i během testování Flatpaku můžeme narazit na různé problémy. Níže zkusím sepsat ty, do kterých jsem nejčastěji padal - a stále padám :) - a přidám k nim i svá řešení:

1) Jak sestavit Flatpak s využitím lokální verze RPM?

Občas se může stát, že si stáhnete RPM balíček a provedete změnu v jeho spec souboru, kterou byste rádi otestovali před jejím zvěřejněním. V souboru `/etc/module-build-service/config.py` změňte `RPMS_ALLOW_REPOSITORY = False` na `RPMS_ALLOW_REPOSITORY = True` a dále na jeho konec přidejte následující text, kde nahradíte `<path to checkouts>` cestou, kde máte stažené git repozitáře RPM balíčků.
~~~
LocalBuildConfiguration.DISTGITS = {
    'https://src.fedoraproject.org': ('fedpkg clone --anonymous {}',
                                     'fedpkg --release module sources'),
    'file:///<path to checkouts>/': ('git clone file:///<path to checkouts>/{0}; git -C {0} remote set-url origin ssh://<username>@pkgs.fedoraproject.org/rpms/{0}',
                                     'fedpkg --release module sources'),
}
~~~
V yaml souboru aplikace poté přidejte k dané závislosti řádek `repository: file:///<path to checkouts>/<jmeno_zavislosti>`. Pokud bych chtěl vyzkoušet například změny v balíčku libpeas, jeho záznam v yaml souboru by vypadal takto - upozorňuji, že je nutné změnit hodnotu `ref:` na "master":  
~~~
rpms:
      libpeas:
        repository: file:///<path to checkouts>/libpeas
        rationale: Runtime dependency
        ref: master
~~~

2) Sestavení RPM balíčku havaruje na problému s manuálovými soubory

Relativně častá chyba v RPM balíčcích je uvádění manuálových souborů s příponou .gz. Ve spec souboru stačí nahradit problematickou příponu .gz za hvězdičku, jak je popsané třeba tu: [4](https://docs.fedoraproject.org/en-US/packaging-guidelines/#_manpages). 

3) Jak debugovat vytvořený Flatpak?

Existuje několik způsobů, jak vytvořený Flatpak debugovat, pokud nám stačí se podívat "dovnitř" Flatpaku - třeba když zjišťujeme, co všechno aplikace ve Flatpaku vidí a kam má přístup - stačí spustit Flatpak s flagy jako ve vzoru níže. `-d` je zkratka pro `--devel` a `--command=bash` způsobí, že se Flatpak automaticky nespustí, ale dostaneme se "dovnitř" - spustí se bash.
~~~
flatpak run -d --command=bash org.nazev.aplikace
~~~ 


[0]: https://src.fedoraproject.org/flatpaks/supertux
[1]: https://src.fedoraproject.org/
[2]: https://koji.fedoraproject.org/koji/
[3]: https://bodhi.fedoraproject.org/updates/new
[4]: https://docs.fedoraproject.org/en-US/packaging-guidelines/#_manpages
