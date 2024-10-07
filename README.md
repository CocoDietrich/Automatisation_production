# DIETRICH Corentin et CHAUVEL Clément

## Explication du fichier ci.yml permettant d'effectuer les tests PHPunit :

#### Dans un premier temps, on configure les conditions de démarrage et l'environnement des jobs.

### 1. Nom de l'action
```
name: Run PHPUnit Tests with Code Coverage
```
Explication : Le workflow est nommé "Run PHPUnit Tests with Code Coverage". Ce nom permet d'identifier cette action dans l'onglet GitHub Actions, et indique que cette action exécute des tests PHPUnit avec un rapport de couverture de code.

### 2. Déclenchement de l'action

```
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

```
Explication : Cette section décrit les événements qui déclenchent l'exécution de ce workflow.
- push sur la branche main : L'action est déclenchée chaque fois qu'un commit est poussé vers la branche principale (main).
- pull_request vers la branche main : Le workflow s'exécute également chaque fois qu'une pull request est ouverte ou mise à jour vers la branche main.

### 3. Définition du job

```
jobs:
  phpunit:

```
Explication : Un job nommé phpunit est défini. Ce job contient toutes les étapes nécessaires pour exécuter les tests PHPUnit et générer le rapport de couverture de code.

### 4. Environnement d'exécution

```
    runs-on: ubuntu-latest

```
Explication : Le job sera exécuté sur un environnement Linux utilisant la dernière version d'Ubuntu, ce qui garantit que le code est testé dans un environnement standardisé.

### 5. Stratégie de version PHP

```
    strategy:
      matrix:
        php-versions: [8.1]

```
Explication : Cette section utilise une matrice pour permettre l'exécution du workflow sur plusieurs versions de PHP. Dans ce cas, seule la version 8.1 de PHP est utilisée, mais vous pourriez ajouter d'autres versions si vous souhaitez tester la compatibilité.

#### Une fois toutes les configurations terminées, on peut effectuer les tests, le coverage, générer le Github Summary...

### Étape 1 : Récupération du code
```
      - name: Checkout code
        uses: actions/checkout@v3

```
Explication : Cette étape utilise l'action actions/checkout@v3 pour récupérer le code source du repository dans l'environnement de travail.

### Étape 2 : Installation de PHP et des extensions
```
      - name: Set up PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-versions }}
          extensions: mbstring, dom, xml, json, libxml

```
Explication : Cette étape installe PHP (version 8.1 ici) et les extensions nécessaires pour exécuter les tests. L'action shivammathur/setup-php@v2 est utilisée pour gérer l'installation de PHP et des extensions : mbstring, dom, xml, json, et libxml.

### Étape 3 : Installation des dépendances
```
      - name: Install dependencies
        run: composer install --working-dir=PrivateBin-main

```
Explication : Ici, les dépendances du projet définies dans le fichier composer.json sont installées via Composer. On spécifie le répertoire de travail, ici PrivateBin-main, où se trouve le fichier composer.json.

### Étape 4 : Exécution de PHPUnit avec génération de la couverture de code
```
      - name: Run PHPUnit with Code Coverage
        run: |
          cd PrivateBin-main
          ./vendor/bin/phpunit --coverage-cobertura=coverage_cobertura.xml

```
Explication : Cette étape exécute les tests PHPUnit avec un rapport de couverture de code au format Cobertura. Le rapport est généré et stocké dans le fichier coverage_cobertura.xml.
- cd PrivateBin-main : Change le répertoire de travail pour PrivateBin-main, là où se trouvent les fichiers du projet.
- ./vendor/bin/phpunit --coverage-cobertura=coverage_cobertura.xml : Exécute les tests en utilisant PHPUnit et génère un rapport de couverture de code au format Cobertura, stocké dans le fichier coverage_cobertura.xml.
### Le fichier coverage_cobertura.xml est généré localement et stocké dans le répertoire PrivateBin-main.

### Étape 5 : Téléchargement du rapport de couverture
```
      - name: Upload coverage report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: PrivateBin-main/coverage_cobertura.xml

```
Explication : Cette étape sauvegarde le fichier de rapport de couverture coverage_cobertura.xml en tant qu'artefact dans GitHub Actions. Cela permet de récupérer ce fichier après l'exécution du workflow pour analyse ou partage.

### Étape 6 : Vérification du fichier de coverage
```
      - name: Check coverage report file
        run: cat PrivateBin-main/coverage_cobertura.xml

```
Explication : La commande cat affiche le contenu du fichier coverage_cobertura.xml directement dans les logs de l'exécution GitHub Actions. Cela permet de vérifier que le fichier a bien été généré et de voir un aperçu du rapport de couverture. Cette étape n'est pas obligatoire mais permet de vérifier que le fichier est bien stocké.

### Étape 7 : Génération d'un résumé de la couverture de code dans GitHub Summary
```
      - name: Code Coverage Summary Report
        uses: irongut/CodeCoverageSummary@v1.3.0
        with:
          filename: PrivateBin-main/coverage_cobertura.xml
          badge: true
          format: markdown
          output: both

```
Explication :
- ```irongut/CodeCoverageSummary@v1.3.0``` : Utilisation de cette action pour analyser le fichier de couverture Cobertura et générer un résumé dans la section Summary de GitHub Actions.
- ```filename``` : PrivateBin-main/coverage_cobertura.xml : Le fichier de couverture utilisé est coverage_cobertura.xml généré à l'étape précédente.
- ```badge: true``` : Indique que l'action doit générer un badge de couverture de code, utile pour un affichage visuel de la couverture dans les README ou dans GitHub Actions.
- ```format: markdown``` : Le résumé est généré au format Markdown, compatible avec le rendu dans GitHub.
- ```output: both``` : L'output est généré à la fois dans la section "Summary" et dans les logs.

### Étape 8 : Générer et afficher le résumé du code coverage
```
      - name: Generate and Display Code Coverage Summary
        run: |
          echo "### Code Coverage Summary" >> $GITHUB_STEP_SUMMARY
          echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
          ./vendor/bin/phpunit --coverage-text --colors=never >> $GITHUB_STEP_SUMMARY
          echo "\`\`\`" >> $GITHUB_STEP_SUMMARY

```
Explication :
Cette étape génère un rapport de couverture de code au format text (--coverage-text) et l'ajoute au résumé de l'exécution GitHub Actions en utilisant la variable spéciale $GITHUB_STEP_SUMMARY.
- ```echo "### Code Coverage Summary" >> $GITHUB_STEP_SUMMARY``` : Ajoute un titre au résumé dans le format Markdown.
- ```echo "\``" >> $GITHUB_STEP_SUMMARY``` : Ajoute des balises pour formater le texte du rapport en tant que bloc de code dans Markdown.
- ```./vendor/bin/phpunit --coverage-text --colors=never >> $GITHUB_STEP_SUMMARY``` : Exécute PHPUnit pour générer un rapport de couverture en texte brut et l'ajoute directement dans le résumé.
- ```echo "\``" >> $GITHUB_STEP_SUMMARY``` : Ferme le bloc de code Markdown.

### Etape 9 : Déployer le site sur un serveur FTP
```
- name: Sync files
  uses: SamKirkland/FTP-Deploy-Action@v4.0.0
  with:
    server: ${{ secrets.FTP_URL }}
    username: ${{ secrets.LOGIN }}
    password: ${{ secrets.MDP }}
    local-dir: 'PrivateBin-main/'
    server-dir: 'www/'
    exclude: "[**/.git/**, **/vendor/**]"

```
Paramètres importants :
- ```server: ${{ secrets.FTP_URL }}``` :
Le paramètre server spécifie l'URL du serveur FTP auquel se connecter. Il utilise une variable de secret GitHub appelée FTP_URL, qui est stockée en toute sécurité dans les secrets du dépôt. Cela protège les informations sensibles comme les adresses de serveurs.
- ```username: ${{ secrets.LOGIN }}``` :
Il spécifie le nom d'utilisateur FTP pour se connecter au serveur, également stocké sous forme de secret (LOGIN) dans GitHub pour des raisons de sécurité.
- ```password: ${{ secrets.MDP }}``` :
Ce champ contient le mot de passe FTP (référence à la variable secrète MDP), utilisé pour authentifier la connexion FTP. Ici encore, il est sécurisé dans les secrets GitHub.
- ```local-dir: 'PrivateBin-main/'``` :
Indique le répertoire local qui doit être synchronisé avec le serveur distant. Ici, il s'agit du répertoire PrivateBin-main/, qui contient les fichiers à déployer.
- ```server-dir: 'www/'``` :
Ce paramètre définit le répertoire cible sur le serveur distant. Ici, il s'agit du répertoire www/, où les fichiers seront téléchargés ou mis à jour.
- ```exclude: "[**/.git/**, **/vendor/**]"``` :
Ce paramètre exclut certains fichiers ou répertoires du déploiement. Ici, les dossiers .git/ (les fichiers de contrôle de version) et vendor/ (les dépendances PHP gérées par Composer) ne sont pas synchronisés avec le serveur.

Explication :

Cette étape du workflow permet de déployer automatiquement les fichiers du projet (situés dans PrivateBin-main/) vers un serveur FTP distant dans le répertoire www/. En excluant certains dossiers (comme .git et vendor), seuls les fichiers nécessaires au fonctionnement de l'application ou du site seront transférés. Cela permet une mise à jour continue du site web via GitHub Actions à chaque modification du code source.

Grâce à l'utilisation des secrets GitHub, les informations sensibles comme le serveur FTP, le nom d'utilisateur et le mot de passe restent protégées, même si le fichier YML est public.

### En résumé
Après l'exécution du workflow, le rapport de coverage sera visible sous forme de texte formaté directement dans le résumé du job sur la page GitHub Actions.
Ainsi, le fichier ci.yml permet non seulement de générer le rapport et de le stocker en fichier, mais aussi de le lire facilement directement dans l'interface GitHub Actions.

## Les problèmes auxquels nous avons fait face

### 1. Erreur : "Unable to resolve action irongut/code-coverage-summary"
- Problème : Au début, il y avait une erreur lors de l'utilisation de l'action ```irongut/code-coverage-summary```. Le message d'erreur indiquait que la version ou l'action n'était pas trouvée.
- Cause : La version ou l'action demandée n'existait pas sous ce nom ou elle avait été mal spécifiée dans le fichier YML.
- Solution : Nous avons confirmé que l'action correcte était ```irongut/CodeCoverageSummary@v1.3.0```. En changeant la version et en utilisant cette action correctement, cela a permis de résoudre le problème.

### 2. Erreur de format "reporter" dans PHPUnit
- Problème : Lorsque nous avons essayé d'utiliser un autre format de rapport pour la couverture de code, nous avons reçu l'erreur Input variable 'reporter' is set to invalid value 'cobertura' et plus tard pour 'clover' également.
- Cause : PHPUnit ne supportait pas les formats non pris en charge par l'action utilisée pour résumer le rapport de couverture, ou bien le mauvais format de sortie était utilisé dans les configurations.
- Solution : Nous avons ajusté le format de génération de couverture avec ```--coverage-cobertura``` pour spécifier le bon format de rapport (Cobertura) et avons utilisé l'outil correspondant pour la lecture de ce fichier dans l'action GitHub.

### 3. Téléchargement et stockage de l'artifact de couverture de code
- Problème : Il y avait confusion quant à la localisation du fichier de couverture généré (coverage_cobertura.xml). On a initialement pensé qu'il restait stocké localement dans le répertoire PrivateBin-main.
- Cause : Ce fichier est d'abord généré localement, mais il est ensuite téléversé dans GitHub comme un artifact, et non seulement localement sur la machine virtuelle.
- Solution : L'utilisation de l'action ```actions/upload-artifact@v3``` a permis de stocker le fichier de couverture en tant qu'artifact GitHub, disponible pour téléchargement depuis l'interface GitHub Actions après l'exécution du job.

### 4. Affichage du résumé de couverture de code dans GitHub Actions
- Problème : Initialement, le rapport de couverture de code n'était affiché qu'en tant que fichier brut (via la commande cat). Cependant, cela ne fournissait pas une vue claire dans l'interface GitHub.
- Cause : La commande cat affichait simplement le contenu XML dans les logs, mais n'était pas bien formatée ou présentée dans l'interface utilisateur GitHub.
- Solution : Nous avons utilisé la fonctionnalité de GitHub Actions Job Summaries pour générer un résumé enrichi en Markdown du rapport de couverture de code. Cela a permis de créer une table récapitulative plus lisible avec des informations claires sur la couverture du code directement dans l'onglet Summary de GitHub Actions.

### 5. Structure et chemin d'accès du projet
- Problème : Les tests PHPUnit devaient être exécutés dans un sous-répertoire (PrivateBin-main) et cela n'était pas explicitement précisé au départ, provoquant l'échec de certaines étapes.
- Cause : Les commandes PHPUnit cherchaient à s'exécuter à la racine du projet sans naviguer vers le répertoire approprié.
- Solution : Nous avons ajouté une commande explicite dans le fichier YAML pour naviguer vers le bon répertoire avant d'exécuter les tests PHPUnit : ```cd PrivateBin-main```

### 6. Amélioration de l'affichage du rapport avec un tableau Markdown
- Problème : Bien que le résumé de couverture de code ait été affiché, il manquait une structuration claire, comme un tableau, pour rendre l'information plus compréhensible.
- Solution : Nous avons utilisé la fonctionnalité de Markdown avancé et de tableaux dans GitHub Actions. En ajoutant un tableau dans le résumé du job, nous avons pu afficher les résultats de manière plus lisible et structurée directement dans la section Summary de l'action.

### Conclusion :
Ces problèmes étaient tous liés à des erreurs de configuration dans GitHub Actions, de génération de couverture de code, et à des erreurs dans la gestion des fichiers et formats. En apportant les corrections nécessaires, notamment en utilisant des actions appropriées, en choisissant les bons formats de rapport, et en configurant correctement l'affichage des résultats dans GitHub Actions, nous avons réussi à automatiser les tests avec un rapport de couverture bien formaté et accessible.
