# Documentation technique : Architecture et fonctionnement de l'outil de remplacement et d'annotation pour textes en espéranto

## Introduction

Cette documentation technique vise à expliquer en détail le fonctionnement interne de l'application de remplacement et d'annotation pour textes en espéranto. Destinée aux programmeurs de niveau intermédiaire, elle explore l'architecture, les algorithmes et les mécanismes qui permettent à l'application de transformer des textes en espéranto.

## Architecture globale de l'application

L'application est construite sur le framework Streamlit et s'articule autour de quatre composants principaux :

1. **`main.py`** : Script principal qui gère l'interface utilisateur et les fonctions de transformation de texte
2. **`Page pour générer un fichier JSON...`** : Module secondaire permettant de générer des fichiers JSON de règles de remplacement
3. **`esp_text_replacement_module.py`** : Module contenant les fonctions de traitement et remplacement de texte
4. **`esp_replacement_json_make_module.py`** : Module auxiliaire pour la création et manipulation des fichiers JSON

Le flux de données général suit ce schéma :

```
Entrée texte → Chargement des règles JSON → Traitement/Remplacement → Formatage → Sortie HTML/texte
```

## Structures de données fondamentales

### Les listes de remplacement

L'application utilise trois listes principales de règles de remplacement, stockées dans le fichier JSON :

1. **`replacements_final_list`** : Liste principale pour les remplacements globaux
   - Format : `[(ancien_texte, nouveau_texte, placeholder), ...]`
   - Priorité basée sur la longueur des mots (les mots plus longs sont traités en premier)

2. **`replacements_list_for_2char`** : Liste spécifique pour les racines de 2 caractères
   - Gère les préfixes, suffixes et racines indépendantes de 2 caractères
   - Subdivisée en `replacements_list_for_suffix_2char_roots`, `replacements_list_for_prefix_2char_roots` et `replacements_list_for_standalone_2char_roots`

3. **`replacements_list_for_localized_string`** : Liste pour les remplacements localisés (notation @...@)
   - Permet des remplacements spécifiques à certaines portions de texte

### Le système de placeholders

Les placeholders sont des chaînes de caractères uniques qui servent d'intermédiaires dans le processus de remplacement en deux étapes :

1. Remplacement initial : `texte original → placeholder`
2. Remplacement final : `placeholder → texte transformé`

Cette approche évite les problèmes de remplacement en cascade et les conflits entre règles.

## Analyse détaillée de `main.py`

### Initialisation et configuration

```python
# Importation des modules
import streamlit as st
import re
import io
import json
import pandas as pd
from typing import List, Dict, Tuple, Optional
import streamlit.components.v1 as components
import multiprocessing

# Configuration de multiprocessing pour éviter les erreurs de pickling
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass  # Ignore si déjà configuré
```

L'application utilise Streamlit pour l'interface et configure multiprocessing pour le traitement parallèle. Les modules typés (typing) permettent un code plus robuste et documenté.

### Fonction de chargement des fichiers JSON

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    """
    Charge le fichier JSON et retourne les trois listes de remplacement.
    """
    with open(json_path, 'r', encoding='utf-8') as f:
        data = json.load(f)
    replacements_final_list = data.get(
        "全域替换用のリスト(列表)型配列(replacements_final_list)", []
    )
    replacements_list_for_localized_string = data.get(
        "局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)", []
    )
    replacements_list_for_2char = data.get(
        "二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)", []
    )
    return (
        replacements_final_list,
        replacements_list_for_localized_string,
        replacements_list_for_2char,
    )
```

Point technique important : l'utilisation du décorateur `@st.cache_data` permet de mettre en cache le résultat de la fonction, évitant de recharger le fichier JSON (potentiellement volumineux, jusqu'à 50 Mo) à chaque interaction avec l'interface.

### Flux de traitement principal

Le flux de traitement dans `main.py` suit ces étapes :

1. **Chargement des règles** : Depuis le fichier JSON par défaut ou téléversé
2. **Chargement des placeholders** : Pour le traitement des sections à préserver ou à traiter spécifiquement
3. **Configuration du traitement parallèle** : Option pour accélérer le traitement des textes volumineux
4. **Sélection du format de sortie** : Différentes options de formatage (HTML, parenthèses, etc.)
5. **Traitement du texte** : Appel à `orchestrate_comprehensive_esperanto_text_replacement` ou `parallel_process`
6. **Formatage final** : Conversion des caractères spéciaux et ajout des en-têtes/pieds de page HTML

## Mécanismes de remplacement de texte

### La fonction pivot : `orchestrate_comprehensive_esperanto_text_replacement`

```python
def orchestrate_comprehensive_esperanto_text_replacement(
    text, 
    placeholders_for_skipping_replacements: List[str],
    replacements_list_for_localized_string: List[Tuple[str, str, str]],
    placeholders_for_localized_replacement: List[str],
    replacements_final_list: List[Tuple[str, str, str]],
    replacements_list_for_2char: List[Tuple[str, str, str]],
    format_type: str
) -> str:
    """
    Fonction principale qui orchestre toutes les étapes de remplacement.
    """
    # Étapes détaillées du traitement...
```

Cette fonction est le cœur du système de remplacement et exécute séquentiellement :

1. **Normalisation des espaces et conversion des caractères espéranto**
2. **Protection des sections avec %...%** : Les remplace par des placeholders
3. **Traitement des sections avec @...@** : Pour remplacements localisés
4. **Remplacement global** : Application de `replacements_final_list`
5. **Double remplacement des racines de 2 caractères** : Application de `replacements_list_for_2char`
6. **Restauration des placeholders** : Remplacement des placeholders par les textes transformés
7. **Adaptation au format HTML** : Conversion des sauts de ligne et des espaces

### Le mécanisme de remplacement sécurisé : `safe_replace`

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    """
    Effectue un remplacement en deux étapes pour éviter les remplacements en cascade.
    """
    valid_replacements = {}
    # Première étape : text → placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # Deuxième étape : placeholder → new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

Cette fonction est cruciale pour éviter les problèmes de remplacement en cascade. Par exemple, sans cette approche, le remplacement de "amir" par "友" suivi du remplacement de "ami" par "愛" pourrait transformer incorrectement "amir" en "愛r".

### Traitement parallèle

Pour les textes volumineux, l'application propose un mécanisme de traitement parallèle :

```python
def parallel_process(
    text: str,
    num_processes: int,
    # Autres paramètres...
) -> str:
    """
    Divise le texte en segments et les traite en parallèle.
    """
    # Division du texte en segments
    lines = re.findall(r'.*?\n|.+$', text)
    # Répartition des segments entre les processus
    # Traitement parallèle
    # Combinaison des résultats
```

Ce mécanisme divise le texte en segments (lignes), les distribue entre plusieurs processus, et recombine les résultats. Cette approche peut considérablement accélérer le traitement des textes longs.

## Analyse de `esp_text_replacement_module.py`

Ce module contient les fonctions essentielles pour la manipulation des textes en espéranto :

### Dictionnaires de conversion des caractères spéciaux

```python
x_to_circumflex = {
    'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ',
    'Cx': 'Ĉ', 'Gx': 'Ĝ', 'Hx': 'Ĥ', 'Jx': 'Ĵ', 'Sx': 'Ŝ', 'Ux': 'Ŭ'
}
# Plusieurs autres dictionnaires similaires...
```

Ces dictionnaires permettent la conversion entre différentes notations des caractères spéciaux de l'espéranto :
- Forme avec accent circonflexe (ĉ)
- Notation avec x (cx)
- Notation avec ^ (c^)

### Traitement des sections spéciales

```python
# Expressions régulières pour détecter les sections spéciales
PERCENT_PATTERN = re.compile(r'%(.{1,50}?)%')
AT_PATTERN = re.compile(r'@(.{1,18}?)@')

def find_percent_enclosed_strings_for_skipping_replacement(text: str) -> List[str]:
    """Extrait les chaînes entourées de % (à préserver)"""
    # ...

def find_at_enclosed_strings_for_localized_replacement(text: str) -> List[str]:
    """Extrait les chaînes entourées de @ (pour remplacement localisé)"""
    # ...
```

Ces fonctions utilisent des expressions régulières pour détecter et extraire les sections spéciales du texte. Notez les limites de longueur imposées : 50 caractères maximum pour les sections %...% et 18 caractères pour les sections @...@.

## Analyse de `esp_replacement_json_make_module.py`

Ce module fournit les fonctions nécessaires à la création des fichiers JSON de règles de remplacement :

### Formatage des sorties

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    """
    Formate le texte espéranto et sa traduction selon le format choisi.
    """
    if format_type == 'HTML格式_Ruby文字_大小调整':
        # Calcul des largeurs et ratios
        width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
        width_main = measure_text_width_Arial16(main_text, char_widths_dict)
        ratio_1 = width_ruby / width_main
        
        # Différentes classes de taille selon le ratio
        if ratio_1 > 6:
            return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
        elif ratio_1 > (9/3):
            # ... autres conditions
```

Cette fonction sophistiquée adapte la présentation des annotations ruby en fonction du rapport entre la longueur du texte original et celle de la traduction. Pour les traductions très longues, elle ajoute même des sauts de ligne (`insert_br_at_third_width`, `insert_br_at_half_width`).

### Mesure des largeurs de texte

```python
def measure_text_width_Arial16(text, char_widths_dict: Dict[str, int]) -> int:
    """
    Calcule la largeur totale du texte en pixels, selon la police Arial 16.
    """
    total_width = 0
    for ch in text:
        char_width = char_widths_dict.get(ch, 8)
        total_width += char_width
    return total_width
```

Cette fonction précise permet de calculer la largeur réelle du texte en tenant compte de la largeur variable des caractères dans la police Arial 16. Ces informations sont stockées dans un dictionnaire JSON précompilé.

## Analyse de la page de génération des fichiers JSON

### Structure du traitement

La page de génération des fichiers JSON fonctionne par étapes :

1. **Importation des données CSV** : Contenant les correspondances racines-traductions
2. **Chargement des fichiers JSON** : Pour les règles de décomposition et les chaînes personnalisées
3. **Construction d'un dictionnaire temporaire** : À partir des racines espéranto
4. **Application des règles de traduction** : À partir du CSV
5. **Traitement spécifique des terminaisons** : Pour les verbes, noms, adjectifs
6. **Construction des trois listes de remplacement** : Globale, localisée, racines de 2 caractères
7. **Exportation du fichier JSON combiné**

### Traitement des cas spéciaux

Un aspect particulièrement intéressant est le traitement des terminaisons grammaticales et des racines courtes :

```python
# Variables pour différentes catégories grammaticales
verb_suffix_2l = {
    'as':'as', 'is':'is', 'os':'os', 'us':'us','at':'at','it':'it','ot':'ot',
    'ad':'ad','iĝ':'iĝ','ig':'ig','ant':'ant','int':'int','ont':'ont'
}
suffix_2char_roots = ['ad', 'ag', 'am', 'ar', 'as', /* ... */]
prefix_2char_roots = ['al', 'am', 'av', 'bo', /* ... */]
standalone_2char_roots = ['al', 'ci', 'da', 'de', /* ... */]
```

Ces listes permettent de gérer correctement les affixes grammaticaux et les petites racines qui peuvent porter à confusion dans le processus de remplacement.

### Gestion des priorités de remplacement

```python
# Définition de la priorité basée sur la longueur des mots
replacement_priority_by_length = len(esperanto_Word_before_replacement)*10000

# Priorités spéciales pour certains cas
if "verbo_s1" in i[2]:
    for k1,k2 in verb_suffix_2l_2.items():
        pre_replacements_dict_3[esperanto_Word_before_replacement + k1] = [
            Replaced_String + k2, 
            replacement_priority_by_length+len(k1)*10000
        ]
```

Le système de priorité est basé sur la longueur des mots multipliée par 10000, avec des ajustements pour les cas spéciaux comme les terminaisons verbales. Cette approche garantit que les mots plus longs sont traités avant les plus courts, évitant ainsi des remplacements partiels incorrects.

## Architecture des fichiers JSON et format des données

Le fichier JSON principal combine trois structures :

```json
{
  "全域替换用のリスト(列表)型配列(replacements_final_list)": [
    ["aĉet", "<ruby>aĉet<rt class=\"M_M\">acheter</rt></ruby>", "$20987$"],
    // ...
  ],
  "局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)": [
    // ...
  ],
  "二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)": [
    // ...
  ]
}
```

Chaque entrée contient :
- Le texte original à remplacer
- Le texte de remplacement (souvent avec balisage HTML)
- Un placeholder unique

## Fonctionnalités d'optimisation avancées

### Capitalisation intelligente des balises Ruby

```python
def capitalize_ruby_and_rt(text: str) -> str:
    """
    Met en majuscule le premier caractère du texte parent et du texte ruby.
    """
    def replacer(match):
        # Extraction des groupes capturés par l'expression régulière
        # Capitalisation sélective
        # ...
    
    replaced_text = RUBY_PATTERN.sub(replacer, text)
    # Fallback sur la capitalisation standard si nécessaire
    return replaced_text
```

Cette fonction utilise des expressions régulières complexes pour capitaliser correctement le texte à l'intérieur des balises ruby, ce qui est crucial pour maintenir la cohérence visuelle du texte transformé.

### Suppression des balises Ruby redondantes

```python
def remove_redundant_ruby_if_identical(text: str) -> str:
    """
    Supprime les balises ruby quand le texte parent et le ruby sont identiques.
    """
    def replacer(match: re.Match) -> str:
        group1 = match.group(1)
        group2 = match.group(2)
        if group1 == group2:
            return group1
        else:
            return match.group(0)
    
    replaced_text = IDENTICAL_RUBY_PATTERN.sub(replacer, text)
    return replaced_text
```

Cette optimisation élimine les annotations ruby inutiles lorsque le texte d'annotation est identique au texte principal, améliorant ainsi la lisibilité et réduisant la taille du HTML généré.

## Conclusion de la première partie

Cette première partie de la documentation technique a couvert l'architecture globale de l'application, les principales structures de données, et les mécanismes fondamentaux de traitement du texte. Dans la suite, nous approfondirons les aspects algorithmiques du traitement parallèle, les subtilités de la gestion des racines espéranto, et les techniques d'optimisation utilisées pour gérer efficacement les textes volumineux.