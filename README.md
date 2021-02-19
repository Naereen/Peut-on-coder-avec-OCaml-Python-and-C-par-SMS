# :fr: :phone: Peut on coder avec OCaml Python et C par SMS ?

Je souhaite répondre à la question suivante : peut on coder avec OCaml Python et C par SMS ?
*Spoiler alert*: **oui !**

## Quel objectif ?

Je souhaite pouvoir faire ça, depuis mon téléphone :

1. J'envoie un texto qui content « `pw:PASSWORD python: print("Hello world from Python!")` » à `0612345678` (un numéro spécifique, pas le vrai), depuis mon téléphone (sans appli, sans Internet, sans rien d'autre que des vieux SMS en GSM) ;

2. Quelques secondes plus tard, je reçois de ce numéro `0612345678` un SMS qui content « `Python:Out[1] Hello world from Python!` » ;

3. Je veux que ça marche pour des *petits* programmes en *Python 3*, *OCaml 4.05+* et *C11* ;

4. Je veux les fonctionnalités suivantes :
   - qu'il y ait ce mot de passe ;
   - qu'il y ait un numéro incrémental de cellule sortie : trois requêtes de suite seront `Out[1]: ...`, `Out[2]: ...`, `Out[3]: ...`, etc.
   - que ça fonctionne sans problème pour ces trois langages (voir plus ?) ;
 - - que l'exécution soit sécurisée, et isolée (avec [camisole](https://camisole.prologin.org/) dans [une VM](https://camisole.prologin.org/installation.html#vm-image) ?) ;

5. Premières étapes :
   - que le code soit exécuté sur *ma machine* (ou sur un serveur distant quand ce sera prêt) ;
   - je veux devoir lancer manuellement le serveur, et afficher dans une console ce qui se passe ;

- Références : <https://www.fullstackpython.com/blog/respond-sms-text-messages-python-flask.html> en anglais (lu le 2021-02-19).

## Solution ?

Ce dépôt GitHub contient un petit script Python 3 (avec un serveur [Flask](https://flask.palletsprojects.com/)) pour expérimenter et essayer cela.

Quelques questions et réponse :

1. **Quel numéro de SMS ?**
  Avec un compte **payant** sur [Twilio](https://www.twilio.com/) (voir [les prix](https://www.twilio.com/sms/pricing/fr)) ;

2. **Quel architecture logicielle ?**

   - Localement sur mon ordinateur, je vais lancer une petite application Web écrite avec [Flask](https://flask.palletsprojects.com/).
   - Cette application écoute un *webhook* local (qui peut être ouvert sur l'Internet global avec [ngrok](https://ngrok.com/)).
   - Quand un message arrive sur ce *webhook*, l'application Flask répond en renvoyant un SMS avec le résultat de l'exécution du code soumis par la requête au webhook.
   - Avec l'API de Twilio, on peut connecter cette appli (ouverte avec ngrok) au numéro de téléphone (payant) fourni par le compte Twilio.
   - Avec tout ça, je peux exécuter (et voir la sortie et le code de retour) en envoyant un SMS à ce numéro.

3. **Où ça en est ?** ~~Juste une idée pour l'instant.~~ C'était une idée le vendredi 19 février vers 13h, c'était quasiment terminé le soir même !

TODO: la suite n'est pas terminée !

## Exemples

### Usage simple

1. [Installer ce projet](#Installation),
2. Créer un compte [Twilio](https://www.twilio.com/try-twilio),
3. Lancer le serveur avec `make local`, tester le,
4. Si ça marche, essayez le serveur distant avec `make ngrok` et allez ajouter l'adresse ngrok du webhook dans [le panneau de contrôle Twilio](https://www.twilio.com/console/phone-numbers),
5. Tester avec un exemple !
6. Soyez tout content :+1: ! Et ajouter une [petite étoile](https://github.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS/stargazers) :star: à ce projet !

Cela va vous envoyer un texto contenant la réponse de l'exécution de ce programme, si tout est bien configuré.

![screenshots/example1.png](screenshots/example1.png)

### Tester votre configuration

Essayez les SMS suivants :

```text
Input: test
Output: It works!
```

```text
Input: Hello
Output: Hello back to you from Python!
```

```text
Input: Bonjour
Output: Bien le bonjour depuis Python !
```

```text
Input: Languages?
Output: List of supported languages are: c, ocaml,python
```

```text
Input: Langages ?
Output: La liste des langues prises en charge est : c, ocaml, python
```

> L'ordre de réponse dans les langages peut changer.

TODO: make screenshots of this!

### Aide

Tout se fait avec un Makefile, donc l'aide aussi :

```bash
$ make help
Help for utilities (by Lilian BESSON, https://github.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS.git)
Please use 'make <target>' where <target> is one of
  setupvenv  to set-up and install requirements in a Python3 virtualenv
  server     to do everything
  local      to do local only
  test_hello_api connects and tests Camisole backend
  test_api   connects and tests Camisole backend
  notify     to notify that the server is ready
  clean      to clean the temp files.
  ngrok      if the local server is ready, open it to the world with https://ngrok.com
```

### Cas d'échec

Le script est traduit en français et anglais, et il affiche des messages d'erreurs clairs selon les causes d'échec.

----

## Installation

> Uniquement testé sur ma machine, avec Ubuntu 18.04.
>
> - Ca ne marchera probablement PAS sous Windows ou Mac.
> - Ca marchera peut-être sous d'autres GNU/Linux, ou *NIX-like.

### Prérequis !

Suivez ces instructions :

- Il faut avoir Python 3, et `virtualenv` (dans la librairie standard) ;
- Il faut avoir `curl` et GNU `make` (normalement présent, sinon `sudo apt get curl make`) ;
- Il faut suivre les instructions [d'installation de Camisole](https://camisole.prologin.org/installation.html), avec l'option la plus sécurisée qui est avec une VM ;
- Démarrez la VM, connectez-vous, changez le mot de passe (`$ passwd`, ancien mot de passe, nouveau mot de passe deux fois) ;
- Depuis votre bureau, vérifiez que <http://localhost:42920/> est bien accessible, que <http://localhost:42920/languages> donne une liste de langages avec au moins Python, C et OCaml, et que <http://localhost:42920/system> est cohérent ;
- Faites `make test_api` pour vérifier que la VM peut bien exécuter des petits codes Python, OCaml et C ;
- Si tout est bon, étape suivante !

### Cette application Flask

Facile !
Cloner ce dépôt, aller dans le dossier, et utilisez le directement, sans le copier ailleurs.

- *Remarque* : si le mot de passe n'existe pas encore, il faudra le créer lors du premier lancement du service. Le [Makefile](./Makefile) automatise cela, mais si jamais :

   ```bash
   echo "password" | base64 > .password.b64
   ```

- Première exécution :

   ```bash
   cd /tmp/
   git clone https://GitHub.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS
   cd Peut-on-coder-avec-OCaml-Python-et-C-par-SMS/
   # set-up and install everything in a Python3 virtualenv
   make setupvenv
   # starts the Flask API, connects to and tests Camisole backend, and notify you of success
   make local
   # check that it works by going to http//localhost:5000 it should say hi and direct you to your https://www.twilio.com/console/ dashboard, and activate redirect to http://CHANGE.io
   ```

   Maintenant, si tout a bien marché, ouvrez votre navigateur sur <http://localhost:42920/test/python>, <http://localhost:42920/test/ocaml> ou <http://localhost:42920/test/c> pour tester l'exécution de code via Camisole.
   Si ça marche, vous êtes prêt-e à passer à l'étape suivante :

- Exécutions suivantes :

   ```
   # if this works, kill it, and restart with launching ngrok
   make server
   # add ngrok webhook to https://www.twilio.com/console/phone-numbers
   # test it, using phone number!
   ```

   Maintenant, si tout a bien marché, ouvrez votre navigateur sur <https://TRUC.ngrok.io/test/python>, <https://TRUC.ngrok.io/test/ocaml> ou <https://TRUC.ngrok.io/test/c> pour tester l'exécution de code via Camisole, depuis n'importe OU sur Internet !
   **Gardez ce lien toujours privé !**
   Si ça marche, vous êtes prêt-e à passer à l'étape suivante :

### Connexion avec Twilio

> Lisez l'article <https://www.fullstackpython.com/blog/respond-sms-text-messages-python-flask.html> pour plus de détails.

- Créez un compte d'essai Twilio. Il faut un mail valide, et un numéro de téléphone valide.

- Créer un numéro de téléphone Twilio. Il faut remplir quelques informations légales et téléverser une preuve d'identité, e.g., un passeport (ou un échantillon sanguin 🤔).

- Accédez à [l'écran de gestion des numéros de téléphone](https://www.twilio.com/console/phone-numbers) et cliquez sur le numéro de téléphone que vous souhaitez configurer pour répondre aux messages texte entrants.

- Faites défiler vers le bas jusqu'à près du bas de la page et recherchez l'en-tête "Messagging". Modifiez la zone de texte "A Message Comes in" afin qu'elle ait votre URL de transfert ngrok plus la route "/twilio", comme indiqué dans cette capture d'écran.

![respond-sms-python-flask/number-configuration.png](https://www.fullstackpython.com/img/160530-respond-sms-python-flask/number-configuration.png)

- Relancez l'appli Flask, tout en ayant encore la VM Camisole ouverte, évidemment :

   ```
   # if this works, kill it, and restart with launching ngrok
   make server
   # test it, using phone number!
   ```

   Maintenant, si tout a bien marché, 🎉 vous pouvez envoyer un SMS au format suivant au numéro Twilio, et l'appli Flask va vous répondre, en passant par le tunnel ngrok !

   ```
   pw:PASSWORD python: print("Hello world from Python!")
   ```

   ```
   pw:PASSWORD ocaml: print_endline "Hello world from OCaml!"
   ```

   ```
   pw:PASSWORD c: #include <stdio.h>;\n/* Say hello */\nint main(void) {\nprintf("Hello world from C!");\nreturn 0;\n}"
   ```

   TODO: format SMS possible sur plusieurs lignes, pour le C notamment ?

   TODO: capture d'écran réussite !


TODO: je n'ai pas encore pu tester cette partie, mais je le fais dès que mon numéro Twilio aura été activé !


> Si quelque chose ne fonctionne pas bien, merci [de signaler un problème](https://github.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS/issues/new) :clap: !

### D'autres trucs

- Au lancement de l'appli Flask, le programme vérifie que le mot de passe (encodé en base64 comme un fichier local) est bien présent dans `.password.b64` : ne le donnez à personne, ne l'envoyez pas sur un Git, ou [en ligne](https://perso.crans.org/besson/publis/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS.git/.password.b64) ;
- Au lancement, l'appli teste pour voir que la connexion avec la VM Camisole fonctionne bien, et qu'elle est capable d'exécuter du code Python, OCaml et C .
- Quand quelque chose se passe mal, la console dans laquelle on a lancé l'appli Flask affiche plein de choses. Essayez de régler ça vous même, sinon [ouvrez un ticket !](https://github.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS/issues/new).
- Malgré la VM, et les précautions, ce n'est PAS DU TOUT SÉCURISÉ ! Ne testez pas les limites du système, je ne saurai tenu responsable de RIEN !
- Regardez [`Tests.md`](./Tests.md) pour plus d'informations sur des entrées/sorties qui devraient être sécurisées ! TODO: tester ça !

---

## Que reste-t-il à faire ?

### :boom: TODO: Tester en vrai !

### :boom: TODO

- More tests from [`json_tests/`](json_tests/) folder ;
- Automate creation of `.json` files from `.python`, `.ocaml`, `.c` files ;
- When reading `input()` for password, use a "hidden" input like real password on UNIX ? Useless, but fun to try!
- Allow any language supported by Camisole (Ada, C, C#, C++, D, Go, Haskell, Java, Javascript, Lua, OCaml, PHP, Pascal, Perl, Prolog, Python, Ruby, Rust, Scheme) ?
- Clean up code ? It's already not bad.

### More TODO?

- Write a wrapper script like `run-camisoled`, that can read a file in Python/OCaml/C, and safely pass it to Camisole VM, and pretty-print its JSON results! For my teaching next year this would be veryyy useful!

### Avec `pip` ? Non.

Ce projet ne sera **pas** distribué sur [le dépôt de packet Pypi](https://pypi.org/), et je ne souhaite pas qu'il puisse être installé directement depuis GitHub avec [`pip`](http://pip.pypa.io/).

![PyPI implementation TODO](https://img.shields.io/pypi/implementation/lempel_ziv_complexity.svg)

----

## Comparaison à d'autres projets

> Si je me suis ~~embêté~~ amusé à faire ça moi-même, croyez bien que c'est parce que je n'ai rien trouvé d'équivalent !
> Mais je suis curieux, [envoyez moi un mail](https://perso.crans.org/besson/callme.fr.html) ou [ouvrez un ticket](https://github.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS/issues/new) si vous connaissez une autre solution !

- TODO

----

## À propos :notebook:

### Langage et version(s) ?

Écrit en [Python v3.6+](https://www.python.org/3/) (version CPython), et bien en se basant sur de super projets libres et gratuits :

- Avec [Flask v1.1](https://flask.palletsprojects.com/en/1.1.x/) ;
- Avec [l'API officielle Python de Twilio](https://www.twilio.com/docs/libraries/python) (gratuit, mais payant pour avoir un numéro de téléphone, évidemment !) ;
- Avec [Camisole v1.0](https://camisole.prologin.org/) de [Prologin](https://prologin.org/) ;
- Avec [ngrok](https://ngrok.com/) ;
- Et avec amour !

### :scroll: Licence ? [![GitHub licence](https://img.shields.io/github/license/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS.svg)](https://github.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS/blob/master/LICENSE)

Ce script et cette documentation sont distribuées en accès libre selon les conditions de la [licence MIT](https://lbesson.mit-license.org/) (cf le fichier [LICENSE](LICENSE) en anglais).
© [Lilian Besson](https://GitHub.com/Naereen), 2021.

[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/Naereen/Peut-on-coder-avec-OCaml-Python-et-C-par-SMS/graphs/commit-activity)
[![Demandez moi n'importe quoi !](https://img.shields.io/badge/Demandez%20moi-n'%20importe%20quoi-1abc9c.svg)](https://GitHub.com/Naereen/ama.fr)
[![ForTheBadge uses-badges](http://ForTheBadge.com/images/badges/uses-badges.svg)](http://ForTheBadge.com)
[![ForTheBadge uses-git](http://ForTheBadge.com/images/badges/uses-git.svg)](https://GitHub.com/)
[![forthebadge made-with-python](http://ForTheBadge.com/images/badges/made-with-python.svg)](https://www.python.org/)
[![ForTheBadge built-with-swag](http://ForTheBadge.com/images/badges/built-with-swag.svg)](https://GitHub.com/Naereen/)
