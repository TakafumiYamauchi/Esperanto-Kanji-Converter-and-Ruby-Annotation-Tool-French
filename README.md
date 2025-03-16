# Guide d'utilisation de l'outil de remplacement de texte en espéranto par des caractères chinois (kanji) et d'annotation HTML

## Introduction

Cette application web, développée avec Streamlit, permet de transformer du texte en espéranto de plusieurs façons :

1. **Remplacer les racines espéranto** par des caractères chinois (kanji) ou des traductions en français
2. **Ajouter des annotations Ruby HTML** sur les mots espéranto pour faciliter l'apprentissage
3. **Choisir différents formats de sortie** (HTML avec Ruby, format avec parenthèses, etc.)

Elle offre également la possibilité de générer des fichiers JSON personnalisés pour définir vos propres règles de substitution.

## Table des matières

1. [Page principale : Remplacement de texte](#page-principale--remplacement-de-texte)
   - [Charger un fichier JSON de remplacement](#charger-un-fichier-json-de-remplacement)
   - [Paramètres avancés](#paramètres-avancés)
   - [Sélectionner le format de sortie](#sélectionner-le-format-de-sortie)
   - [Source du texte d'entrée](#source-du-texte-dentrée)
   - [Saisie du texte et options d'affichage](#saisie-du-texte-et-options-daffichage)
   - [Utilisation des marqueurs spéciaux (% et @)](#utilisation-des-marqueurs-spéciaux--et-)
   - [Résultats et téléchargement](#résultats-et-téléchargement)

2. [Page secondaire : Génération de fichier JSON](#page-secondaire--génération-de-fichier-json)
   - [Préparer le fichier CSV](#préparer-le-fichier-csv)
   - [Préparer les fichiers JSON](#préparer-les-fichiers-json)
   - [Paramètres avancés](#paramètres-avancés-1)
   - [Créer le fichier JSON final](#créer-le-fichier-json-final)

3. [Exemples d'utilisation](#exemples-dutilisation)
   - [Exemple simple](#exemple-simple)
   - [Exemple avec annotations Ruby](#exemple-avec-annotations-ruby)
   - [Utilisation des marqueurs spéciaux](#utilisation-des-marqueurs-spéciaux)

4. [Ressources disponibles](#ressources-disponibles)
   - [Fichiers d'exemple](#fichiers-dexemple)
   - [Versions dans d'autres langues](#versions-dans-dautres-langues)

---

## Page principale : Remplacement de texte

La page principale permet de transformer du texte en espéranto selon vos besoins.

### Charger un fichier JSON de remplacement

Le premier choix à effectuer concerne le fichier JSON qui contient les règles de remplacement :

- **Utiliser le fichier JSON par défaut** : Option recommandée pour les débutants. Ce fichier contient déjà des milliers de correspondances entre racines espéranto et leurs traductions.
- **Téléverser un fichier** : Si vous avez créé votre propre fichier JSON avec des règles personnalisées, vous pouvez l'utiliser ici.

Vous pouvez également télécharger un fichier JSON d'exemple en cliquant sur le bouton dans la section dépliable "Télécharger un fichier JSON d'exemple".

### Paramètres avancés

Pour les utilisateurs avancés, vous pouvez activer le traitement parallèle qui utilise plusieurs cœurs de processeur pour accélérer le traitement des textes volumineux :

- Cochez **Utiliser le traitement parallèle**
- Définissez le **Nombre de processus simultanés** (généralement entre 2 et 4)

### Sélectionner le format de sortie

Choisissez le format dans lequel vous souhaitez obtenir le texte transformé :

- **Format HTML avec annotations (ruby) et ajustement de taille** : Ajoute des annotations au-dessus des mots espéranto, avec ajustement automatique de la taille des annotations selon leur longueur
- **Format HTML avec annotations (ruby), ajustement de taille et remplacement de kanji** : Comme ci-dessus, mais remplace aussi les mots espéranto par des caractères chinois (kanji)
- **Format HTML** : Format HTML simple avec annotations
- **Format HTML avec remplacement de kanji** : Format HTML avec remplacement des mots espéranto par des caractères chinois
- **Format avec parenthèses** : Ajoute les traductions entre parenthèses après chaque mot
- **Format avec parenthèses et remplacement de kanji** : Comme ci-dessus, mais place les mots espéranto entre parenthèses
- **Conserver uniquement le texte remplacé** : Remplace simplement les mots espéranto sans ajouter d'annotations

### Source du texte d'entrée

Deux options pour fournir le texte en espéranto à transformer :

- **Saisie manuelle** : Tapez ou collez directement le texte dans le champ
- **Téléverser un fichier** : Importez un fichier texte (TXT, CSV ou MD) encodé en UTF-8

### Saisie du texte et options d'affichage

Une fois la source sélectionnée :

1. Saisissez ou vérifiez le texte dans le champ prévu
2. Choisissez la forme d'affichage des caractères spéciaux de l'espéranto dans le résultat :
   - **Accent sur la lettre (ĉ → c + ˆ)** : Affiche les caractères espéranto avec des accents au-dessus
   - **Format avec x (ĉ → cx)** : Remplace les caractères spéciaux par la notation "x" (ex: "cx" pour "ĉ")
   - **Format avec ^ (ĉ → c^)** : Remplace les caractères spéciaux par la notation "^" (ex: "c^" pour "ĉ")
3. Cliquez sur **Envoyer** pour lancer le traitement

### Utilisation des marqueurs spéciaux (% et @)

L'application offre deux marqueurs spéciaux pour contrôler précisément les remplacements :

- **%texte%** : Le texte entre les signes % ne sera pas remplacé et sera conservé tel quel
  - Exemple : `La %Universala Esperanto-Asocio% estas grava organizo` → Seul "Universala Esperanto-Asocio" restera inchangé
  - Limite : 50 caractères maximum entre les %

- **@texte@** : Le texte entre les signes @ sera remplacé de manière localisée (différemment du reste du texte)
  - Exemple : `Mi @amas@ vin` → Le mot "amas" sera traité spécifiquement
  - Limite : 18 caractères maximum entre les @

### Résultats et téléchargement

Après le traitement, les résultats sont affichés selon le format choisi :

- Pour les formats HTML : Deux onglets sont disponibles
  - **Aperçu HTML** : Visualisation du résultat avec les annotations
  - **Résultat (code HTML)** : Code source HTML généré

- Pour les autres formats : Un seul onglet **Texte résultant**

Un bouton **Télécharger le résultat** vous permet de sauvegarder le résultat au format HTML ou texte sur votre ordinateur.

---

## Page secondaire : Génération de fichier JSON

La seconde page (accessible via le menu latéral) vous permet de créer vos propres fichiers JSON de substitution. Cette fonctionnalité est utile si vous souhaitez personnaliser les remplacements ou ajouter de nouvelles correspondances entre racines espéranto et traductions.

### Préparer le fichier CSV

Première étape : préparer un fichier CSV contenant les correspondances entre les racines espéranto et leurs traductions :

1. Choisissez entre :
   - **Importer un fichier CSV** : Téléversez votre propre fichier CSV
   - **Utiliser le fichier par défaut** : Utilise le fichier intégré à l'application

Le format du CSV doit comporter au minimum deux colonnes :
- Première colonne : Racine en espéranto
- Deuxième colonne : Traduction en français ou caractère chinois

Vous pouvez télécharger différents exemples dans la section "Liste de fichiers d'exemple".

### Préparer les fichiers JSON

Ensuite, vous devez choisir deux fichiers JSON qui définissent les règles de transformation :

1. **Fichier JSON définissant la décomposition des racines en espéranto** :
   - Définit comment les mots espéranto sont divisés en racines
   - Détermine quand ajouter un suffixe ou une terminaison verbale
   - Exemple de format : `["am", "dflt", ["verbo_s1"]]` (pour traiter "am" comme un verbe avec ses terminaisons)

2. **Fichier JSON définissant la chaîne de substitution** :
   - Définit des caractères chinois ou un format personnalisé pour certains mots
   - Généralement facultatif car l'édition du CSV est souvent suffisante

Pour chacun, vous pouvez importer votre propre fichier ou utiliser le fichier par défaut.

### Paramètres avancés

Comme dans la page principale, vous pouvez activer le traitement parallèle pour accélérer la génération du fichier JSON :

- Cochez **Utiliser le traitement parallèle**
- Définissez le **Nombre de processus simultanés** (généralement entre 2 et 6)

### Créer le fichier JSON final

Après avoir configuré tous les paramètres :

1. Cliquez sur **Créer le fichier JSON pour la substitution**
2. Un traitement (qui peut prendre plusieurs secondes) va générer un fichier JSON combiné
3. Une fois la génération terminée, cliquez sur **Télécharger la liste finale de substitution**

Ce fichier JSON généré peut ensuite être utilisé dans la page principale comme fichier téléversé pour effectuer vos remplacements personnalisés.

---

## Exemples d'utilisation

### Exemple simple

**Texte en espéranto :**
```
Mi lernas Esperanton.
```

**Avec le format "Format HTML avec annotations (ruby)" :**
Résultat : Les mots seront annotés avec leur traduction au-dessus.

### Exemple avec annotations Ruby

**Texte en espéranto :**
```
La suno brilas en la blua ĉielo.
```

**Avec le format "Format HTML avec annotations (ruby) et ajustement de taille" :**
Résultat : Chaque mot aura une annotation au-dessus, avec une taille d'annotation ajustée selon la longueur du mot et de sa traduction.

### Utilisation des marqueurs spéciaux

**Texte en espéranto avec marqueurs :**
```
Mi %ne volas% @manĝi@ pomon.
```

**Résultat :**
- "ne volas" restera inchangé (grâce aux %)
- "manĝi" sera remplacé selon des règles spécifiques à ce fragment (grâce aux @)
- "Mi" et "pomon" seront remplacés normalement selon les règles générales

---

## Ressources disponibles

### Fichiers d'exemple

L'application propose plusieurs fichiers d'exemple que vous pouvez télécharger :

- **CSV d'exemple** : Correspondances entre racines espéranto et traductions
- **JSON d'exemple** : Règles de décomposition et de substitution
- **Excel d'exemple** : Racines espéranto avec traductions dans plusieurs langues

Ces fichiers peuvent servir de modèles pour créer vos propres règles de substitution.

### Versions dans d'autres langues

L'application est disponible dans 14 langues différentes, accessibles via les liens en bas de la page principale :

- Esperanto, English, 日本語, 中文, 한국어, Русский, español, italiano, français, Deutsch, العربية, हिन्दी, polski, Tiếng Việt, Bahasa Indonesia

Vous pouvez également consulter les instructions d'utilisation détaillées (README.md) dans la langue de votre choix sur GitHub.

---

En suivant ce guide, vous pourrez exploiter pleinement les fonctionnalités de cet outil de remplacement de texte en espéranto, que ce soit pour l'apprentissage, la traduction ou la visualisation de textes avec des caractères chinois.
