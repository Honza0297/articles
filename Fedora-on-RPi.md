# Rozjeďte Fedoru Workstation na svém Raspberry Pi 4!

Raspberry Pi, malý jednodeskový počítač od stejnojmenného výrobce, zná alespoň podle jména skoro každý. V současné době se můžete nejvíc setkat se dvěma verzemi. První z nich je Raspberry Pi 3, které bylo představené v roce 2016 s 1 GB RAM. Klasickou Fedoru Workstation na něm sice rozjedete, ale nečekejte žádnou extra rychlost. Na druhou stranu, pro Fedoru IoT se bude hodit víc než dobře. Druhá verze, Raspberry Pi 4, může být k vidění hned se čtyřmi velikostmi RAM: 1(už se neprodává), 2, 4 a dokonce 8 GB RAM. Cokoli lepšího než 1 GB vám už dá slušnou plynulost, se kterou půjde pracovat bez větších problémů. 

V článku si představíme, jak na čtvrtou malinu dostat klasickou Fedoru Workstation. Pokud byste přeci jen chtěli vyzkoušet Fedoru i na třetí verzi, postup bude prakticky totožný. Mnohem lepší ale bude využít ho pro Fedoru IoT - novou edici, která je určená především pro nasazení kontejnerizovaných aplikací. Instalaci Fedory IoT na Raspberry Pi 3 si ukážeme v některém z příštích článků.  

## Co budeme potřebovat:

* Raspberry Pi 4 (doporučuji verzi s minimálně 2 GB RAM)
* SD kartu o velikosti ideálně 32 GB (pravděpodobně bude stačit i poloviční, ale to se dostáváme opravdu na hranu použitelnosti)
* Napájecí kabel pro Raspberry Pi (USB C konektor, ideálně 3 A při 5 V, ještě lépe originální zdroj)
* Monitor s HDMI vstupem
* microHDMI-to-HDMI kabel
* Klávesnici a myš
* Další počítač, na kterém připravíme obraz na SD kartu (návod je primárně určený pro počítače s Fedorou, ale postup tvorby obrazu bude podobný i na dalších systémech včetně Windows) - počítač musí mít čtečku SD karet, jinak budeme potřebovat navíc také externí čtečku

## Stažení Fedora Media Writeru

Ze všeho nejdříve si musíme obstarat Fedora Media Writer, který pro nás na SD kartu zapíše obraz, do kterého následně v Raspberry Pi nabootujeme. Pokud používáme Fedoru Workstation, Media Writer nainstalujeme pomocí

~~~
$ sudo dnf install mediawriter
~~~

A pokud používáme Fedoru Silverblue (nebo jakýkoli jiný systém, který používá flatpaky), Media Writer získáme takto:

~~~
$ flatpak install mediawriter
~~~

Pokud používáte Windows (nebo MacOS), i na vás vývojáři mysleli. Fedora Media Writer si můžete stáhnout z  https://getfedora.org/cs/workstation/download/.

## Stažení obrazu Fedory 33

Následně z [webu Fedory](https://getfedora.org/en/workstation/download/) stáhneme raw image Fedory 33 pro aarch64. Až se obraz stáhne, otevřeme Fedora Media Writer. Na úvodní obrazovce klikneme ne "Custom image" a ve vyskakujícím okně vybereme námi stažený soubor. Poté zvolíme správnou SD kartu (tenhle krok si opravdu dobře ověřte, ať nezapíšete Fedoru někam, kam nechcete). Nejjednodušší způsob, jak identifikovat správnou kartu, je mít v době zápisu připojenou jen jednu. Pokud jich ale máte připojených více, stačí kartu odpojit a znovu připojit k počítači - karta, která v rolovací nabídce zmizí a poté se znovu objeví, bude ta správná. Jako jako zařízení zvolme "Other". Nyní stačí počkat, až Media Writer zapíše Fedoru na SD kartu. 

## Připojení periferií a vložení SD karty

Mezitím si můžeme k Raspberry připojit klávesnici, myš a propojit ho s monitorem - pro připojení doporučuji použít HDMI0 výstup (ten blíže k napájení), i přes to, že by měly fungovat oba stejně. 

Až se obraz zapíše, stačí kartu vložit do Raspberry Pi a připojit napájení. Pokud jsme vše udělali správně, Na monitoru se za pár desítek vteřin objeví uvítací obrazovka Fedory s iniciálním nastavováním.


### Troubleshooting aneb co se může rozbít

Tahle sekce bude průběžně doplňovaná a bude odpovídat na nejčastější problémy s instalací. 

1. Raspberry Pi se spustí, ale na displeji vypisuje mj. "Unable to read partition as FAT".
   Raspberry říká, že nemůže přečíst SD kartu jako FAT. Problém se může vyskytnout mj. v situaci, kdy nestáhnete obraz ze stránek Fedory, ale snažíte se instalovat aarch64 Fedoru z nabídky Fedora Media Writeru. V takovém případě stačí stáhnout obraz ze stránek Fedory a použít ten (tedy postupovat přesně podle návodu).


