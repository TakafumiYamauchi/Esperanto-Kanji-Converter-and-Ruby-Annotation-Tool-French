# Guide d'utilisation de l'outil de remplacement et d'annotation pour le texte en espéranto

## Table des matières

- [Guide d'utilisation de l'outil de remplacement et d'annotation pour le texte en espéranto](#guide-dutilisation-de-loutil-de-remplacement-et-dannotation-pour-le-texte-en-espéranto)
  - [Table des matières](#table-des-matières)
- [Guide d'utilisation de l'outil de remplacement et d'annotation pour le texte en espéranto](#guide-dutilisation-de-loutil-de-remplacement-et-dannotation-pour-le-texte-en-espéranto-1)
  - [1. Introduction](#1-introduction)
  - [2. Page principale](#2-page-principale)
    - [2.1 Chargement du fichier JSON](#21-chargement-du-fichier-json)
    - [2.2 Format de sortie](#22-format-de-sortie)
    - [2.3 Source du texte d'entrée](#23-source-du-texte-dentrée)
    - [2.4 Traitement du texte](#24-traitement-du-texte)
    - [2.5 Affichage et téléchargement des résultats](#25-affichage-et-téléchargement-des-résultats)
  - [3. Page de génération de fichiers JSON](#3-page-de-génération-de-fichiers-json)
    - [3.1 Comprendre les fichiers JSON de remplacement](#31-comprendre-les-fichiers-json-de-remplacement)
    - [3.2 Préparation du fichier CSV](#32-préparation-du-fichier-csv)
    - [3.3 Règles de décomposition et chaînes de substitution](#33-règles-de-décomposition-et-chaînes-de-substitution)
    - [3.4 Création du fichier JSON final](#34-création-du-fichier-json-final)
  - [4. Fonctionnalités avancées](#4-fonctionnalités-avancées)
    - [4.1 Traitement parallèle](#41-traitement-parallèle)
    - [4.2 Marqueurs spéciaux](#42-marqueurs-spéciaux)
  - [5. Astuces et résolution de problèmes](#5-astuces-et-résolution-de-problèmes)
  - [1. Introduction](#1-introduction-1)
  - [2. Page principale](#2-page-principale-1)
    - [Chargement du fichier JSON](#chargement-du-fichier-json)
    - [Format de sortie](#format-de-sortie)
    - [Source du texte d'entrée](#source-du-texte-dentrée)
    - [Traitement du texte](#traitement-du-texte)
    - [Affichage et téléchargement des résultats](#affichage-et-téléchargement-des-résultats)
  - [3. Page de génération de fichiers JSON](#3-page-de-génération-de-fichiers-json-1)
    - [Comprendre les fichiers JSON de remplacement](#comprendre-les-fichiers-json-de-remplacement)
    - [Préparation du fichier CSV](#préparation-du-fichier-csv)
    - [Règles de décomposition et chaînes de substitution](#règles-de-décomposition-et-chaînes-de-substitution)
    - [Création du fichier JSON final](#création-du-fichier-json-final)
  - [4. Fonctionnalités avancées](#4-fonctionnalités-avancées-1)
    - [Traitement parallèle](#traitement-parallèle)
    - [Marqueurs spéciaux](#marqueurs-spéciaux)
  - [5. Astuces et résolution de problèmes](#5-astuces-et-résolution-de-problèmes-1)


# Guide d'utilisation de l'outil de remplacement et d'annotation pour le texte en espéranto

## 1. Introduction

Cette application web vous permet de transformer du texte en espéranto de différentes manières :

- Remplacer des racines de mots espéranto par des caractères chinois (ou des traductions françaises)
- Ajouter des annotations "ruby" (petit texte explicatif au-dessus des mots)
- Formater le texte de différentes façons (HTML avec annotations, format parenthèses, etc.)

L'application se compose de deux pages principales :
1. **Page principale** : pour traiter directement votre texte en espéranto
2. **Page de génération de fichiers JSON** : pour créer vos propres règles de remplacement personnalisées

## 2. Page principale

La page principale vous permet de traiter immédiatement du texte en espéranto avec les règles de remplacement prédéfinies ou personnalisées.

### 2.1 Chargement du fichier JSON

Le fichier JSON contient toutes les règles de remplacement qui seront appliquées à votre texte. Vous avez deux options :

- **Utiliser le fichier JSON par défaut** : option la plus simple pour commencer
- **Téléverser un fichier** : si vous avez créé votre propre fichier JSON personnalisé

Pour obtenir un exemple de fichier JSON, cliquez sur "Télécharger un fichier JSON d'exemple" dans la section dépliable.

### 2.2 Format de sortie

Sélectionnez le format de sortie souhaité parmi les options suivantes :

- **Format HTML avec annotations (ruby) et ajustement de taille** : affiche le texte original avec des annotations ajustées proportionnellement à la taille du texte
- **Format HTML avec annotations (ruby), ajustement de taille et remplacement de kanji** : remplace les mots espéranto par des caractères chinois avec le texte espéranto en annotation
- **Format HTML** : format HTML simple avec annotations
- **Format HTML avec remplacement de kanji** : format HTML simple avec remplacement par caractères chinois
- **Format avec parenthèses** : affiche le texte original suivi de la traduction entre parenthèses
- **Format avec parenthèses et remplacement de kanji** : affiche les caractères chinois suivis du texte espéranto entre parenthèses
- **Conserver uniquement le texte remplacé** : montre uniquement le résultat du remplacement, sans le texte original

### 2.3 Source du texte d'entrée

Vous pouvez fournir le texte à traiter de deux manières :

- **Saisie manuelle** : tapez ou collez votre texte directement dans la zone de texte
- **Téléverser un fichier** : importez un fichier texte (TXT, CSV ou MD) en encodage UTF-8

### 2.4 Traitement du texte

1. Saisissez ou importez votre texte en espéranto
2. Choisissez la forme d'affichage des caractères spéciaux de l'espéranto :
   - **Accent sur la lettre (ĉ → c + ˆ)** : utilise des accents circonflexes
   - **Format avec x (ĉ → cx)** : utilise la notation avec x
   - **Format avec ^ (ĉ → c^)** : utilise le caractère ^

3. Cliquez sur le bouton "Envoyer" pour traiter le texte

> **Astuce** : Vous pouvez utiliser le traitement parallèle dans les paramètres avancés pour accélérer le traitement des textes volumineux.

### 2.5 Affichage et téléchargement des résultats

Après le traitement, le résultat s'affiche selon le format choisi :

- Pour les formats HTML : deux onglets sont disponibles - "Aperçu HTML" (rendu visuel) et "Résultat (code HTML)" (code source)
- Pour les autres formats : le résultat s'affiche dans l'onglet "Texte résultant"

Vous pouvez télécharger le résultat en cliquant sur le bouton "Télécharger le résultat".

## 3. Page de génération de fichiers JSON

Cette page vous permet de créer vos propres fichiers JSON de remplacement personnalisés.

### 3.1 Comprendre les fichiers JSON de remplacement

Le fichier JSON final contient trois listes principales :

1. **Liste pour le remplacement global** : règles appliquées sur l'ensemble du texte
2. **Liste pour le remplacement localisé** : règles appliquées uniquement aux sections marquées avec @...@
3. **Liste pour le remplacement de racines à deux caractères** : règles spécifiques pour les racines courtes

### 3.2 Préparation du fichier CSV

Le fichier CSV est la base de votre système de remplacement. Il contient deux colonnes :

1. **Racine en espéranto** : le mot ou la racine à remplacer
2. **Traduction/annotation** : le texte qui remplacera ou annotera la racine

Vous avez deux options :
- **Utiliser le fichier CSV par défaut** : contient des traductions françaises avec annotations ruby
- **Importer un fichier CSV** : utiliser votre propre fichier de correspondances

Des exemples de fichiers CSV sont disponibles dans la section "Liste de fichiers d'exemple".

### 3.3 Règles de décomposition et chaînes de substitution

Vous devez également spécifier deux fichiers JSON complémentaires :

1. **Fichier de décomposition des racines** : définit comment décomposer les mots espéranto en racines pour appliquer les remplacements
   - Par exemple : définir qu'un mot se terminant par "as" est un verbe au présent

2. **Fichier de chaînes de substitution** : définit des remplacements spécifiques pour certains mots complets
   - Généralement utilisé pour des cas particuliers non couverts par les règles générales

Pour chaque fichier, vous pouvez :
- Utiliser le fichier par défaut
- Importer votre propre fichier JSON

### 3.4 Création du fichier JSON final

1. Sélectionnez le format de sortie souhaité (le même que celui que vous utiliserez dans la page principale)
2. Si nécessaire, configurez le traitement parallèle dans les paramètres avancés
3. Cliquez sur "Créer le fichier JSON pour la substitution"
4. Téléchargez le fichier JSON généré en cliquant sur le bouton qui apparaît

Ce fichier JSON pourra ensuite être utilisé dans la page principale en sélectionnant "Téléverser un fichier" dans la section de chargement du fichier JSON.

## 4. Fonctionnalités avancées

### 4.1 Traitement parallèle

Pour accélérer le traitement des textes volumineux ou la génération de fichiers JSON complexes :

1. Ouvrez la section "Paramètres avancés"
2. Cochez "Utiliser le traitement parallèle"
3. Définissez le nombre de processus simultanés (généralement entre 2 et 4)

Le traitement parallèle est particulièrement utile pour :
- Les textes de plus de 1000 lignes
- Les fichiers JSON contenant de nombreuses règles de remplacement

### 4.2 Marqueurs spéciaux

Vous pouvez utiliser des marqueurs spéciaux dans votre texte pour contrôler précisément les remplacements :

- **%texte%** : Le texte entre % ne sera pas remplacé et sera conservé tel quel
  - Exemple : `La vorto %knabo% signifas "garçon"` → Le mot "knabo" restera inchangé

- **@texte@** : Le texte entre @ sera remplacé de manière localisée (différemment du reste du texte)
  - Exemple : `Mi @vidas@ la domon` → Seul le mot "vidas" recevra un traitement particulier

Ces marqueurs sont particulièrement utiles pour préserver certains termes ou pour appliquer des règles spécifiques à certaines parties du texte.

## 5. Astuces et résolution de problèmes

- **Encodage des fichiers** : Assurez-vous que tous vos fichiers sont encodés en UTF-8 pour éviter les problèmes de caractères spéciaux.

- **Textes volumineux** : Pour les textes très longs, l'application affiche un aperçu partiel (247 premières lignes et 3 dernières). Le téléchargement contiendra toujours le texte complet.

- **Conflits de remplacement** : Si certains remplacements semblent incorrects, vérifiez les priorités dans vos fichiers JSON ou utilisez les marqueurs % pour protéger certains mots.

- **Format de sortie incohérent** : Assurez-vous que le format de sortie sélectionné dans la page principale correspond à celui utilisé lors de la génération du fichier JSON.

- **Fichiers d'exemple** : Pour mieux comprendre la structure des fichiers, téléchargez et examinez les fichiers d'exemple fournis dans la section "Liste des fichiers d'exemple".

- **Versions dans d'autres langues** : Des liens vers les versions de l'application dans 14 autres langues sont disponibles en bas de la page principale.


## 1. Introduction

Cette application web est un outil spécialisé qui vous permet de transformer des textes écrits en espéranto de diverses manières. Vous pouvez notamment :

- Remplacer les racines des mots en espéranto par des caractères chinois (kanji) ou des traductions françaises
- Ajouter des annotations "ruby" (petits textes explicatifs placés au-dessus des mots)
- Formater votre texte selon différentes présentations (HTML avec annotations, format parenthèses, etc.)

Cette application est particulièrement utile pour les apprenants de l'espéranto qui souhaitent visualiser les correspondances entre les mots espéranto et leurs traductions, ou pour créer des documents pédagogiques bilingues.

## 2. Page principale

La page principale de l'application vous permet de traiter immédiatement un texte en espéranto selon vos préférences.

### Chargement du fichier JSON

Le fichier JSON est essentiel car il contient toutes les règles de remplacement qui seront appliquées à votre texte. Vous avez deux options :

- **Utiliser le fichier JSON par défaut** : solution recommandée pour débuter
- **Téléverser votre propre fichier** : si vous avez créé ou modifié un fichier JSON personnalisé

Pour voir à quoi ressemble un fichier JSON type, cliquez sur "Télécharger un fichier JSON d'exemple" dans la section dépliable correspondante.

### Format de sortie

L'application propose plusieurs formats de sortie, chacun ayant son utilité spécifique :

- **Format HTML avec annotations (ruby) et ajustement de taille** : présente le texte original avec des annotations dont la taille s'ajuste proportionnellement au texte principal
- **Format HTML avec annotations, ajustement de taille et remplacement de kanji** : remplace les mots espéranto par des caractères chinois avec le texte espéranto en annotation
- **Format HTML simple** : présentation HTML basique avec annotations
- **Format HTML avec remplacement de kanji** : HTML simple avec substitution par caractères chinois
- **Format avec parenthèses** : affiche le texte original suivi de la traduction entre parenthèses
- **Format avec parenthèses et remplacement de kanji** : montre les caractères chinois suivis du texte espéranto entre parenthèses
- **Conserver uniquement le texte remplacé** : présente seulement le résultat du remplacement

### Source du texte d'entrée

Vous pouvez fournir votre texte de deux façons :

- **Saisie manuelle** : directement dans la zone de texte
- **Téléversement de fichier** : en important un fichier texte (formats TXT, CSV ou MD) en encodage UTF-8

### Traitement du texte

1. Une fois votre texte saisi ou importé, vous devez choisir comment afficher les caractères spéciaux de l'espéranto :
   - **Accent sur la lettre (ĉ → c + ˆ)** : utilise des accents circonflexes (forme standard)
   - **Format avec x (ĉ → cx)** : utilise la notation avec x (courante pour la saisie)
   - **Format avec ^ (ĉ → c^)** : utilise le caractère ^ (alternative)

2. Cliquez sur "Envoyer" pour lancer le traitement

3. Si vous souhaitez annuler l'opération, cliquez simplement sur "Annuler"

### Affichage et téléchargement des résultats

Une fois le traitement terminé, vous verrez le résultat s'afficher selon le format choisi :

- Pour les formats HTML : deux onglets sont disponibles
  - "Aperçu HTML" : vous montre le rendu visuel comme il apparaîtrait dans un navigateur
  - "Résultat (code HTML)" : affiche le code source HTML généré

- Pour les autres formats : le résultat s'affiche dans l'onglet "Texte résultant"

Pour conserver votre travail, cliquez sur "Télécharger le résultat" pour obtenir un fichier contenant le texte transformé.

## 3. Page de génération de fichiers JSON

Cette page spéciale vous permet de créer vos propres fichiers JSON de remplacement, offrant ainsi une personnalisation complète du système.

### Comprendre les fichiers JSON de remplacement

Le fichier JSON que vous allez créer contient trois listes principales :

1. **Liste pour le remplacement global** : règles appliquées sur l'ensemble du texte
2. **Liste pour le remplacement localisé** : règles appliquées uniquement aux sections marquées avec @...@
3. **Liste pour le remplacement de racines à deux caractères** : règles spécifiques pour les préfixes, suffixes et petites racines

### Préparation du fichier CSV

Le fichier CSV est la base de votre système de remplacement. Il doit contenir deux colonnes :

1. **Première colonne** : la racine en espéranto à remplacer
2. **Seconde colonne** : la traduction ou l'annotation correspondante

Vous pouvez :
- **Utiliser le fichier CSV par défaut** : déjà configuré avec des traductions françaises
- **Importer votre propre fichier CSV** : si vous souhaitez des traductions personnalisées

Plusieurs exemples de fichiers CSV sont disponibles dans la section "Liste de fichiers d'exemple", notamment :
- CSV avec traductions françaises et annotations ruby
- CSV avec correspondances espéranto-caractères chinois (version Mingeo)
- Fichier Excel avec traductions dans 14 langues différentes

### Règles de décomposition et chaînes de substitution

Deux fichiers JSON complémentaires sont nécessaires :

1. **Fichier de décomposition des racines** : définit comment analyser les mots espéranto
   - Par exemple, il précise comment traiter les terminaisons verbales (-as, -is, -os)
   - Ce fichier est crucial pour la reconnaissance correcte des racines

2. **Fichier de chaînes de substitution** : définit des remplacements spécifiques
   - Utilisé principalement pour les exceptions ou cas particuliers
   - Moins couramment modifié que le fichier de décomposition

Pour chaque fichier, vous pouvez utiliser la version par défaut ou importer la vôtre.

### Création du fichier JSON final

1. Sélectionnez le format de sortie désiré (identique à celui que vous utiliserez ensuite)
2. Configurez éventuellement le traitement parallèle dans les paramètres avancés
3. Cliquez sur "Créer le fichier JSON pour la substitution"
4. Une fois le traitement terminé, téléchargez le fichier JSON généré

Ce fichier pourra ensuite être utilisé dans la page principale en choisissant l'option "Téléverser un fichier" lors du chargement du fichier JSON.

## 4. Fonctionnalités avancées

### Traitement parallèle

Pour les textes volumineux ou les opérations complexes, l'application propose une option de traitement parallèle :

1. Ouvrez la section "Paramètres avancés"
2. Cochez "Utiliser le traitement parallèle"
3. Définissez le nombre de processus simultanés (généralement entre 2 et 4)

Cette fonctionnalité est particulièrement utile pour :
- Les textes de plus de 1000 lignes
- Les fichiers JSON contenant de nombreuses règles
- Les ordinateurs multi-cœurs

### Marqueurs spéciaux

L'application reconnaît deux types de marqueurs spéciaux que vous pouvez insérer dans votre texte :

- **%texte%** : Le texte entre signes % ne sera pas modifié
  - Exemple : `La vorto %knabo% signifas "garçon"` → Le mot "knabo" restera intact
  - Limité à 50 caractères maximum entre les %

- **@texte@** : Le texte entre signes @ sera traité selon les règles de remplacement localisé
  - Exemple : `Mi @vidas@ la domon` → Seul "vidas" recevra un traitement spécifique
  - Limité à 18 caractères maximum entre les @

Ces marqueurs vous permettent un contrôle précis sur les parties de texte à traiter ou à préserver.

## 5. Astuces et résolution de problèmes

- **Pour les textes très longs** : L'application affiche un aperçu partiel (247 premières lignes et 3 dernières), mais le fichier téléchargé contiendra l'intégralité du texte traité.

- **Encodage des fichiers** : Tous vos fichiers doivent être encodés en UTF-8 pour éviter les problèmes avec les caractères spéciaux de l'espéranto.

- **Cohérence des formats** : Assurez-vous que le format de sortie sélectionné dans la page principale correspond à celui utilisé lors de la génération du fichier JSON.

- **Problèmes de remplacement** : Si certains mots ne sont pas correctement remplacés, vérifiez leur présence dans votre fichier CSV et utilisez les marqueurs % pour protéger les termes sensibles.

- **Versions linguistiques** : L'application est disponible en 14 langues différentes. Les liens vers ces versions se trouvent en bas de la page principale.

- **Navigation entre pages** : Pour passer de la page principale à la page de génération de fichiers JSON, utilisez le menu latéral de Streamlit.

Cette application polyvalente vous offre un contrôle précis sur la transformation de textes en espéranto, que ce soit pour l'apprentissage, la pédagogie ou simplement pour explorer les correspondances entre l'espéranto et d'autres langues.