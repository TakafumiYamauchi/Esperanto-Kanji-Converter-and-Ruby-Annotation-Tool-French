# Documentation technique : Outil de remplacement de texte en espéranto et d'annotation Ruby

## Introduction

Cette documentation technique est destinée aux programmeurs de niveau intermédiaire souhaitant comprendre le fonctionnement interne de l'application Streamlit de remplacement de texte en espéranto. Nous partirons du principe que vous comprenez déjà l'interface utilisateur et souhaitez explorer les mécanismes sous-jacents.

L'application est conçue pour transformer des textes en espéranto de deux manières principales :
1. Remplacer des mots/racines en espéranto par des caractères kanji ou des traductions
2. Ajouter des annotations Ruby HTML au-dessus des mots pour faciliter la compréhension

## Architecture globale de l'application

### Structure des fichiers

L'application est composée de quatre fichiers Python principaux :

1. **main.py** : Point d'entrée principal de l'application Streamlit
2. **Page pour générer un fichier JSON...** : Page secondaire pour créer des fichiers JSON de règles de remplacement
3. **esp_text_replacement_module.py** : Module contenant les fonctions de transformation de texte
4. **esp_replacement_json_make_module.py** : Module pour la génération des fichiers JSON de remplacement

### Flux de données général

Le flux de données dans l'application suit généralement ce schéma :

```
Entrée (texte espéranto) → Chargement des règles de remplacement (JSON) → 
Prétraitement du texte → Application des règles de remplacement → 
Formatage du résultat (HTML/parenthèses) → Affichage/Téléchargement
```

## Analyse détaillée du module principal (main.py)

### Initialisation et configuration

Le fichier `main.py` commence par configurer l'environnement Streamlit et importer les dépendances nécessaires :

```python
import streamlit as st
import re
import io
import json
import pandas as pd
from typing import List, Dict, Tuple, Optional
import streamlit.components.v1 as components
import multiprocessing
```

Un point important à noter est la configuration de multiprocessing :

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass
```

Cet élément est crucial car Streamlit peut rencontrer des problèmes de `PicklingError` avec multiprocessing. L'utilisation du mode 'spawn' évite ces erreurs.

### Fonction clé : load_replacements_lists

Cette fonction est décorée avec `@st.cache_data` pour optimiser les performances :

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    """
    Charge un fichier JSON et retourne trois listes en tant que tuple :
    1) replacements_final_list
    2) replacements_list_for_localized_string
    3) replacements_list_for_2char
    """
```

Cette mise en cache est essentielle puisque les fichiers JSON de remplacement peuvent être volumineux (jusqu'à 50 Mo).

### Les trois listes de remplacement

L'application utilise trois types différents de listes pour les remplacements :

1. **replacements_final_list** : liste principale pour les remplacements globaux
2. **replacements_list_for_localized_string** : pour les remplacements localisés (avec @...@)
3. **replacements_list_for_2char** : pour les racines de deux caractères (préfixes/suffixes)

Chaque liste contient des tuples de la forme `(old, new, placeholder)`.

### Interface utilisateur principale

L'interface est construite avec des composants Streamlit comme :
- `st.radio` pour les options de chargement JSON
- `st.selectbox` pour le format de sortie
- `st.form` pour le formulaire principal
- `st.tabs` pour l'affichage des résultats

### Le cœur du traitement : orchestration des remplacements

Lorsque l'utilisateur soumet le formulaire, l'application traite le texte selon que le traitement parallèle est activé ou non :

```python
if use_parallel:
    processed_text = parallel_process(
        text=text0,
        num_processes=num_processes,
        # ... autres paramètres ...
    )
else:
    processed_text = orchestrate_comprehensive_esperanto_text_replacement(
        text=text0,
        # ... autres paramètres ...
    )
```

La fonction `orchestrate_comprehensive_esperanto_text_replacement` est le point central du traitement, que nous examinerons plus en détail lors de l'analyse du module `esp_text_replacement_module.py`.

## Le module de remplacement de texte (esp_text_replacement_module.py)

Ce module contient les fonctions essentielles pour transformer le texte espéranto.

### Dictionnaires de conversion des caractères espéranto

Plusieurs dictionnaires sont définis pour gérer les différentes notations des caractères spéciaux en espéranto :

```python
x_to_circumflex = {
    'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ',
    'Cx': 'Ĉ', 'Gx': 'Ĝ', 'Hx': 'Ĥ', 'Jx': 'Ĵ', 'Sx': 'Ŝ', 'Ux': 'Ŭ'
}
```

Et d'autres dictionnaires similaires pour les conversions entre différentes notations.

### La fonction safe_replace

Cette fonction est fondamentale pour le processus de remplacement :

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    """
    Reçoit une liste de tuples (old, new, placeholder) et
    effectue un remplacement en deux étapes : old → placeholder → new
    """
```

L'approche en deux étapes (utilisant des placeholders) est cruciale pour éviter les problèmes de remplacement en cascade ou de conflit. Par exemple, si on remplace directement "a" par "b" puis "ab" par "c", on pourrait obtenir des résultats incorrects à cause de l'ordre des remplacements.

### Les expressions régulières pour le traitement spécial

Le module utilise des expressions régulières pour identifier les parties spéciales du texte :

```python
PERCENT_PATTERN = re.compile(r'%(.{1,50}?)%')  # Pour les segments à conserver tels quels
AT_PATTERN = re.compile(r'@(.{1,18}?)@')  # Pour les segments à transformer localement
```

### Fonction principale : orchestrate_comprehensive_esperanto_text_replacement

Cette fonction orchestratrice combine toutes les étapes du traitement :

1. Normalisation des espaces et conversion des caractères espéranto
2. Traitement des parties délimitées par % (à conserver)
3. Traitement des parties délimitées par @ (remplacement localisé)
4. Application des remplacements globaux
5. Application des remplacements pour les racines de deux caractères
6. Restauration des placeholders
7. Formatage final (HTML/parenthèses)

### Traitement parallèle

Pour optimiser les performances sur des textes volumineux, le module implémente un traitement parallèle :

```python
def parallel_process(
    text: str,
    num_processes: int,
    # ... autres paramètres ...
) -> str:
```

Cette fonction découpe le texte en segments (par lignes) et les traite en parallèle à l'aide de `multiprocessing.Pool`.

## Le module de génération JSON (esp_replacement_json_make_module.py)

Ce module est spécifique à la création des fichiers JSON de règles de remplacement.

### Fonctions de formatage de sortie

La fonction `output_format` détermine comment un mot espéranto et sa traduction seront combinés selon le format choisi :

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    """
    Combine le texte espéranto (main_text) et sa traduction (ruby_content)
    selon le format spécifié
    """
```

Cette fonction gère différents formats, notamment :
- HTML avec ajustement de la taille des annotations Ruby
- HTML basique
- Format parenthèses
- Remplacement simple

### Gestion des largeurs de caractères

Une particularité intéressante est l'utilisation d'un dictionnaire de largeurs de caractères pour ajuster la taille des annotations Ruby :

```python
def measure_text_width_Arial16(text, char_widths_dict: Dict[str, int]) -> int:
    """
    Calcule la largeur totale d'un texte en pixels à l'aide d'un dictionnaire de largeurs
    """
```

Pour les annotations particulièrement longues, le module peut même insérer des sauts de ligne aux bons endroits :

```python
def insert_br_at_half_width(text, char_widths_dict: Dict[str, int]) -> str:
    """
    Insère une balise <br> à la moitié de la largeur du texte
    """
```

### Construction parallèle du dictionnaire de remplacements

Pour optimiser la génération des fichiers JSON volumineux, le module utilise également le traitement parallèle :

```python
def parallel_build_pre_replacements_dict(
    E_stem_with_Part_Of_Speech_list: List[List[str]],
    replacements: List[Tuple[str, str, str]],
    num_processes: int = 4
) -> Dict[str, List[str]]:
```

## La page de génération de fichiers JSON

Le fichier `Page pour générer un fichier JSON...` est une page Streamlit dédiée à la création des fichiers JSON de règles de remplacement.

### Étapes principales de la génération

La génération des fichiers JSON suit ces étapes :
1. Chargement d'un fichier CSV de correspondances racines-traductions
2. Construction d'un dictionnaire temporaire de remplacements
3. Traitement parallèle pour construire les listes de remplacements
4. Ajustement des priorités et optimisation des remplacements
5. Génération des trois listes finales (globale, localisée, racines de 2 caractères)
6. Combinaison des listes en un seul fichier JSON

### Gestion complexe des priorités

Un aspect particulièrement sophistiqué est la gestion des priorités de remplacement :

```python
if j[2]==20000:
    # Traitement spécial pour les racines de 2 caractères
    if "名词" in j[1]:  # Nom
        for k in ["o","on",'oj']:
            # ...
    if "形容词" in j[1]:  # Adjectif
        for k in ["a","aj",'an']:
            # ...
```

Ce code montre comment l'application gère différemment les racines selon leur catégorie grammaticale, ajoutant automatiquement les terminaisons appropriées et ajustant les priorités.

## Aspects techniques avancés

### Système de placeholders

L'application utilise un système sophistiqué de placeholders pour éviter les conflits lors des remplacements :

1. Les placeholders pour les parties à ne pas remplacer (%...%) : `$20987$-$499999$`
2. Les placeholders pour les racines de deux caractères : `$13246$-$19834$`
3. Les placeholders pour les remplacements localisés (@...@) : `@20374@-@97648@`

Ces placeholders sont importés depuis des fichiers texte pour garantir leur unicité.

### Optimisation des performances

Plusieurs techniques sont utilisées pour optimiser les performances :

1. Mise en cache des données avec `@st.cache_data`
2. Traitement parallèle avec `multiprocessing`
3. Tri des règles de remplacement par longueur pour traiter d'abord les plus longs
4. Utilisation de structures de données efficaces (dictionnaires pour les accès rapides)

### Gestion des formats de caractères espéranto

L'application peut gérer trois formats différents pour les caractères spéciaux de l'espéranto :
1. Format avec accent circonflexe (ĉ, ĝ, etc.)
2. Format avec x (cx, gx, etc.)
3. Format avec ^ (c^, g^, etc.)

La conversion entre ces formats est gérée par des fonctions dédiées comme `convert_to_circumflex()`.

Dans ma prochaine partie, je vais détailler les algorithmes clés et les structures de données importantes, ainsi que fournir des exemples concrets de flux de traitement.