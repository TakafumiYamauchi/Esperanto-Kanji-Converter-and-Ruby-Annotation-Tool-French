# Architecture technique de l'outil de remplacement de texte en espéranto

Ce document détaille l'architecture et le fonctionnement interne de l'application Streamlit dédiée au remplacement de texte en espéranto par des caractères chinois (kanji) et à l'ajout d'annotations HTML. Cette explication est destinée aux programmeurs de niveau intermédiaire qui connaissent déjà l'interface utilisateur, mais qui souhaitent comprendre les mécanismes sous-jacents.

## Structure générale de l'application

L'application est composée de quatre fichiers Python principaux :

1. **main.py** - Point d'entrée principal et interface utilisateur de base
2. **Page pour générer un fichier JSON pour remplacer du texte en espéranto par des chaînes (kanji).py** - Interface utilisateur secondaire pour la génération de fichiers JSON de substitution
3. **esp_text_replacement_module.py** - Module utilitaire pour les opérations de remplacement de texte
4. **esp_replacement_json_make_module.py** - Module utilitaire pour la génération des fichiers JSON de substitution

Étudions en détail le rôle et le fonctionnement de chaque composant.

## 1. Analyse du fichier principal (main.py)

### Architecture globale de main.py

Le fichier `main.py` structure l'application en plusieurs sections logiques :

1. **Importations et configuration initiale**
2. **Fonctions utilitaires**
3. **Configuration de l'interface Streamlit**
4. **Chargement des règles de substitution (JSON)**
5. **Paramètres avancés (traitement parallèle)**
6. **Sélection du format de sortie**
7. **Source du texte d'entrée**
8. **Formulaire de saisie et traitement du texte**
9. **Affichage des résultats et option de téléchargement**
10. **Liens vers les autres versions linguistiques**

### Points techniques clés de main.py

#### Configuration du multiprocessing

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass
```

Ce code configure le mode de démarrage des processus enfants sur 'spawn' pour éviter les erreurs de `PicklingError` lors de l'utilisation du multiprocessing avec Streamlit. Le mode 'spawn' crée un tout nouveau processus Python, ce qui est plus sûr mais légèrement plus lent que le mode 'fork' par défaut sous Unix.

#### Mise en cache des données avec @st.cache_data

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    # ...
```

Le décorateur `@st.cache_data` permet de mettre en cache le résultat de la fonction `load_replacements_lists()`. C'est crucial pour les performances car les fichiers JSON de substitution peuvent atteindre 50 Mo, et les recharger à chaque interaction utilisateur serait prohibitif en termes de temps d'exécution.

#### Structure des listes de remplacement

L'application manipule trois types de listes de remplacement principales :

1. **replacements_final_list** - Pour les remplacements globaux
2. **replacements_list_for_localized_string** - Pour les remplacements localisés (délimités par @)
3. **replacements_list_for_2char** - Pour le traitement spécial des racines de deux caractères

Ces listes contiennent des tuples sous la forme `(old, new, placeholder)`, où :
- `old` est la chaîne à remplacer
- `new` est la chaîne de remplacement
- `placeholder` est une chaîne temporaire unique utilisée pendant le processus de remplacement

#### Traitement du texte avec multiprocessing

Si l'option de traitement parallèle est activée, la fonction `parallel_process()` est utilisée :

```python
if use_parallel:
    processed_text = parallel_process(
        text=text0,
        num_processes=num_processes,
        # ... autres paramètres
    )
```

Cette fonction divise le texte en segments, les traite en parallèle, puis combine les résultats. Cela améliore considérablement les performances sur les textes volumineux.

#### Mécanisme de remplacement à deux étapes

Le cœur du traitement est la fonction `orchestrate_comprehensive_esperanto_text_replacement()` qui effectue le remplacement en plusieurs étapes :

1. Normalisation des espaces et conversion des caractères espéranto
2. Protection des segments délimités par %...%
3. Traitement spécial des segments délimités par @...@
4. Application des remplacements globaux
5. Traitement des racines de deux caractères
6. Restauration des segments protégés
7. Formatage final selon le type de sortie choisi

## 2. Analyse de la page de génération JSON

Le fichier `Page pour générer un fichier JSON pour remplacer du texte en espéranto par des chaînes (kanji).py` est une page Streamlit spécifique qui permet de générer les fichiers JSON utilisés par l'application principale.

### Processus de génération du JSON

La génération du fichier JSON se déroule en plusieurs étapes complexes :

1. **Chargement des données sources** :
   - Importation d'un CSV contenant les correspondances de racines espéranto et leurs traductions
   - Importation de fichiers JSON contenant des règles de décomposition de mots et des chaînes de substitution personnalisées

2. **Construction des structures de données temporaires** :
   - Création d'un dictionnaire initial de remplacements basé sur toutes les racines espéranto
   - Mise à jour de ce dictionnaire avec les correspondances du CSV importé

3. **Traitement des cas spéciaux** :
   - Gestion des suffixes verbaux (`as`, `is`, `os`, etc.)
   - Traitement des racines comportant le suffixe `-an` (membre)
   - Traitement des racines comportant le suffixe `-on` (fraction)
   - Gestion des racines de deux caractères
   - Ajustement des priorités de remplacement en fonction de la longueur des chaînes

4. **Application des règles de décomposition personnalisées** :
   - Traitement des règles définies dans le premier fichier JSON
   - Application des chaînes de substitution personnalisées du second fichier JSON

5. **Génération des variantes** :
   - Création de variantes pour les majuscules, minuscules et capitales
   - Ajustement des balises ruby pour les formats HTML

6. **Assemblage du fichier JSON final** :
   - Combinaison des trois types de listes de remplacement
   - Création d'un objet JSON structuré prêt à être téléchargé

### Aspects techniques importants

#### Utilisation de placeholders pour éviter les remplacements en cascade

Pour éviter les problèmes de remplacements en cascade (où un remplacement pourrait affecter un autre remplacement), l'application utilise des chaînes temporaires uniques (placeholders) :

```python
# Exemple simplifié du processus en deux étapes :
# 1. old → placeholder
text = text.replace(old, placeholder)
# 2. placeholder → new
text = text.replace(placeholder, new)
```

#### Système de priorités pour les remplacements

L'application utilise un système de priorités sophistiqué pour déterminer l'ordre des remplacements, basé principalement sur la longueur des chaînes :

```python
replacement_priority_by_length = len(esperanto_Word_before_replacement)*10000
```

Les chaînes plus longues ont une priorité plus élevée pour éviter les remplacements partiels indésirables.

## 3. Analyse des modules utilitaires

### esp_text_replacement_module.py

Ce module contient les fonctions de base pour la manipulation et le remplacement de texte :

1. **Dictionnaires de conversion de caractères espéranto** :
   - `x_to_circumflex` : conversion de la notation x (cx) vers l'accent circonflexe (ĉ)
   - `circumflex_to_x`, `x_to_hat`, `hat_to_x`, etc. pour d'autres conversions

2. **Fonctions de manipulation de texte** :
   - `replace_esperanto_chars()` : remplace les caractères selon un dictionnaire
   - `convert_to_circumflex()` : convertit toutes les notations vers la forme avec accent
   - `unify_halfwidth_spaces()` : normalise les différents types d'espaces

3. **Fonctions de remplacement sécurisé** :
   - `safe_replace()` : effectue des remplacements en deux étapes via des placeholders
   - `find_percent_enclosed_strings_for_skipping_replacement()` : identifie les segments à protéger
   - `find_at_enclosed_strings_for_localized_replacement()` : identifie les segments à traiter localement

4. **Fonction principale de remplacement** :
   - `orchestrate_comprehensive_esperanto_text_replacement()` : coordonne le processus complet

5. **Fonctions de traitement parallèle** :
   - `process_segment()` : traite un segment de texte
   - `parallel_process()` : divise et traite le texte en parallèle

### esp_replacement_json_make_module.py

Ce module fournit des fonctions spécialisées pour la génération des fichiers JSON :

1. **Fonctions de manipulation de format** :
   - `output_format()` : formate une paire (texte principal, contenu ruby) selon le format choisi
   - `measure_text_width_Arial16()` : calcule la largeur d'un texte en pixels pour l'ajustement de taille
   - `insert_br_at_half_width()`, `insert_br_at_third_width()` : insère des sauts de ligne pour les annotations longues

2. **Fonctions de personnalisation des balises HTML** :
   - `capitalize_ruby_and_rt()` : met en majuscule la première lettre dans les balises ruby
   - `remove_redundant_ruby_if_identical()` : supprime les balises ruby redondantes

3. **Fonctions de traitement parallèle pour la génération JSON** :
   - `process_chunk_for_pre_replacements()` : traite un segment de la liste de racines
   - `parallel_build_pre_replacements_dict()` : construit le dictionnaire de remplacements en parallèle

## 4. Flux de données et interactions entre les composants

Le diagramme conceptuel suivant illustre les interactions entre les différents composants :

```
┌─────────────────────┐       ┌───────────────────────────────┐
│                     │       │                               │
│      main.py        │◄─────►│ esp_text_replacement_module.py│
│   (UI principale)   │       │      (fonctions de base)      │
│                     │       │                               │
└─────────────────────┘       └───────────────────────────────┘
          ▲                                   ▲
          │                                   │
          │                                   │
          │                                   │
          ▼                                   ▼
┌─────────────────────┐       ┌───────────────────────────────┐
│  Page génération    │       │                               │
│       JSON.py       │◄─────►│esp_replacement_json_make_module│
│  (UI secondaire)    │       │  (génération de fichiers JSON) │
│                     │       │                               │
└─────────────────────┘       └───────────────────────────────┘
```

### Flux de données principal :

1. L'utilisateur interagit avec `main.py` ou `Page génération JSON.py`
2. Les entrées utilisateur sont traitées et converties en appels aux fonctions des modules utilitaires
3. Les modules utilitaires effectuent le traitement technique et renvoient les résultats
4. L'interface utilisateur affiche les résultats et offre des options supplémentaires

## 5. Techniques avancées utilisées dans l'application

### Expressions régulières

L'application utilise abondamment les expressions régulières pour la recherche et le remplacement de motifs complexes :

```python
# Exemple de regex pour les segments délimités par %
PERCENT_PATTERN = re.compile(r'%(.{1,50}?)%')

# Exemple de regex pour les segments délimités par @
AT_PATTERN = re.compile(r'@(.{1,18}?)@')

# Exemple de regex pour la détection des balises ruby
RUBY_PATTERN = re.compile(r'^(.*?)(<ruby>)([^<]+)(<rt[^>]*>)([^<]*?(?:<br>[^<]*?){0,2})(</rt>)(</ruby>)?(.*)$')
```

### Calcul dynamique de la taille des annotations Ruby

Une fonctionnalité impressionnante est l'ajustement automatique de la taille des annotations Ruby en fonction du ratio entre la longueur du texte principal et celle de l'annotation :

```python
width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
width_main = measure_text_width_Arial16(main_text, char_widths_dict)
ratio_1 = width_ruby / width_main

if ratio_1 > 6:
    return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
elif ratio_1 > (9/3):
    return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
# ... et ainsi de suite
```

Cette approche garantit une lisibilité optimale des annotations, quelle que soit leur longueur.

### Gestion des formats de caractères espéranto

L'application prend en charge trois formats différents pour les caractères spéciaux de l'espéranto :
1. Format avec accent circonflexe (ĉ, ĝ, etc.)
2. Format avec x (cx, gx, etc.)
3. Format avec ^ (c^, g^, etc.)

La conversion entre ces formats est gérée par des dictionnaires et des fonctions dédiées.

## 6. Optimisations et considérations de performance

### Mise en cache avec Streamlit

L'utilisation du décorateur `@st.cache_data` permet d'optimiser les opérations coûteuses comme le chargement de fichiers JSON volumineux.

### Traitement parallèle

Le traitement parallèle (multiprocessing) est implémenté pour deux opérations principales :
1. Le traitement du texte d'entrée dans `main.py`
2. La génération des dictionnaires de remplacement dans `Page génération JSON.py`

Ces optimisations permettent de traiter efficacement de grands volumes de texte ou des fichiers JSON complexes.

### Processus de remplacement en deux étapes

Le processus de remplacement en deux étapes (texte → placeholder → texte remplacé) évite les problèmes de remplacements en cascade et garantit l'intégrité du résultat final.

## Conclusion

Cette application démontre une architecture bien pensée pour traiter une tâche complexe de manipulation de texte. Les points forts de l'implémentation incluent :

1. **Modularité** - Séparation claire des responsabilités entre les différents fichiers et fonctions
2. **Performance** - Utilisation de techniques d'optimisation comme la mise en cache et le traitement parallèle
3. **Flexibilité** - Support de multiples formats d'entrée et de sortie
4. **Robustesse** - Gestion soignée des cas particuliers et des erreurs potentielles

Pour les développeurs souhaitant étendre cette application, les points d'extension naturels pourraient être :
- L'ajout de nouveaux formats de sortie
- L'amélioration des algorithmes de décomposition de mots
- L'intégration d'outils d'analyse linguistique plus avancés
- L'optimisation supplémentaire pour les très grands corpus de texte

Cette architecture peut également servir de modèle pour d'autres applications de traitement de texte ou de conversion linguistique.


# Flux de données et algorithmes clés

## Analyse approfondie du flux de données

Pour mieux comprendre les mécanismes internes de cette application, examinons en détail le flux de données pour les deux fonctionnalités principales : le remplacement de texte et la génération de fichier JSON.

## 1. Flux de données pour le remplacement de texte (main.py)

### Étape 1 : Chargement des ressources

Tout d'abord, l'application charge les ressources nécessaires :

```
Fichier JSON de substitution
           ↓
load_replacements_lists()
           ↓
3 listes de tuples (old, new, placeholder) :
- replacements_final_list (remplacements globaux)
- replacements_list_for_localized_string (remplacements localisés)
- replacements_list_for_2char (racines de 2 caractères)
```

Puis, elle charge les placeholders pour protéger certaines parties du texte :

```
Fichiers de placeholders
         ↓
import_placeholders()
         ↓
- placeholders_for_skipping_replacements (pour %...%)
- placeholders_for_localized_replacement (pour @...@)
```

### Étape 2 : Traitement du texte

Lorsque l'utilisateur soumet du texte, le flux de traitement est le suivant :

```
Texte espéranto (text0)
         ↓
parallel_process() ou orchestrate_comprehensive_esperanto_text_replacement()
         ↓
Texte transformé (processed_text)
         ↓
Conversion finale selon letter_type (ĉ, cx ou c^)
         ↓
Ajout d'en-têtes et pieds de page HTML si nécessaire
         ↓
Affichage et option de téléchargement
```

L'étape centrale, `orchestrate_comprehensive_esperanto_text_replacement()`, mérite une analyse plus détaillée :

```
Texte d'entrée
    ↓
1. Normalisation (espaces uniformisés, caractères espéranto convertis)
    ↓
2. Identification et remplacement temporaire des segments %...%
    ↓
3. Identification et traitement des segments @...@
    ↓
4. Application des remplacements globaux (old → placeholder → new)
    ↓
5. Traitement des racines de 2 caractères (en deux passes)
    ↓
6. Restauration des segments protégés (@...@ puis %...%)
    ↓
7. Formatage final (HTML, etc.)
    ↓
Texte de sortie
```

## 2. Flux de données pour la génération de fichier JSON

La génération d'un fichier JSON est un processus plus complexe qui comporte plusieurs étapes et transformations de données.

### Étape 1 : Chargement des données sources

```
CSV de correspondances espéranto → traductions
                ↓
DataFrame pandas (CSV_data_imported)
                ↓
Fichier JSON de règles de décomposition
                ↓
Liste Python (custom_stemming_setting_list)
                ↓
Fichier JSON de chaînes de substitution
                ↓
Liste Python (user_replacement_item_setting_list)
```

### Étape 2 : Construction du dictionnaire initial

```
Liste de toutes les racines espéranto
              ↓
Création de temporary_replacements_dict
              ↓
Mise à jour avec les traductions du CSV
              ↓
Transformation en temporary_replacements_list_1
              ↓
Tri par longueur → temporary_replacements_list_2
              ↓
Attribution de placeholders → temporary_replacements_list_final
```

### Étape 3 : Traitement de la liste E_stem_with_Part_Of_Speech_list

```
E_stem_with_Part_Of_Speech_list (liste complète des racines avec leur catégorie grammaticale)
                ↓
parallel_build_pre_replacements_dict() ou traitement séquentiel
                ↓
pre_replacements_dict_1 : { racine : [racine remplacée, catégorie grammaticale] }
                ↓
Transformations et ajustements de priorité → pre_replacements_dict_2
                ↓
Ajout de formes dérivées et ajustements supplémentaires → pre_replacements_dict_3
```

### Étape 4 : Traitement des cas spéciaux (AN, ON, règles personnalisées)

```
Traitement des listes AN (suffixe -an) et ON (suffixe -on)
                ↓
Application des règles de décomposition personnalisées
                ↓
Application des chaînes de substitution personnalisées
                ↓
Transformation en pre_replacements_list_1
                ↓
Tri par priorité → pre_replacements_list_2
                ↓
Suppression des ruby redondants → pre_replacements_list_3
                ↓
Génération des variantes (majuscules, minuscules, capitales) → pre_replacements_list_4
```

### Étape 5 : Assemblage des listes finales

```
pre_replacements_list_4
        ↓
replacements_final_list (avec placeholders ajustés)
        ↓
Création des listes pour les racines de 2 caractères :
- replacements_list_for_suffix_2char_roots
- replacements_list_for_prefix_2char_roots
- replacements_list_for_standalone_2char_roots
        ↓
Fusion → replacements_list_for_2char
        ↓
Création de replacements_list_for_localized_string
        ↓
Assemblage du JSON final avec les 3 listes
        ↓
Téléchargement du fichier JSON
```

## 3. Algorithmes clés analysés en détail

### Algorithme de remplacement sécurisé (safe_replace)

Cette fonction est au cœur du mécanisme de remplacement. Son algorithme en deux étapes évite les problèmes de remplacement en cascade :

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    valid_replacements = {}

    # Étape 1 : Remplacer les chaînes originales par des placeholders
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new

    # Étape 2 : Remplacer les placeholders par les chaînes finales
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)

    return text
```

Cette approche garantit que chaque partie du texte n'est remplacée qu'une seule fois, même si les chaînes de remplacement contiennent des sous-chaînes qui correspondent à d'autres motifs de remplacement.

### Algorithme de traitement parallèle

La fonction `parallel_process` divise le texte en segments et les traite en parallèle :

```python
def parallel_process(text, num_processes, ...) -> str:
    # Diviser le texte en lignes
    lines = re.findall(r'.*?\n|.+$', text)
    num_lines = len(lines)

    # Si peu de lignes, traiter séquentiellement
    if num_lines <= 1:
        return orchestrate_comprehensive_esperanto_text_replacement(text, ...)

    # Calculer la répartition des lignes par processus
    lines_per_process = max(num_lines // num_processes, 1)
    ranges = [(i * lines_per_process, (i + 1) * lines_per_process) for i in range(num_processes)]
    ranges[-1] = (ranges[-1][0], num_lines)  # Attribuer les lignes restantes au dernier processus

    # Traiter les segments en parallèle
    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(
            process_segment,
            [(lines[start:end], ...) for (start, end) in ranges]
        )

    # Combiner les résultats
    return ''.join(results)
```

Cette approche permet d'exploiter efficacement les architectures multi-cœurs pour accélérer le traitement des textes volumineux.

### Algorithme d'ajustement automatique de la taille des annotations Ruby

Un aspect sophistiqué de l'application est l'ajustement automatique de la taille des annotations Ruby en fonction du ratio entre la largeur du texte principal et celle de l'annotation :

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    if format_type == 'HTML格式_Ruby文字_大小调整':
        # Calculer les largeurs
        width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
        width_main = measure_text_width_Arial16(main_text, char_widths_dict)
        ratio_1 = width_ruby / width_main

        # Ajuster la taille et le formatage en fonction du ratio
        if ratio_1 > 6:
            # Très longue annotation : diviser en trois et réduire fortement la taille
            return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
        elif ratio_1 > (9/3):
            # Annotation longue : diviser en deux et réduire la taille
            return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
        elif ratio_1 > (9/4):
            # Annotation assez longue : réduire la taille sans diviser
            return f'<ruby>{main_text}<rt class="XS_S">{ruby_content}</rt></ruby>'
        # ... autres cas
```

Cet algorithme assure une présentation optimale des annotations quelle que soit leur longueur par rapport au texte principal.

### Algorithme de gestion des priorités de remplacement

L'application utilise un système sophistiqué pour déterminer l'ordre des remplacements, basé principalement sur la longueur des chaînes mais avec des ajustements pour des cas particuliers :

```python
# Priorité de base proportionnelle à la longueur
replacement_priority_by_length = len(esperanto_Word_before_replacement) * 10000

# Ajustements pour différents cas
if i==j[0]:  # Pas de changement réel (mot identique)
    priority = len(i) * 10000 - 3000  # Priorité réduite
else:  # Remplacement effectif
    priority = len(i) * 10000  # Priorité standard

# Ajustements spécifiques pour les suffixes grammaticaux
if "名词" in j[1]:  # Nom
    for k in ["o", "on", 'oj']:
        priority = j[2] + len(k) * 10000 - 3000  # Priorité ajustée pour la terminaison nominale
elif "动词" in j[1]:  # Verbe
    for k1, k2 in verb_suffix_2l_2.items():
        priority = j[2] + len(k1) * 10000 - 3000  # Priorité ajustée pour les terminaisons verbales
```

Ce système de priorités garantit que les remplacements les plus spécifiques (chaînes plus longues, formes complètes) sont effectués avant les remplacements plus généraux, évitant ainsi les remplacements partiels indésirables.

## 4. Structures de données clés

### Structure des listes de remplacement

Les listes de remplacement utilisées par l'application ont toutes une structure similaire :

```python
# Format : [(old, new, placeholder), ...]
replacements_final_list = [
    ("espero", "<ruby>esper<rt class=\"M_M\">希望</rt></ruby>o", "$12345$"),
    ("lingvo", "<ruby>lingv<rt class=\"M_M\">語言</rt></ruby>o", "$67890$"),
    # ...
]
```

### Structure du fichier JSON final

Le fichier JSON généré par la page secondaire a la structure suivante :

```json
{
  "全域替换用のリスト(列表)型配列(replacements_final_list)": [
    ["old1", "new1", "placeholder1"],
    ["old2", "new2", "placeholder2"],
    // ...
  ],
  "二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)": [
    ["old1", "new1", "placeholder1"],
    // ...
  ],
  "局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)": [
    ["old1", "new1", "placeholder1"],
    // ...
  ]
}
```

Cette structure permet à l'application principale de charger efficacement les règles de remplacement et de les appliquer selon les besoins.

## Conclusion

Cette analyse détaillée du flux de données et des algorithmes clés illustre la complexité et la sophistication de cette application de manipulation de texte. Les mécanismes de remplacement, les optimisations de performance et les ajustements automatiques de formatage témoignent d'une conception soignée et d'une attention particulière aux détails.

Pour les développeurs souhaitant étendre cette application, la compréhension de ces flux de données et algorithmes est essentielle pour maintenir la cohérence et l'efficacité du système.



# Détails d'implémentation et fonctionnalités avancées

## Implémentation des fonctionnalités clés

Dans cette section, nous analysons en profondeur les aspects les plus techniques de l'application, en expliquant comment les principales fonctionnalités sont implémentées.

## 1. Traitement des caractères spéciaux de l'espéranto

### Représentations multiples des caractères espéranto

L'espéranto utilise des caractères spéciaux avec accents circonflexes (ĉ, ĝ, ĥ, ĵ, ŝ) et un u-bref (ŭ). Dans le code, trois représentations différentes sont prises en charge :

1. **Notation avec accent circonflexe** : ĉ, ĝ, ĥ, ĵ, ŝ, ŭ
2. **Notation avec x** : cx, gx, hx, jx, sx, ux
3. **Notation avec ^** : c^, g^, h^, j^, s^, u^

Pour gérer ces différentes représentations, l'application utilise six dictionnaires de correspondance :

```python
x_to_circumflex = {'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ',
                   'Cx': 'Ĉ', 'Gx': 'Ĝ', 'Hx': 'Ĥ', 'Jx': 'Ĵ', 'Sx': 'Ŝ', 'Ux': 'Ŭ'}
circumflex_to_x = {'ĉ': 'cx', 'ĝ': 'gx', 'ĥ': 'hx', 'ĵ': 'jx', 'ŝ': 'sx', 'ŭ': 'ux',
                   'Ĉ': 'Cx', 'Ĝ': 'Gx', 'Ĥ': 'Hx', 'Ĵ': 'Jx', 'Ŝ': 'Sx', 'Ŭ': 'Ux'}
x_to_hat = {'cx': 'c^', 'gx': 'g^', 'hx': 'h^', 'jx': 'j^', 'sx': 's^', 'ux': 'u^',
            'Cx': 'C^', 'Gx': 'G^', 'Hx': 'H^', 'Jx': 'J^', 'Sx': 'S^', 'Ux': 'U^'}
hat_to_x = {'c^': 'cx', 'g^': 'gx', 'h^': 'hx', 'j^': 'jx', 's^': 'sx', 'u^': 'ux',
            'C^': 'Cx', 'G^': 'Gx', 'H^': 'Hx', 'J^': 'Jx', 'S^': 'Sx', 'U^': 'Ux'}
hat_to_circumflex = {'c^': 'ĉ', 'g^': 'ĝ', 'h^': 'ĥ', 'j^': 'ĵ', 's^': 'ŝ', 'u^': 'ŭ',
                     'C^': 'Ĉ', 'G^': 'Ĝ', 'H^': 'Ĥ', 'J^': 'Ĵ', 'S^': 'Ŝ', 'U^': 'Ŭ'}
circumflex_to_hat = {'ĉ': 'c^', 'ĝ': 'g^', 'ĥ': 'h^', 'ĵ': 'j^', 'ŝ': 's^', 'ŭ': 'u^',
                     'Ĉ': 'C^', 'Ĝ': 'G^', 'Ĥ': 'H^', 'Ĵ': 'J^', 'Ŝ': 'S^', 'Ŭ': 'U^'}
```

Ces dictionnaires sont utilisés par la fonction `replace_esperanto_chars()` pour convertir entre les différentes représentations :

```python
def replace_esperanto_chars(text, char_dict: Dict[str, str]) -> str:
    for original_char, converted_char in char_dict.items():
        text = text.replace(original_char, converted_char)
    return text
```

Pour standardiser le traitement, le texte est d'abord converti en notation avec accent circonflexe (forme Unicode) :

```python
def convert_to_circumflex(text: str) -> str:
    text = replace_esperanto_chars(text, hat_to_circumflex)
    text = replace_esperanto_chars(text, x_to_circumflex)
    return text
```

Puis, à la fin du traitement, il est reconverti selon la préférence de l'utilisateur (`letter_type`).

### Normalisation des espaces

Pour assurer un traitement cohérent, l'application normalise les différents types d'espaces Unicode en espaces ASCII standard :

```python
def unify_halfwidth_spaces(text: str) -> str:
    pattern = r"[\u00A0\u2002\u2003\u2004\u2005\u2006\u2007\u2008\u2009\u200A]"
    return re.sub(pattern, " ", text)
```

Cette normalisation est importante pour garantir que les motifs de remplacement fonctionnent correctement.

## 2. Système des placeholders

L'un des aspects les plus ingénieux de cette application est son système de "placeholders" (marqueurs de position) qui permet d'éviter les problèmes de remplacements en cascade.

### Structure et génération des placeholders

Les placeholders sont des chaînes de caractères uniques utilisées comme intermédiaires lors des remplacements. Ils sont stockés dans des fichiers texte externes et chargés au démarrage :

```python
def import_placeholders(filename: str) -> List[str]:
    with open(filename, 'r') as file:
        placeholders = [line.strip() for line in file if line.strip()]
    return placeholders
```

Trois ensembles de placeholders sont utilisés :
1. `placeholders_for_skipping_replacements` : Pour protéger les zones délimitées par %...%
2. `placeholders_for_localized_replacement` : Pour les zones délimitées par @...@
3. `imported_placeholders_for_global_replacement` : Pour les remplacements globaux

Chaque ensemble a un format spécifique (par exemple, `$12345$`, `@67890@`) pour éviter les collisions.

### Algorithme de remplacement à deux étapes

La fonction `safe_replace()` est au cœur du mécanisme de remplacement. Elle utilise une approche en deux temps :

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    valid_replacements = {}

    # Étape 1 : old → placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new

    # Étape 2 : placeholder → new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)

    return text
```

Cette approche en deux étapes évite les problèmes de remplacement en cascade. Par exemple, si nous devons remplacer "ami" par "友" et "amiko" par "友人", un remplacement direct pourrait transformer "amiko" en "友ko" par erreur. L'utilisation de placeholders intermédiaires garantit que chaque chaîne est remplacée correctement.

## 3. Implémentation des annotations Ruby HTML

Les annotations Ruby HTML permettent d'afficher de petits textes explicatifs au-dessus des mots principaux. L'application offre un contrôle fin sur l'apparence de ces annotations.

### Structure des balises Ruby

La structure de base d'une annotation Ruby est la suivante :

```html
<ruby>texte_principal<rt>annotation</rt></ruby>
```

L'application étend cette structure pour contrôler la taille de l'annotation :

```html
<ruby>texte_principal<rt class="M_M">annotation</rt></ruby>
```

### Calcul automatique de la taille des annotations

La fonction `output_format()` calcule automatiquement la taille appropriée pour les annotations en fonction du rapport entre la largeur du texte principal et celle de l'annotation :

```python
width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
width_main = measure_text_width_Arial16(main_text, char_widths_dict)
ratio_1 = width_ruby / width_main

if ratio_1 > 6:
    return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
elif ratio_1 > (9/3):
    return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
# ...
```

Pour calculer la largeur d'un texte, l'application utilise un dictionnaire de largeurs de caractères :

```python
def measure_text_width_Arial16(text, char_widths_dict: Dict[str, int]) -> int:
    total_width = 0
    for ch in text:
        char_width = char_widths_dict.get(ch, 8)  # 8px par défaut si caractère inconnu
        total_width += char_width
    return total_width
```

Ce dictionnaire est chargé depuis un fichier JSON (`Unicode_BMP全范围文字幅(宽)_Arial16.json`).

### Gestion des annotations longues

Pour les annotations particulièrement longues, l'application peut insérer des sauts de ligne pour améliorer la lisibilité :

```python
def insert_br_at_half_width(text, char_widths_dict: Dict[str, int]) -> str:
    total_width = measure_text_width_Arial16(text, char_widths_dict)
    half_width = total_width / 2
    current_width = 0
    insert_index = None

    for i, ch in enumerate(text):
        char_width = char_widths_dict.get(ch, 8)
        current_width += char_width
        if current_width >= half_width:
            insert_index = i + 1
            break

    if insert_index is not None:
        result = text[:insert_index] + "<br>" + text[insert_index:]
    else:
        result = text

    return result
```

Une fonction similaire, `insert_br_at_third_width()`, divise le texte en trois parties pour les annotations très longues.

### Styles CSS pour les annotations Ruby

La fonction `apply_ruby_html_header_and_footer()` ajoute les styles CSS nécessaires pour le rendu des annotations Ruby :

```python
def apply_ruby_html_header_and_footer(processed_text: str, format_type: str) -> str:
    if format_type in ('HTML格式_Ruby文字_大小调整','HTML格式_Ruby文字_大小调整_汉字替换'):
        ruby_style_head = """<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>大多数の环境中で正常に运行するRuby显示功能</title>
    <style>
    /* Styles CSS pour les annotations Ruby */
    </style>
  </head>
  <body>
  <p class="text-M_M">
"""
        ruby_style_tail = "</p></body></html>"
    # ...
```

Ces styles CSS définissent différentes tailles d'annotations (`rt.XXXS_S`, `rt.XXS_S`, etc.) et gèrent leur positionnement au-dessus du texte principal.

## 4. Traitement des marqueurs spéciaux (% et @)

L'application permet aux utilisateurs de contrôler précisément les remplacements à l'aide de deux types de marqueurs spéciaux.

### Protection des segments avec %...%

Les segments délimités par `%...%` sont protégés des remplacements. La fonction `find_percent_enclosed_strings_for_skipping_replacement()` identifie ces segments :

```python
PERCENT_PATTERN = re.compile(r'%(.{1,50}?)%')

def find_percent_enclosed_strings_for_skipping_replacement(text: str) -> List[str]:
    matches = []
    used_indices = set()

    for match in PERCENT_PATTERN.finditer(text):
        start, end = match.span()
        if start not in used_indices and end-2 not in used_indices:
            matches.append(match.group(1))
            used_indices.update(range(start, end))

    return matches
```

Ces segments sont temporairement remplacés par des placeholders, puis restaurés après le traitement principal.

### Remplacement localisé avec @...@

Les segments délimités par `@...@` sont soumis à un traitement spécial. La fonction `find_at_enclosed_strings_for_localized_replacement()` identifie ces segments :

```python
AT_PATTERN = re.compile(r'@(.{1,18}?)@')

def find_at_enclosed_strings_for_localized_replacement(text: str) -> List[str]:
    matches = []
    used_indices = set()

    for match in AT_PATTERN.finditer(text):
        start, end = match.span()
        if start not in used_indices and end-2 not in used_indices:
            matches.append(match.group(1))
            used_indices.update(range(start, end))

    return matches
```

Ces segments sont traités avec la liste de remplacement localisée (`replacements_list_for_localized_string`) avant le traitement global.

## 5. Gestion des racines de deux caractères

Un aspect particulier de l'application est le traitement spécial des racines espéranto de deux caractères, qui peuvent être des préfixes, des suffixes ou des mots autonomes.

### Types de racines de deux caractères

L'application distingue trois types de racines de deux caractères :

```python
suffix_2char_roots = ['ad', 'ag', 'am', 'ar', 'as', 'at', 'av', ...]  # Suffixes
prefix_2char_roots = ['al', 'am', 'av', 'bo', 'di', 'du', 'ek', ...]  # Préfixes
standalone_2char_roots = ['al', 'ci', 'da', 'de', 'di', 'do', ...]    # Mots autonomes
```

### Traitement spécifique des racines de deux caractères

Pour éviter les remplacements indésirables, les racines de deux caractères sont traitées avec des marqueurs spéciaux :

```python
# Exemples de création de listes de remplacement pour les suffixes
replacements_list_for_suffix_2char_roots = []
for i in range(len(suffix_2char_roots)):
    replaced_suffix = remove_redundant_ruby_if_identical(safe_replace(suffix_2char_roots[i], temporary_replacements_list_final))
    # Format pour les suffixes : $racine → $remplacement
    replacements_list_for_suffix_2char_roots.append([
        "$" + suffix_2char_roots[i],
        "$" + replaced_suffix,
        "$" + imported_placeholders_for_2char_replacement[i]
    ])
    # ... Ajout des variantes majuscules et capitales
```

Des approches similaires sont utilisées pour les préfixes (`racine$`) et les mots autonomes (` racine `).

### Double passe pour les racines de deux caractères

Pour garantir un traitement complet, l'application effectue deux passes de remplacement pour les racines de deux caractères :

```python
# Première passe
valid_replacements_for_2char_roots = {}
for old, new, placeholder in replacements_list_for_2char:
    if old in text:
        text = text.replace(old, placeholder)
        valid_replacements_for_2char_roots[placeholder] = new

# Deuxième passe
valid_replacements_for_2char_roots_2 = {}
for old, new, placeholder in replacements_list_for_2char:
    if old in text:
        place_holder_second = "!" + placeholder + "!"
        text = text.replace(old, place_holder_second)
        valid_replacements_for_2char_roots_2[place_holder_second] = new
```

Cette double passe permet de capturer les racines de deux caractères qui pourraient être révélées après la première passe de remplacement.

## 6. Traitement parallèle

Pour améliorer les performances sur les textes volumineux, l'application implémente un traitement parallèle utilisant le module `multiprocessing` de Python.

### Configuration du multiprocessing

Au démarrage, l'application configure le mode de démarrage des processus pour éviter les erreurs de sérialisation :

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass  # Déjà configuré
```

### Division et traitement parallèle du texte

La fonction `parallel_process()` divise le texte en segments et les traite en parallèle :

```python
def parallel_process(text, num_processes, ...):
    # Diviser le texte en lignes
    lines = re.findall(r'.*?\n|.+$', text)
    num_lines = len(lines)

    # Traitement séquentiel si peu de lignes
    if num_lines <= 1:
        return orchestrate_comprehensive_esperanto_text_replacement(text, ...)

    # Répartir les lignes entre les processus
    lines_per_process = max(num_lines // num_processes, 1)
    ranges = [(i * lines_per_process, (i + 1) * lines_per_process) for i in range(num_processes)]
    ranges[-1] = (ranges[-1][0], num_lines)  # Assigner les lignes restantes au dernier processus

    # Traiter les segments en parallèle
    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(
            process_segment,
            [(lines[start:end], ...) for (start, end) in ranges]
        )

    # Combiner les résultats
    return ''.join(results)
```

La fonction `process_segment()` traite un segment du texte :

```python
def process_segment(lines, ...):
    segment = ''.join(lines)
    result = orchestrate_comprehensive_esperanto_text_replacement(segment, ...)
    return result
```

### Parallélisation de la génération du fichier JSON

La génération du fichier JSON emploie également le traitement parallèle pour construire le dictionnaire de remplacements :

```python
def parallel_build_pre_replacements_dict(E_stem_with_Part_Of_Speech_list, replacements, num_processes):
    # Diviser les données en segments
    total_len = len(E_stem_with_Part_Of_Speech_list)
    chunk_size = -(-total_len // num_processes)
    chunks = []

    # Répartir les données entre les processus
    start_index = 0
    for i in range(num_processes):
        end_index = min(start_index + chunk_size, total_len)
        chunk = E_stem_with_Part_Of_Speech_list[start_index:end_index]
        chunks.append(chunk)
        start_index = end_index
        if start_index >= total_len:
            break

    # Traiter les chunks en parallèle
    with multiprocessing.Pool(num_processes) as pool:
        partial_dicts = pool.starmap(
            process_chunk_for_pre_replacements,
            [(chunk, replacements) for chunk in chunks]
        )

    # Fusionner les résultats
    merged_dict = {}
    for partial_d in partial_dicts:
        # ... Fusion des dictionnaires

    return merged_dict
```

Cette parallélisation permet de traiter efficacement de grandes quantités de données lors de la génération du fichier JSON.

## 7. Interface utilisateur Streamlit

L'application utilise Streamlit pour créer une interface utilisateur interactive et réactive.

### Structure de l'interface principale

L'interface principale (`main.py`) est organisée en sections logiques :

1. Configuration et titre :
   ```python
   st.set_page_config(
       page_title="Outil de remplacement de caractères (kanji) pour le texte en espéranto",
       layout="wide"
   )
   st.title("Remplacement du texte en espéranto par des kanjis ou ajout d'annotations en HTML (version étendue)")
   ```

2. Chargement du fichier JSON :
   ```python
   selected_option = st.radio(
       "Comment gérer le fichier JSON ? (chargement du fichier JSON de remplacement)",
       json_options,
       format_func=lambda x: "Utiliser le fichier JSON par défaut" if x == "デフォルトを使用する" else "Téléverser un fichier"
   )
   ```

3. Paramètres avancés :
   ```python
   with st.expander("Ouvrir la configuration pour le traitement parallèle"):
       # ...
       use_parallel = st.checkbox("Utiliser le traitement parallèle", value=False)
       num_processes = st.number_input(
           "Nombre de processus simultanés",
           min_value=2, max_value=4, value=4, step=1
       )
   ```

4. Format de sortie :
   ```python
   selected_display = st.selectbox(
       "Sélectionnez le format de sortie :",
       display_options,
       format_func=lambda key: options_french_labels[key]
   )
   ```

5. Formulaire de saisie :
   ```python
   with st.form(key='profile_form'):
       # ...
       text0 = st.text_area(
           "Veuillez saisir ici le texte en espéranto",
           height=150,
           value=initial_text
       )
       # ...
       submit_btn = st.form_submit_button('Envoyer')
       cancel_btn = st.form_submit_button("Annuler")
   ```

6. Affichage des résultats :
   ```python
   if "HTML" in format_type:
       tab1, tab2 = st.tabs(["Aperçu HTML", "Résultat (code HTML)"])
       with tab1:
           components.html(preview_text, height=500, scrolling=True)
       with tab2:
           st.text_area("Code HTML généré :", preview_text, height=300)
   else:
       # ...
   ```

### Fonctionnalités interactives

L'application utilise diverses fonctionnalités interactives de Streamlit :

- Onglets (`st.tabs()`) pour afficher les résultats sous différentes formes
- Sections dépliables (`st.expander()`) pour les paramètres avancés
- Formulaires (`st.form()`) pour la saisie de texte
- Sélecteurs (`st.selectbox()`, `st.radio()`) pour les options
- Téléchargement de fichiers (`st.download_button()`) pour exporter les résultats

### Gestion de l'état de la session

L'application utilise `st.session_state` pour conserver la saisie de l'utilisateur entre les interactions :

```python
# Récupérer la valeur précédente ou utiliser une chaîne vide
initial_text = st.session_state.get("text0_value", "")

# ...

# Sauvegarder la saisie actuelle
st.session_state["text0_value"] = text0
```

Cela permet à l'utilisateur de modifier ses paramètres sans perdre son texte.

## Conclusion et considérations techniques

Cette application démontre plusieurs bonnes pratiques et techniques avancées en développement Python :

1. **Modularité** - Séparation claire des responsabilités entre les différents modules
2. **Gestion efficace des données** - Utilisation de structures de données appropriées et optimisées
3. **Parallélisation** - Exploitation des architectures multi-cœurs pour les tâches intensives
4. **Internationalisation** - Support multilingue et traitement correct des caractères Unicode
5. **Interface utilisateur réactive** - Utilisation efficace des composants Streamlit

Pour les développeurs souhaitant étendre cette application, voici quelques pistes d'amélioration potentielles :

1. **Optimisation mémoire** - Réduire la consommation mémoire lors du traitement de très grands textes
2. **Intégration d'API** - Ajouter des services de traduction automatique pour générer automatiquement les fichiers CSV
3. **Analyse linguistique avancée** - Intégrer des outils d'analyse morphologique pour améliorer la décomposition des mots
4. **Visualisation interactive** - Ajouter des visualisations interactives pour explorer les correspondances entre racines

Cette architecture bien conçue et modulaire offre une base solide pour ces améliorations futures.


# Extensions et personnalisation de l'application

Cette dernière section est destinée aux programmeurs qui souhaitent personnaliser ou étendre l'application. Nous y explorerons les points d'extension possibles et fournirons des exemples concrets de modifications.

## Possibilités d'extension de l'application

### 1. Ajout de nouveaux formats de sortie

L'application supporte actuellement plusieurs formats de sortie (HTML avec Ruby, format avec parenthèses, etc.). Voici comment ajouter un nouveau format :

#### Étape 1 : Définir le nouveau format dans le dictionnaire d'options

Dans `main.py`, trouvez et modifiez le dictionnaire `options` :

```python
options = {
    'HTML格式_Ruby文字_大小调整': 'HTML格式_Ruby文字_大小调整',
    # ... formats existants
    'Nouveau_Format': 'Nouveau_Format',  # Ajouter cette ligne
}

# Ajouter également une étiquette française
options_french_labels = {
    # ... labels existants
    'Nouveau_Format': "Description en français du nouveau format",
}
```

#### Étape 2 : Implémenter la logique de formatage

Dans `esp_replacement_json_make_module.py`, modifiez la fonction `output_format()` pour prendre en charge le nouveau format :

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    # ... cas existants
    elif format_type == 'Nouveau_Format':
        # Logique spécifique au nouveau format
        return f'<span class="special">{main_text}<small>{ruby_content}</small></span>'
```

#### Étape 3 : Ajouter les styles nécessaires (si applicable)

Si votre format nécessite des styles CSS, modifiez la fonction `apply_ruby_html_header_and_footer()` dans `esp_text_replacement_module.py` :

```python
def apply_ruby_html_header_and_footer(processed_text: str, format_type: str) -> str:
    # ... cas existants
    elif format_type == 'Nouveau_Format':
        ruby_style_head = """<style>
        .special { position: relative; }
        .special small { position: absolute; top: -1em; font-size: 0.6em; color: blue; }
        </style>
        """
        ruby_style_tail = ""
    # ...
```

### 2. Intégration de nouvelles sources de données

Actuellement, l'application utilise des fichiers CSV et JSON pour les règles de remplacement. Voici comment intégrer une nouvelle source de données, comme une API de traduction :

#### Étape 1 : Ajouter les dépendances nécessaires

Dans les imports, ajoutez :

```python
import requests
```

#### Étape 2 : Créer une fonction pour interroger l'API

Ajoutez une nouvelle fonction dans `esp_replacement_json_make_module.py` :

```python
def fetch_translations_from_api(esperanto_roots: List[str], target_language: str = "fr") -> Dict[str, str]:
    """
    Récupère les traductions depuis une API externe.

    Args:
        esperanto_roots: Liste des racines espéranto à traduire
        target_language: Code de langue cible (fr, en, zh, etc.)

    Returns:
        Dictionnaire {racine_espéranto: traduction}
    """
    translations = {}
    api_url = "https://example-translation-api.com/translate"

    # Traiter les racines par lots pour éviter de surcharger l'API
    batch_size = 50
    for i in range(0, len(esperanto_roots), batch_size):
        batch = esperanto_roots[i:i+batch_size]

        # Préparer la requête
        payload = {
            "text": batch,
            "source": "eo",  # Code pour l'espéranto
            "target": target_language
        }

        # Envoyer la requête
        response = requests.post(api_url, json=payload)

        if response.status_code == 200:
            results = response.json()
            # Fusionner les résultats
            for j, root in enumerate(batch):
                translations[root] = results.get("translations", [])[j].get("text", root)

    return translations
```

#### Étape 3 : Intégrer la fonction dans le processus de génération JSON

Dans le fichier de génération JSON, modifiez le bouton de génération :

```python
if st.button("Créer le fichier JSON pour la substitution"):
    with st.spinner("Génération du fichier JSON de substitution en cours..."):
        # ... code existant

        # Option pour utiliser l'API
        use_api = st.checkbox("Utiliser l'API de traduction pour les racines manquantes", value=False)

        if use_api:
            # Identifier les racines sans traduction
            missing_roots = []
            for root in E_roots:
                root = root.strip()
                if root not in temporary_replacements_dict or temporary_replacements_dict[root][0] == root:
                    missing_roots.append(root)

            # Récupérer les traductions
            translations = fetch_translations_from_api(missing_roots)

            # Mettre à jour le dictionnaire
            for root, translation in translations.items():
                temporary_replacements_dict[root] = [
                    output_format(root, translation, format_type, char_widths_dict),
                    len(root)
                ]

        # ... suite du code
```

### 3. Amélioration de la décomposition des mots espéranto

L'application actuelle décompose les mots espéranto selon des règles prédéfinies. Nous pouvons améliorer ce processus avec une analyse morphologique plus avancée.

#### Étape 1 : Créer un module d'analyse morphologique

Créez un nouveau fichier `esperanto_morphology.py` :

```python
class EsperantoMorphologyAnalyzer:
    """Analyseur morphologique pour l'espéranto."""

    def __init__(self):
        # Charger les terminaisons grammaticales
        self.noun_endings = ['o', 'on', 'oj', 'ojn']
        self.adj_endings = ['a', 'an', 'aj', 'ajn']
        self.adv_endings = ['e']
        self.verb_endings = ['i', 'as', 'is', 'os', 'us', 'u']
        self.participle_endings = ['ant', 'int', 'ont', 'at', 'it', 'ot']

        # Charger les affixes communs
        self.prefixes = ['bo', 'dis', 'ek', 'eks', 'ge', 'mal', 're', 'pra', 'fi']
        self.suffixes = ['ad', 'aĵ', 'ar', 'ebl', 'ec', 'eg', 'ej', 'em', 'end', 'er', 'estr', 'et', 'id', 'ig', 'iĝ', 'il', 'in', 'ind', 'ing', 'ism', 'ist', 'obl', 'on', 'op', 'uj', 'ul', 'um']

    def decompose_word(self, word: str) -> List[str]:
        """
        Décompose un mot espéranto en ses éléments morphologiques.

        Args:
            word: Mot espéranto à décomposer

        Returns:
            Liste des éléments morphologiques (racine, préfixes, suffixes, terminaison)
        """
        # Convertir en minuscules pour simplifier le traitement
        word = word.lower()

        # Identifier la terminaison grammaticale
        ending = None
        word_stem = word

        # Vérifier les terminaisons dans l'ordre de spécificité
        for endings in [self.noun_endings, self.adj_endings, self.adv_endings, self.verb_endings, self.participle_endings]:
            for end in sorted(endings, key=len, reverse=True):
                if word.endswith(end) and len(word) > len(end):
                    ending = end
                    word_stem = word[:-len(end)]
                    break
            if ending:
                break

        # Identifier les préfixes
        prefixes = []
        prefix_found = True

        while prefix_found and word_stem:
            prefix_found = False
            for prefix in sorted(self.prefixes, key=len, reverse=True):
                if word_stem.startswith(prefix) and len(word_stem) > len(prefix):
                    prefixes.append(prefix)
                    word_stem = word_stem[len(prefix):]
                    prefix_found = True
                    break

        # Identifier les suffixes
        suffixes = []
        suffix_found = True

        while suffix_found and word_stem:
            suffix_found = False
            for suffix in sorted(self.suffixes, key=len, reverse=True):
                if word_stem.endswith(suffix) and len(word_stem) > len(suffix):
                    suffixes.insert(0, suffix)
                    word_stem = word_stem[:-len(suffix)]
                    suffix_found = True
                    break

        # Assembler le résultat
        result = []
        if prefixes:
            result.extend(prefixes)
        result.append(word_stem)  # Racine
        if suffixes:
            result.extend(suffixes)
        if ending:
            result.append(ending)

        return result
```

#### Étape 2 : Intégrer l'analyseur dans le processus de génération JSON

Dans le fichier de génération JSON, utilisez l'analyseur pour améliorer la décomposition des mots :

```python
# Importer l'analyseur morphologique
from esperanto_morphology import EsperantoMorphologyAnalyzer

# Instancier l'analyseur
morphology_analyzer = EsperantoMorphologyAnalyzer()

# Lors du traitement de E_stem_with_Part_Of_Speech_list
for i, j in enumerate(E_stem_with_Part_Of_Speech_list):
    if len(j) == 2:
        word = j[0]
        pos = j[1]

        # Utiliser l'analyseur morphologique pour obtenir une décomposition plus précise
        decomposition = morphology_analyzer.decompose_word(word)

        # Adapter le traitement en fonction de la décomposition
        if len(decomposition) > 1:
            root = decomposition[0]  # Racine principale

            # Traiter séparément chaque élément de la décomposition
            replaced_elements = []
            for element in decomposition:
                replaced_element = safe_replace(element, temporary_replacements_list_final)
                replaced_elements.append(replaced_element)

            # Combiner les éléments remplacés
            replaced_word = ''.join(replaced_elements)

            pre_replacements_dict_1[word] = [replaced_word, pos]
        else:
            # Traitement standard pour les mots non décomposables
            if word in pre_replacements_dict_1:
                # ... code existant
            else:
                # ... code existant
```

### 4. Ajout d'un mode d'apprentissage interactif

Une extension intéressante serait d'ajouter un mode d'apprentissage interactif qui aide l'utilisateur à apprendre l'espéranto.

#### Étape 1 : Créer une nouvelle page Streamlit

Créez un nouveau fichier nommé `Page d'apprentissage interactif.py` dans le dossier des pages :

```python
import streamlit as st
import random
from esp_text_replacement_module import convert_to_circumflex, safe_replace
import json

st.set_page_config(
    page_title="Apprentissage interactif de l'espéranto",
    layout="wide"
)

st.title("Mode d'apprentissage interactif de l'espéranto")
st.write("---")

# Charger le vocabulaire
@st.cache_data
def load_vocabulary(json_path: str):
    with open(json_path, 'r', encoding='utf-8') as f:
        data = json.load(f)

    replacements_final_list = data.get(
        "全域替换用のリスト(列表)型配列(replacements_final_list)", []
    )

    vocab = {}
    for old, new, _ in replacements_final_list:
        if len(old) >= 3 and not any(char.isdigit() for char in old):
            # Extraire la traduction des balises ruby si présente
            if "<ruby>" in new and "<rt>" in new:
                translation = new.split("<rt>")[1].split("</rt>")[0]
                vocab[old] = translation

    return vocab

# Charger le vocabulaire
vocab = load_vocabulary("./Appの运行に使用する各类文件/最终的な替换用リスト(列表)(合并3个JSON文件).json")

# Modes d'apprentissage
modes = ["Espéranto → Français", "Français → Espéranto", "Compléter la phrase"]
selected_mode = st.radio("Choisissez le mode d'apprentissage :", modes)

# Sélection du niveau de difficulté
difficulty = st.select_slider("Niveau de difficulté :",
                              options=["Débutant", "Intermédiaire", "Avancé"])

# Nombre de mots à apprendre selon le niveau
words_per_session = {"Débutant": 5, "Intermédiaire": 10, "Avancé": 15}
n_words = words_per_session[difficulty]

# Sélectionner des mots aléatoires du vocabulaire
if 'selected_words' not in st.session_state:
    st.session_state.selected_words = random.sample(list(vocab.items()), n_words)
    st.session_state.current_idx = 0
    st.session_state.score = 0

# Bouton pour générer une nouvelle série de mots
if st.button("Nouvelle série de mots"):
    st.session_state.selected_words = random.sample(list(vocab.items()), n_words)
    st.session_state.current_idx = 0
    st.session_state.score = 0

# Afficher le mot ou la phrase actuel
if st.session_state.current_idx < len(st.session_state.selected_words):
    current_word, current_translation = st.session_state.selected_words[st.session_state.current_idx]

    # Mode Espéranto → Français
    if selected_mode == "Espéranto → Français":
        st.subheader(f"Mot {st.session_state.current_idx + 1}/{n_words}")
        st.write(f"### {current_word}")

        user_translation = st.text_input("Entrez la traduction en français :")

        if st.button("Vérifier"):
            if user_translation.lower() == current_translation.lower():
                st.success("Correct ! 👍")
                st.session_state.score += 1
            else:
                st.error(f"Incorrect. La bonne réponse était : {current_translation}")

            # Passer au mot suivant
            st.session_state.current_idx += 1
            st.experimental_rerun()

    # Mode Français → Espéranto
    elif selected_mode == "Français → Espéranto":
        st.subheader(f"Mot {st.session_state.current_idx + 1}/{n_words}")
        st.write(f"### {current_translation}")

        user_word = st.text_input("Entrez le mot en espéranto :")

        if st.button("Vérifier"):
            if user_word.lower() == current_word.lower():
                st.success("Correct ! 👍")
                st.session_state.score += 1
            else:
                st.error(f"Incorrect. La bonne réponse était : {current_word}")

            # Passer au mot suivant
            st.session_state.current_idx += 1
            st.experimental_rerun()

    # Mode Compléter la phrase
    elif selected_mode == "Compléter la phrase":
        # Créer une phrase simple avec le mot
        if current_word.endswith('o'):  # Nom
            phrase = f"Mi vidas la _____."  # Je vois le _____.
            complete_phrase = f"Mi vidas la {current_word}."
        elif current_word.endswith('a'):  # Adjectif
            phrase = f"La _____ domo estas bela."  # La maison _____ est belle.
            complete_phrase = f"La {current_word} domo estas bela."
        elif current_word.endswith('i'):  # Verbe infinitif
            phrase = f"Mi volas _____."  # Je veux _____.
            complete_phrase = f"Mi volas {current_word}."
        elif current_word.endswith('e'):  # Adverbe
            phrase = f"Li parolas _____."  # Il parle _____.
            complete_phrase = f"Li parolas {current_word}."
        else:
            phrase = f"_____ estas grava vorto."  # _____ est un mot important.
            complete_phrase = f"{current_word} estas grava vorto."

        st.subheader(f"Phrase {st.session_state.current_idx + 1}/{n_words}")
        st.write(f"### {phrase}")
        st.write(f"Indice : {current_translation}")

        user_word = st.text_input("Complétez la phrase avec le mot correct :")

        if st.button("Vérifier"):
            if user_word.lower() == current_word.lower():
                st.success(f"Correct ! 👍\nPhrase complète : {complete_phrase}")
                st.session_state.score += 1
            else:
                st.error(f"Incorrect. La bonne réponse était : {current_word}\nPhrase complète : {complete_phrase}")

            # Passer au mot suivant
            st.session_state.current_idx += 1
            st.experimental_rerun()
else:
    # Afficher le score final
    st.success(f"Exercice terminé ! Votre score : {st.session_state.score}/{n_words}")

    # Proposer de recommencer avec de nouveaux mots
    if st.button("Recommencer avec de nouveaux mots"):
        st.session_state.selected_words = random.sample(list(vocab.items()), n_words)
        st.session_state.current_idx = 0
        st.session_state.score = 0
        st.experimental_rerun()
```

### 5. Amélioration des performances pour les textes volumineux

Pour les textes très volumineux, nous pouvons optimiser davantage le traitement parallèle en utilisant une approche par blocs.

#### Étape 1 : Créer une fonction de traitement par blocs

Ajoutez cette fonction dans `esp_text_replacement_module.py` :

```python
def chunk_based_processing(
    text: str,
    chunk_size: int,
    num_processes: int,
    placeholders_for_skipping_replacements: List[str],
    replacements_list_for_localized_string: List[Tuple[str, str, str]],
    placeholders_for_localized_replacement: List[str],
    replacements_final_list: List[Tuple[str, str, str]],
    replacements_list_for_2char: List[Tuple[str, str, str]],
    format_type: str
) -> str:
    """
    Traite le texte par blocs, en préservant les frontières de paragraphe.
    Cette approche est plus efficace pour les textes très volumineux.

    Args:
        text: Texte à traiter
        chunk_size: Taille approximative des blocs en nombre de caractères
        num_processes: Nombre de processus parallèles à utiliser
        ... autres paramètres identiques à parallel_process

    Returns:
        Texte traité
    """
    # Diviser le texte en paragraphes (préserver les sauts de ligne)
    paragraphs = text.split('\n')

    # Regrouper les paragraphes en blocs de taille approximative chunk_size
    chunks = []
    current_chunk = []
    current_size = 0

    for para in paragraphs:
        para_size = len(para) + 1  # +1 pour le saut de ligne

        if current_size + para_size > chunk_size and current_chunk:
            # Si ajouter ce paragraphe dépasse la taille limite et qu'il y a déjà du contenu,
            # finaliser le bloc actuel et en commencer un nouveau
            chunks.append('\n'.join(current_chunk))
            current_chunk = [para]
            current_size = para_size
        else:
            # Ajouter le paragraphe au bloc actuel
            current_chunk.append(para)
            current_size += para_size

    # Ajouter le dernier bloc s'il n'est pas vide
    if current_chunk:
        chunks.append('\n'.join(current_chunk))

    # Traiter les blocs en parallèle
    with multiprocessing.Pool(processes=min(num_processes, len(chunks))) as pool:
        processed_chunks = pool.starmap(
            orchestrate_comprehensive_esperanto_text_replacement,
            [(chunk,
              placeholders_for_skipping_replacements,
              replacements_list_for_localized_string,
              placeholders_for_localized_replacement,
              replacements_final_list,
              replacements_list_for_2char,
              format_type) for chunk in chunks]
        )

    # Recombiner les blocs traités
    if "HTML" in format_type:
        # Pour les formats HTML, fusionner correctement en gérant les balises d'en-tête/pied de page
        header = None
        footer = None
        content = []

        for i, chunk in enumerate(processed_chunks):
            # Extraire l'en-tête du premier bloc
            if i == 0 and "<head>" in chunk:
                header_end = chunk.find("<body>") + len("<body>")
                header = chunk[:header_end]
                content.append(chunk[header_end:])
            # Extraire le pied de page du dernier bloc
            elif i == len(processed_chunks) - 1 and "</body>" in chunk:
                footer_start = chunk.rfind("</body>")
                content.append(chunk[:footer_start])
                footer = chunk[footer_start:]
            else:
                content.append(chunk)

        return (header or "") + "".join(content) + (footer or "")
    else:
        # Pour les formats non-HTML, simplement joindre les blocs
        return "".join(processed_chunks)
```

#### Étape 2 : Intégrer cette fonction dans l'interface utilisateur

Dans `main.py`, modifiez le code de traitement pour inclure une option pour les très grands textes :

```python
# Après la section des paramètres avancés
very_large_text = st.checkbox("Texte très volumineux (> 100 000 caractères)", value=False)
if very_large_text:
    chunk_size = st.slider("Taille des blocs (caractères)", 5000, 50000, 20000, step=5000)

# Dans la section de traitement
if submit_btn:
    # ...
    if very_large_text:
        processed_text = chunk_based_processing(
            text=text0,
            chunk_size=chunk_size,
            num_processes=num_processes,
            placeholders_for_skipping_replacements=placeholders_for_skipping_replacements,
            replacements_list_for_localized_string=replacements_list_for_localized_string,
            placeholders_for_localized_replacement=placeholders_for_localized_replacement,
            replacements_final_list=replacements_final_list,
            replacements_list_for_2char=replacements_list_for_2char,
            format_type=format_type
        )
    elif use_parallel:
        processed_text = parallel_process(
            # ... paramètres existants
        )
    else:
        processed_text = orchestrate_comprehensive_esperanto_text_replacement(
            # ... paramètres existants
        )
```

## Bonnes pratiques pour les extensions

Lorsque vous étendez ou modifiez cette application, gardez à l'esprit ces bonnes pratiques :

### 1. Préserver la modularité

Maintenez la séparation des responsabilités entre les différents modules. Si vous ajoutez une nouvelle fonctionnalité majeure, envisagez de créer un nouveau module dédié plutôt que d'étendre les modules existants.

### 2. Tests unitaires

Avant d'intégrer vos modifications, créez des tests unitaires pour valider leur comportement. Par exemple, pour tester l'analyseur morphologique :

```python
import unittest
from esperanto_morphology import EsperantoMorphologyAnalyzer

class TestEsperantoMorphology(unittest.TestCase):
    def setUp(self):
        self.analyzer = EsperantoMorphologyAnalyzer()

    def test_simple_noun(self):
        result = self.analyzer.decompose_word("libro")
        self.assertEqual(result, ["libr", "o"])

    def test_complex_word(self):
        result = self.analyzer.decompose_word("malsanulejo")
        self.assertEqual(result, ["mal", "san", "ul", "ej", "o"])

    # ... autres tests

if __name__ == "__main__":
    unittest.main()
```

### 3. Documentation

Documentez clairement vos extensions, en particulier les API et les fonctions que d'autres développeurs pourraient utiliser. Utilisez des docstrings Python conformes à la norme PEP 257.

### 4. Performances

Gardez à l'esprit les performances, en particulier pour les opérations susceptibles d'être appliquées à de grands volumes de données. Utilisez le profilage pour identifier les goulots d'étranglement :

```python
import cProfile
import pstats

# Profiler une fonction
def profile_function(func, *args, **kwargs):
    profiler = cProfile.Profile()
    profiler.enable()
    result = func(*args, **kwargs)
    profiler.disable()
    stats = pstats.Stats(profiler).sort_stats('cumtime')
    stats.print_stats(20)  # Afficher les 20 fonctions les plus coûteuses
    return result

# Exemple d'utilisation
profile_function(
    orchestrate_comprehensive_esperanto_text_replacement,
    text,
    placeholders_for_skipping_replacements,
    replacements_list_for_localized_string,
    placeholders_for_localized_replacement,
    replacements_final_list,
    replacements_list_for_2char,
    format_type
)
```

### 5. Gestion des erreurs

Implémentez une gestion robuste des erreurs, en particulier pour les fonctionnalités qui interagissent avec des ressources externes (API, fichiers, etc.). Utilisez des blocs try-except avec des messages d'erreur informatifs :

```python
try:
    # Opération potentiellement risquée
    result = fetch_translations_from_api(roots)
except requests.RequestException as e:
    st.error(f"Erreur lors de la connexion à l'API : {str(e)}")
    st.info("Utilisation du mode hors ligne comme solution de repli.")
    result = fallback_translations(roots)
except ValueError as e:
    st.warning(f"Problème de format de données : {str(e)}")
    st.info("Les données ont été partiellement traitées.")
    result = partial_process(roots)
```

## Conclusion

L'application de remplacement de texte en espéranto offre une base solide pour de nombreuses extensions et personnalisations. En comprenant son architecture et ses mécanismes internes, vous pouvez l'adapter à vos besoins spécifiques ou y ajouter de nouvelles fonctionnalités innovantes.

Les suggestions d'extension présentées dans cette section ne sont qu'un point de départ. Avec une bonne compréhension du code sous-jacent, vous pouvez envisager des améliorations plus ambitieuses, comme l'intégration d'un moteur de traduction automatique, l'ajout d'un éditeur visuel interactif ou le développement d'une API REST pour permettre à d'autres applications d'utiliser ses fonctionnalités.

Quelle que soit la direction que vous choisissez, gardez à l'esprit les principes de conception qui ont guidé le développement initial : modularité, performance et expérience utilisateur intuitive.







Je vais continuer avec la dernière partie de l'explication technique pour les programmeurs francophones.

Je viens de terminer une explication technique complète de l'application de remplacement de texte en espéranto, spécifiquement conçue pour les programmeurs francophones de niveau intermédiaire. Cette documentation comprend quatre parties principales :

1. **Architecture technique de l'application** - Une vue d'ensemble de la structure du code, expliquant comment les quatre fichiers Python principaux interagissent et quels sont leurs rôles respectifs.

2. **Flux de données et algorithmes clés** - Une analyse détaillée des processus de traitement de texte et de génération de fichiers JSON, avec des diagrammes conceptuels et l'explication des algorithmes les plus importants.

3. **Détails d'implémentation et fonctionnalités avancées** - Un examen approfondi des techniques utilisées pour gérer les caractères espéranto, les placeholders, les annotations Ruby HTML et le traitement parallèle.

4. **Extensions et personnalisation de l'application** - Des exemples concrets de modifications que vous pourriez apporter à l'application, comme l'ajout de nouveaux formats de sortie, l'intégration de nouvelles sources de données ou l'amélioration des performances.

Cette documentation technique vous permettra de comprendre en profondeur le fonctionnement interne de l'application, bien au-delà de son interface utilisateur. Elle vous donne les connaissances nécessaires pour adapter l'application à vos besoins spécifiques ou pour développer des fonctionnalités similaires dans vos propres projets.
