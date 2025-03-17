# Manuel d'utilisation : Outil de remplacement et d'annotation pour textes en espéranto

## Introduction

Bienvenue dans ce guide d'utilisation de l'outil de remplacement et d'annotation pour textes en espéranto. Cette application Streamlit vous permet de transformer des textes en espéranto de deux manières principales :

1. **Remplacer des mots/racines en espéranto** par des caractères kanji ou des traductions en français
2. **Ajouter des annotations** (furigana/ruby) au-dessus des mots en espéranto

L'application est composée de deux pages principales :
- **Page d'accueil** : pour effectuer les remplacements et annotations sur votre texte
- **Page secondaire** : pour créer vos propres fichiers JSON de règles de remplacement

## Page principale : Remplacement et annotation de texte

### 1. Démarrage et chargement du fichier JSON

Dès l'ouverture de l'application, vous devez d'abord choisir comment charger le fichier JSON contenant les règles de remplacement :

- **Utiliser le fichier JSON par défaut** : option recommandée pour débuter
- **Téléverser un fichier** : si vous disposez déjà d'un fichier JSON personnalisé

Si vous choisissez de télécharger le fichier par défaut pour référence ou modification, vous pouvez cliquer sur "Télécharger le fichier JSON d'exemple" dans la section dépliable.

### 2. Configuration des paramètres avancés

Dans la section "Paramètres avancés", vous pouvez activer le **traitement parallèle** pour accélérer la conversion des textes volumineux :

- Cochez la case "Utiliser le traitement parallèle"
- Définissez le nombre de processus simultanés (2 à 4 recommandés)

Cette option est particulièrement utile pour les textes de grande taille.

### 3. Sélection du format de sortie

Choisissez le format dans lequel vous souhaitez obtenir votre texte transformé :

- **Format HTML avec annotations (ruby) et ajustement de taille** : ajoute des annotations au-dessus des mots en espéranto, avec ajustement automatique de la taille du texte d'annotation
- **Format HTML avec annotations, ajustement de taille et remplacement de kanji** : remplace le texte espéranto par des kanji/français et place le texte original en annotation
- **Format HTML** : version basique des annotations HTML
- **Format HTML avec remplacement de kanji** : version basique du remplacement
- **Format avec parenthèses** : remplace les annotations HTML par des parenthèses
- **Format avec parenthèses et remplacement de kanji** : version avec parenthèses et remplacement
- **Conserver uniquement le texte remplacé** : effectue un simple remplacement sans annotations

### 4. Saisie du texte à transformer

Vous avez deux options pour fournir le texte à transformer :

- **Saisie manuelle** : entrez directement le texte en espéranto dans la zone de texte
- **Téléverser un fichier** : importez un fichier texte (UTF-8) contenant le texte à transformer

### 5. Options spéciales de formatage du texte

Deux mécanismes spéciaux vous permettent de contrôler finement les remplacements :

- **Texte entre %...%** : les portions de texte entourées par le signe % ne seront **pas remplacées**  
  Exemple : `La %universala% lingvo estas bona` → seul "universala" sera conservé tel quel

- **Texte entre @...@** : les portions entourées par @ seront remplacées de manière **localisée**  
  Exemple : `Mi @amas@ vin` → seul "amas" sera remplacé selon des règles spécifiques

### 6. Forme d'affichage des caractères spéciaux de l'espéranto

Choisissez comment afficher les caractères spéciaux de l'espéranto dans le résultat :

- **Accent sur la lettre** (ĉ → c + ˆ) : utilise les accents circonflexes au-dessus des lettres
- **Format avec x** (ĉ → cx) : utilise la notation avec 'x' après la lettre
- **Format avec ^** (ĉ → c^) : utilise le symbole circonflexe après la lettre

### 7. Traitement et affichage des résultats

Après avoir cliqué sur "Envoyer", l'application transforme votre texte selon les paramètres choisis :

- Si vous avez sélectionné un format HTML, vous verrez deux onglets :
  - **Aperçu HTML** : visualisation du résultat avec les annotations rendues
  - **Résultat (code HTML)** : code source HTML que vous pouvez copier

- Pour les autres formats, le résultat s'affiche dans un seul onglet texte

Vous pouvez ensuite télécharger le résultat en cliquant sur le bouton "Télécharger le résultat".

## Page secondaire : Création de fichiers JSON de remplacement

Cette page vous permet de créer votre propre fichier JSON pour personnaliser les règles de remplacement.

### 1. Présentation et téléchargement des fichiers d'exemple

Dans la section dépliable "Liste de fichiers d'exemple", vous pouvez télécharger :

- **Fichiers CSV** contenant les correspondances entre racines d'espéranto et traductions
- **Fichiers JSON** définissant les règles de décomposition des racines
- **Fichiers Excel** contenant des listes de racines avec traductions dans différentes langues

Ces fichiers vous serviront de modèles et de références pour créer vos propres règles.

### 2. Sélection du format de sortie

Choisissez le format que vous souhaitez utiliser pour les remplacements, comme dans la page principale.

### 3. Préparation du fichier CSV

Le fichier CSV est essentiel car il définit les correspondances entre racines d'espéranto et leurs traductions :

- Vous pouvez importer votre propre fichier CSV ou utiliser celui par défaut
- Le format attendu est : première colonne = racine espéranto, deuxième colonne = traduction/kanji

### 4. Préparation des fichiers JSON

Deux fichiers JSON sont nécessaires :

- **Premier fichier JSON** : définit la décomposition des racines en espéranto
  - Format : `["racine", "priorité", ["modificateurs"]]`
  - Exemple : `["am", "dflt", ["verbo_s1"]]` signifie que "am" est une racine verbale qui peut recevoir des terminaisons verbales

- **Second fichier JSON** : définit les chaînes de substitution personnalisées (généralement facultatif)
  - Permet d'attribuer des caractères ou formats spécifiques à certains mots

### 5. Paramètres avancés

Comme dans la page principale, vous pouvez configurer le traitement parallèle pour accélérer la génération du fichier JSON.

### 6. Création du fichier JSON final

Cliquez sur "Créer le fichier JSON pour la substitution" pour générer le fichier combiné qui contient :

- La liste des remplacements globaux
- La liste des remplacements pour les racines de deux caractères
- La liste des remplacements localisés

Une fois le traitement terminé, vous pourrez télécharger le fichier JSON généré.

## Fonctionnalités avancées et astuces

### Comprendre les différents types de remplacements

L'application utilise trois types de listes de remplacement :

1. **Remplacements globaux** (`replacements_final_list`) : appliqués à l'ensemble du texte
2. **Remplacements pour racines de deux caractères** (`replacements_list_for_2char`) : spécifiques aux petites racines comme préfixes/suffixes
3. **Remplacements localisés** (`replacements_list_for_localized_string`) : utilisés avec la notation @...@

### Annotation Ruby et ajustement de taille

Le format HTML avec annotations ruby ajuste automatiquement la taille des annotations en fonction du rapport entre la longueur du texte original et celle de la traduction :

- Pour les traductions très longues par rapport au mot original, le texte d'annotation sera plus petit
- Pour les traductions très courtes, le texte d'annotation sera plus grand
- L'application peut également insérer des sauts de ligne dans les annotations trop longues

### Optimisation pour les grands textes

Si vous traitez de grands volumes de texte :

- Activez le traitement parallèle
- Augmentez le nombre de processus simultanés (si votre ordinateur dispose de plusieurs cœurs)
- Notez que pour les textes très longs, seul un aperçu partiel sera affiché (les 247 premières lignes et les 3 dernières)

### Personnalisation des remplacements spécifiques

Pour les cas particuliers où vous souhaitez un contrôle précis :

- Utilisez la notation %...% pour préserver certaines parties du texte
- Utilisez la notation @...@ pour appliquer des remplacements spécifiques à certaines parties
- Créez un fichier JSON personnalisé pour définir des règles de remplacement spéciales

## Ressources supplémentaires

L'application offre des liens vers différentes versions linguistiques et ressources :

- Versions de l'application dans 14 langues différentes
- Documentation et instructions d'utilisation sur GitHub
- Fichiers d'exemple pour vous aider à démarrer

## Dépannage

Si vous rencontrez des problèmes :

- Vérifiez que vos fichiers CSV et JSON suivent le format attendu
- Assurez-vous que l'encodage de vos fichiers texte est en UTF-8
- Pour les textes avec des caractères spéciaux, privilégiez l'option "Accent sur la lettre"
- Si l'application semble bloquée pendant le traitement d'un texte volumineux, essayez de désactiver le traitement parallèle

---

Nous espérons que ce guide vous aidera à utiliser efficacement cet outil de remplacement et d'annotation pour textes en espéranto. N'hésitez pas à explorer les différentes options et formats pour trouver la configuration qui correspond le mieux à vos besoins.