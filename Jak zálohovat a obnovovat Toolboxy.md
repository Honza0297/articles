# Jak zálohovat a obnovovat toolbox kontejnery

Článek je přeložený a lehce upravený z [originálu](https://fedoramagazine.org/backup-and-restore-toolboxes-with-podman/) na [fedoramagazine.org](https://fedoramagazine.org). 

[Toolbox](https://fedoramagazine.org/a-quick-introduction-to-toolbox-on-fedora/), úzce spjatý s [Fedorou Silverblue](https://mojefedora.cz/co-je-silverblue/) je primárně používaný jako nástroj pro jednoduchou tvorbu snadno vytvořiltelných a zrušitelných kontejnerů. Životní cyklus takového typického kontejneru vypadá asi takhle: vytvořit kontejner, nainstalovat nezbytnosti, udělat svou práci v relativním bezpečí kontejneru a po dokončení práce se kontejneru čistě zbavit. Žádné riziko pro hostující systém a žádné reziduální knihovny, které by v systému mohly strašit ještě celé věky. 

Proč by tedy někdo chtěl zálohovat některý ze svých toolbox kontejnerů? Občas totiž mohou mít i mnohem trvalejší charakter, zdlouhavé a pracné prvotní nastavování nebo jsou používané pro kritické aplikace. Občas se mohou hodit i v klasické Fedoře Workstation, ovšem naprosto klíčovou roli hrají v nové Fedoře Silverblue, kde nabízejí jednoduchou možnost, jak instalovat a používat různé aplikace bez nutnosti layerování balíčků přes rpm-ostree. Jedním příkladem může být například nastavení vývojářského prostředí, které zahrnuje vše od hardwarových ovladačů přes různé aplikace a toolchainy až po nastavení terminálu. Druhým příkladem může být spouštění alikace, pro kterou zatím neexistuje [flatpak](https://www.flatpak.org/). V obou takovýh příkladech může rozbitý toolbox kontejner zapříčinit poměrně výraznou ztrátu času. Pokud ovšem existuje záloha, můžeme Toolbox rychle obnovit, nebo dokonce přesunout na jiný počítač!

Proces, který si dále vysvětlíme, využívá [Podman](https://podman.io/), který je v pozadí využíván i samotným Toolboxem. Nejprve je vytvořen obraz existujícího kontejneru, který je následně uložený do archivu. Jeho obnovení probíhá tak, že se načte obraz z archivu a následně se vytvoří nový toolbox kontejner z tohoto obnoveného obrazu. Nový kontejner bude tedy naprosto identickou kopií toho původního. 

Ještě před vysvětlením si dovolím upozornit, že tato metoda **nezálohuje data**, ale pouze to, co je v kontejneru nainstalované.

## Vytváření zálohy

Pro vytvoření zálohy budeme potřebovat jeho jméno a ID kontejneru, oboje zjistíme pomocí příkazu *toolbox list*. V ukázce si vše budeme ukazovat na velmi originálně pojmenovaném kontejneru *priklad*.

~~~
$ toolbox list
CONTAINER ID  CONTAINER NAME     CREATED        STATUS      IMAGE NAME
ca108b141a9a  priklad            8 seconds ago  running  registry.fedoraproject.org/f32/fedora-toolbox:32
~~~

Pokud je status kontejneru *running*, tedy stejný, jako v ukázce, měli bychom ho nejprve zastavit pomocí následujícího příkazu, kde *jmeno* je jméno kontejneru, v našem případě *priklad*. 

~~~
podman container stop jmeno
~~~

Vytvoření obrazu z kontejneru se provede pomocí následujícího příkazu, kde za ID kontejneru dosadíme ID kontejneru (které bude jiné než v příkladu). Jméno zálohy si můžeme vybrat liovolně.

~~~
podman container commit -p ID_kontejneru jmeno_vytvoreneho_obrazu
~~~

Abychom si zkontrolovali, že se obraz skutečně vytvořil, můžeme si vše ověřit pomocí příkazu *toolbox list*, kde by se náš obraz měl objevit. V našem případě se bude záloha jmenovat opět velmi originálně, tedy *zaloha*.

~~~
[jaberan@localhost ~]$ toolbox list
IMAGE ID      IMAGE NAME                                        CREATED
cdb1d2d2763f  localhost/zaloha:latest                           40 seconds ago
~~~

Nyní pomocí Podmanu vytvoříme archiv se zálohou:

~~~
podman save -o zaloha.tar zaloha
~~~

Pomocí *ls* si můžeme ověřit, že se archiv skutečně vytvořil. Jelikož obraz už nebudeme potřebovat, smažeme ho:

~~~
podman rmi zaloha
~~~

V tuto chvíli tedy máme archiv, který můžeme libovolně přesouvat a případně také znovu rozbalit, což si ukážeme dále:


## Obnova Toolboxu

Teoretický postup bude přesně opačný, níže si postupně probereme jednotlivé kroky. Nejprve musíme vytvořit obraz z archivu:

~~~
podman load -i zaloha.tar
~~~

Pomocí *toolbox list* si opět ověříme, že se obraz vytvořil:

~~~
[jaberan@localhost ~]$ toolbox list
IMAGE ID      IMAGE NAME                                        CREATED
cdb1d2d2763f  localhost/zaloha:latest                           10 minutes ago
~~~

Nyní si vytvoříme kontejner z obnoveného obrazu. Volba -c nám specifikuje jméno nového toolbox kontejneru, -i nám určuje obraz, který musíme specifikovat kompletním jménem.

~~~
toolbox create -c novy_priklad -i localhost/zaloha:latest
~~~

Pomocí nám už známého *toolbox list* můžeme snadno ověřit, že se toolbox vytvořil:

~~~
[jaberan@localhost ~]$ toolbox list
CONTAINER ID  CONTAINER NAME     CREATED         STATUS      IMAGE NAME
55f95fa21684  novy_priklad       7 seconds ago   configured  localhost/zaloha:latest
~~~

Nakonec můžeme nový kontejner otestovat:

~~~
toolbox enter -c novy_priklad
~~~

Pokud se všechno povedlo, nyní vstoupíte do nového kontejneru, ve kterém najdete všechno, co jste nainstalovali do toho původního. 