# How to make Flatpak from RPM


## Flatpak in general

Recently, more and more people are talking about the new FEdora edition - Silverblue. It, among other thing, does not use classic packaging system as the majority of linux disctributions (like RPM in normal Fedora), but relies on app distribution via Flatpak. 

Flatpak, like RPM, is a way how to deploy your application. However, it differs from classic packaging systems in several things. Flatpak applications are sandboxed - every app runs inside its own container - and they need explicit permissions to access host sources (like network or files). They also carry all their dependencies with them - this eliminates the incompatibility among different library versions for different applications - all has been solved by the developers.Thanks to these features, Flatpak applications are portable - the only condition is the target system has to support Flatpak 

Unfortunately, because Flatpak is relatively new technology (the first idea was formulated in 2013 and SIlverblue, which relies (almost) exclusively to Flatpak, is here with us roughly two years), not all aplications are distributed in Flatpak. Most of Flatpak applications can be found on [flathub.org](https://www.flathub.org), which becames *de facto* standard or hosting Flatpak applications. However, there is an alternative in he form of the Flatpak application repository maintained by Fedora. You can find Flatpak applications made from RPMs there and the rest of the article will be about them.


## How does "normal" Flatpak differs from the Fedora one?

The main difference is the data source."Normal" Flatpaks are build directly from source code, while Fedora Flatpaks use RPMs as their data source, which has quite interesting consequences. RPM packages are made in a transparent way under the control of the distribution and thus they can be trusted, which could not be always true with Flathub, where Flatpaks are made by volunteers. 
Next, the dependecy tree is known in advance and application has to by "only" packed. Thanks to RPM is also clear what is installed where - in Flatpak, application data are installed into /app, but it is still good to have a way how to find to which application this or that file belongs to. 
As everything, this solution has its disadvantages. Because normal packages without additional patch possibility are used, there can be found special tweaks because of Flatpaks inside their spec files, which adds several new lines into them.

## How does the conversion work? 

The general process is to make a module at first and then make a container from it. In the Fedora context, Flatpaks are only a special form of a container and they are handled in a very similar way as with server-use containers
Now, when we know everything neccessary, we can take a practical example. If we use Silverblue, create new container and enter it with these two commands:
~~~
$ toolbox create -c jmeno-kontejneru
$ toolbox enter -c jmeno-kontejneru
~~~
Next, it is neccessary to install all needed tools...
~~~
$ sudo dnf install flatpak-module-tools fedmod
~~~
...add yourself to the mock group...
~~~
$ sudo usermod -a -G mock $USER
~~~
...and install Fedora Flatpak Runtime.
~~~
$ flatpak remote-add fedora-testing oci+https://registry.fedoraproject.org#testing
$ flatpak install fedora-testing org.fedoraproject.Platform/x86_64/f31
~~~

If everything is alright, we can move to the application coversion itself. For this example we will use Supertux game- it is already present in Fedora repository [0](https://src.fedoraproject.org/flatpaks/supertux) and it is very simple, so it is a great first application to convert. Move ourselves to the folder in which we plan to convert our applications - I recommend to make a "flatpaks" folder and inside it make a folde rofr every application -  create folder for Supertux and move into it. The name of the folder should be the same as the name of future Flatpak, which should be tha same as the name of application's RPM package name, supertux in our case.
~~~
$ mkdir supertux && cd supertux
~~~
Now we generate input files for our Flatpak.
~~~
$ fedmod fetch-metadata
$ fedmod rpm2flatpak --flatpak-common --flathub=supertux supertux
~~~
First command gets metadata for our RPM packages and the second command generate initial versions of two files, which will be described later. The *--flatpak-common* options adds a dependency of  the generated module on flatpak-common module. This helps us making applications smaller and easier to build. The second option - *--flathub=supertux* - means that flathub manifest will be used for container.yaml initialization. If we make an application which is not on Flathub, we can omit this option. If the value of this option causes more found records, all are shown and we have to re-run the last command with more specific name. For example I edited the last command to use full name of flathub version of Supertux. 
~~~
$ fedmod rpm2flatpak --flatpak-common --flathub=org.supertuxproject.SuperTux  supertux
~~~


## Files <application>.yaml and container.yaml 
Now we will have a look at generated files.
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

We are most interested in the *components* part, which contains only one record - supertux - in our case. However, there can be found units up to tens of dependencies, it depends on what application we convert. The "supertux" record has several items inside: "rationale" is a short description of the component, "ref:" refers to the branch of the component in [1](https://src.fedoraproject.org/), which will be used during build - we can use whatever branch we want, but if there is no special reason current or master branch are used. The last item is "buildorder", indicating in with order will be dependencies build. Build starts with the lowest-numbered ones. If "buildorder" is not specified, is set to 0 by default. Later, we will show some other parts, but they are not needed by now. 

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
Now we will explain the "finish-args" part - we will need nothing more right now. In this section we allow the application to interact with the host system. Because supertux is only a simple gamem it does not need to use network, so we can safely delete --share-network. What exactly we can allow to our aplication can be found for example here: http://docs.flatpak.org/en/latest/sandbox-permissions.html.

## The conversion starts!

Now we are getting to theitself. We will start it with the following command:
~~~
$ flatpak-module local-build --install
~~~
This command is a shortcut for three commands - build a module, build a container and install the OCI image of the container - as I mentioned before, Fedora handles Flatpaks similarly to containers. The three commands can also be run separately - for example in the situation when we test only the permissions in the container, we do not need to rebuild the whole module.  
~~~
$ flatpak-module build-module
$ flatpak-module build-container --from-local
$ flatpak-module install <application>-master-<version>.oci.tar.gz
~~~
If everything went well, we can test our first created Flatpak - if any problems occured, look at the end of the article, where some common problems will be discussed. 
~~~
$ flatpak run org.supertuxproject.SuperTux 
~~~

## Flatpak and Fedora distribution infrastructure
Now we have locally build Flatpak, which should be working. However, we want to make it available to the others as well, so we have to upload it to the Fedora infrastructure. Because SUpertux is already present as Flatpak in Fedora, you can try the next steps once you will convert your own application. Before we can do something, we need several things to do. The first is the Fedora account, which you can create here: https://admin.fedoraproject.org/accounts/. Once the account is created, fill in all neccesary information and import your public ssh key. Next, you have to be added to the packager group. You have to be invited by someone who already is there and who has the desired permissions to invite new members. If you do not know such a person, ask at https://ask.fedoraproject.org/, somebody will help you. 

Besides FAS (Fedora Account System) account you will need also the Pagure account: https://pagure.io/. It will be connected to your FAS account and it is neccesary to have the same name on both the accounts. Now we need to make some local settings, too. On Pagure, we generate our API key and insert it into `.config/rpkg/fedpkg.conf` v the format shown below, where `<APIkey>` will be replaced with your API key:
~~~
[fedpkg.pagure]
token = <APIkey>
~~~

During creating of the account or its setting, some problems ma occur, so do not be afraid and ask on https://ask.fedoraproject.org/, where somebody will help you.

Now we can go back to the RPM to Flatpak conversion. We should have two .yaml files prepared and we were going to upload to the Fedora distribution infrastructure. First step is to send a request to create a repository. It may take even tens of hours before the repository is created because real people do this job - they have to check validity of the request etc. 
~~~
$ fedpkg request-repo --namespace=flatpaks <jmeno_aplikace>
~~~ 
Once our repository is created, we clone it and copy there our two .yaml files. I recommend also wirte a short app description into the README.md file.
~~~
$ mv <jmeno_aplikace> <jmeno_aplikace>.old
$ fedpkg clone flatpaks/<jmeno_aplikace>
$ cd <jmeno_aplikace>
$ cp ../jmeno_aplikace.old/{<jmeno_aplikace>.yaml,container.yaml} .
$ git add <jmeno_aplikace>.yaml container.yaml
$ git commit -m "Initial import"
$ git push origin master
~~~

Now we have to let Feedora infrastructure to validate our Flatpak by building it in Koji[2](https://koji.fedoraproject.org/koji/). Again, first the module...
~~~
$ fedpkg module-build
~~~
...and then the container.
~~~
$ fedpkg flatpak-build
~~~
Now we should test the Flatpak which wa sbuild in koji to check whether everything alright. 
~~~
$ flatpak-module install --koji <jmeno_aplikace>:master
~~~
The last step is to make new update in Bodhi[3](https://bodhi.fedoraproject.org/updates/new). To the field *Candidate Builds* insert NVR (described below) of our Flatpak - if we cannot find it in the log of the previous steps, we can find in in Koji. Go to https://koji.fedoraproject.org/koji/ and search for the name of the application. The NVR we are looking for should be one of the last record and should look like myapplication-20b180601144429.2. To the "Update notes" field just write something like "Initial Flatpak of <application_name>", choose "newpackage" as a *Type* and hit "Submit". Now just wait until Flatpak pass the testing. It takes one week when everyboddy can download and test your Flatpak and give you karma - positive if everything seems alright, or negative if something does not work, which is a signal for you that something needs fix. 
And that's it!

## Troubleshooting or what can go wrong

During the conversion itself or during the Flatpak testing, you can face many problems. Below I will try to write down the ones I was falling into  - and I still fall :) - most often and try to add my solutions to them:

1) How to build Flatpak with local version of RPM?

Sometimes, it can happen that you will download RPM package a edit it. Then you probably want to test the change before its publication. It is quite easy to do so. In file `/etc/module-build-service/config.py`  change `RPMS_ALLOW_REPOSITORY = False` to `RPMS_ALLOW_REPOSITORY = True` and to the end of the file add following text where `<path to checkouts>` will be the path where you have your downloaded git repositories of the RPM packages.
~~~
LocalBuildConfiguration.DISTGITS = {
    'https://src.fedoraproject.org': ('fedpkg clone --anonymous {}',
                                     'fedpkg --release module sources'),
    'file:///<path to checkouts>/': ('git clone file:///<path to checkouts>/{0}; git -C {0} remote set-url origin ssh://<username>@pkgs.fedoraproject.org/rpms/{0}',
                                     'fedpkg --release module sources'),
}
~~~
In the yaml file then add this line to the given dependency: `repository: file:///<path to checkouts>/<dependency_name>`. If you would like to try changes in libpeas RPM, its record would look like this - notice that it is necessary to change value of `ref:` to "master":
~~~
rpms:
      libpeas:
        repository: file:///<path to checkouts>/libpeas
        rationale: Runtime dependency
        ref: master
~~~

2) Build of RPM fails because of manual files problem:

Relatively common mistake in RPM packages is listing them with the .gz 	suffix. In the spec files just replace problematic suffix .gz with star. You can see Packaging Guidelines for more info: https://docs.fedoraproject.org/en-US/packaging-guidelines/#_manpages. 

3) How to debug created FLatpak?

There are several ways how to debug created Flatpak. If you just need to "look inside" the Flatpak - for example when you want to know what files the application can see and where it has access - just run the Flatpak with flags like the example below. `-d` is a shortcut for `--devel` and `--command=bash` prevents Flatpak to run application, but bash is run instead, so you can move inside the Flatpak. 
~~~
flatpak run -d --command=bash org.nazev.aplikace
~~~ 

[0]: https://src.fedoraproject.org/flatpaks/supertux
[1]: https://src.fedoraproject.org/
[2]: https://koji.fedoraproject.org/koji/
[3]: https://bodhi.fedoraproject.org/updates/new
[4]: https://docs.fedoraproject.org/en-US/packaging-guidelines/#_manpages
