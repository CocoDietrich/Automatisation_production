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

### En résumé
Après l'exécution du workflow, le rapport de coverage sera visible sous forme de texte formaté directement dans le résumé du job sur la page GitHub Actions.
Ainsi, le fichier ci.yml permet non seulement de générer le rapport et de le stocker en fichier, mais aussi de le lire facilement directement dans l'interface GitHub Actions.
