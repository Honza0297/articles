# Z čeho se skládá Silverblue?

Tento článek je volným překladem [originálu](https://fedoramagazine.org/pieces-of-fedora-silverblue/) na fedoramagazine.org.

Fedora Silverblue je novou generací desktopového operačního systému, jehož hlavní výhodou je jeho neměnnost. V článku [Co je Silverblue?](https://mojefedora.cz/co-je-silverblue/) jste se mohli dozvědět o hlavních benefitech, které tento neměnný systém přináší. Z čeho se ale vlastně skládá? Na to si odpovíme v tomto článku.

## Souborový systém

Uživatelům klasické Fedory Workstation se může zdát jako nejtajemnější část celého projektu Silverblue právě ona "neměnnost". Co to vlastně znamená? Pár odpovědí můžeme získat, pokud se podíváme  na vlastní souborový systém. 

Na první pohled vypadá všechno tak, jako na obyčejné Fedoře. Na druhý pohled už ale můžeme najít pár rozdílů, jako například _/home_, který je ve skutečnosti symbolickým odkazem na _/var/home_.  A další odpovědi najdeme, pokud se podíváme, jak funguje samotnné libostree. To se chová, jako by celý adresářový strom byl jedním objektem. Tenhle objekt může kontrolovat v repozitáři a případně použít jeho kopii pro váš lokální stroj. 

### libostree

Projekt libostree (https://ostree.readthedocs.io/en/latest/ ) zajišťuje vše potřebné ke správě unikátního filesystému Silverblue.