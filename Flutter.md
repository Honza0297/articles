# Vyvíjejte vlastní GUI aplikace ve Flutteru na Fedoře

_Tento článek je volným překladem [anglického originálu](https://fedoramagazine.org/develop-gui-apps-using-flutter-on-fedora/)_

Když přijde řeč na frameworky pro vývoj aplikací, o Flutteru je v poslední době slyšet pořád víc. Google vypadá, že v oblasti tvorby GUI pro aplikace chce s Flutterem převzít nadvládu. Svou výpravu započal na poli mobilních aplikací, kde už je Flutter skvěle podporován. m mu pomáhá hlavní výhoda Flutteru: možnost vyvíjet multiplatformní GUI aplikace pro různá cílová zařízení - mobil, web i desktop - zatímco většina kódu je společná. 

V tomto článku si ukážeme, jak na Fedoru nainstalovat Flutter SDK a všechny potřebné nástroje a jak je používat pro vývoj aplikací se zacílením jak na smartphony, tak na web/desktopy. 

## Instalace Flutteru a Android SDK

Pro začátek si musíme obstarat samotný Flutter, Android SDK a nějaké IDE spolu s pluginy pro Flutter.

Flutter vyžaduje [kompletní instalaci Android Studia](https://developer.android.com/studio), kterou lze stáhnout z odkazu. Po rozbalení spustíme  přejdeme do složky <jmeno_rozbaleneho_archivu>/android-studio/bin a spustíme skript studio.sh. Spustí se průvodce instalací. 

~~~
cd <jmeno_rozbaleneho_archivu>/android-studio/bin
./studio.sh
~~~

Nyní nainnstalujeme Flutter SDK. Před samotnou instalací zvažte, který _release channel_ chcete využívat. Kanál _stable_ by měl mít funkční a je u něj nejmenší pravděpodobnost, že se něco pokazí. Zároveň obsahuje vše, co potřebujete, zvlášť, pokud plánujete vytvářet pouze mobilní aplikaci. 

Na druhé straně, pokud chcete používat nejnovější funkce, zvlášť pro webové a desktoppové aplikace - Flutter je stále relativně nový a nové funkce přibývají velmi rychle - zvažte poslední verze kanálů _beta_ nebo _dev_. 

Ať se rozhodnete jakkoli, volba kanálu není definitivní a může být změněna kdykoli později pomocí příkazu _flutter channel_, ke kterému se vrátíme později. 

Zamiřme tedy na [oficiální stránku s archivy Flutter SDK](https://flutter.dev/docs/development/tools/sdk/releases?tab=linux) a stáhněme si poslední instalační archiv kanálu, který se pro nás hodí nejvíc. Jelikož se jedná o obyčejné tar.xz archivy, můžeme je poté otevřít kdekoli. Stačí jen přidat složku _flutter/bin_ do proměnné PATH.

## Instalace pluginů pro do IDE

Flutter nabízí pluginy pro Visual Studio Code i Android Studio. Ve Visual Studiu je nainstalujeme přes záložku Rozšíření/Extensions, kde vybereme plugin _Flutter_. Instalace Flutteru vyžaduje zároveň plugin _Dart_. 

V Android Studiu je situace podobná, stačí otevřít _Settings/Plugins_ a tam vybrat plugin _Flutter_.


## Upgrade a udržování Flutteru

Pokud chceme ověřit, že je instalace Flutteru v pořádku a není třeba nic aktualiovat atp., použijeme nástroj _flutter doctor_.  
#TODO :) 