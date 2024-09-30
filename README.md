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

### Étape 2 : Installation de PHP et des extensions
```
      - name: Set up PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-versions }}
          extensions: mbstring, dom, xml, json, libxml

```

### Étape 3 : Installation des dépendances
```
      - name: Install dependencies
        run: composer install --working-dir=PrivateBin-main

```

### Étape 4 : Exécution de PHPUnit avec génération de la couverture de code
```
      - name: Run PHPUnit with Code Coverage
        run: |
          cd PrivateBin-main
          ./vendor/bin/phpunit --coverage-cobertura=coverage_cobertura.xml

```

### Étape 5 : Téléchargement du rapport de couverture
```
      - name: Upload coverage report
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: PrivateBin-main/coverage_cobertura.xml

```

### Étape 6 : Vérification du fichier de couverture
```
      - name: Check coverage report file
        run: cat PrivateBin-main/coverage_cobertura.xml

```

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

