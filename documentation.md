# Diagramme d'interaction 2D/3D
_Programme développé par Xavier MORIN_

Le programme DI permet de calculer le diagramme d’interaction de toute section composée de contours et de nuages de points associés à des lois quelconques de déformation-contraintes telles que les définissent les règlements de calculs.

Le programme est développé pour fonctionner directement sur Excel. Par l’intermédiaire de lignes de commandes, l’utilisateur doit fournir la géométrie de la section, les lois déformation-contrainte et les pivots de calcul.

La première partie de cette note présente la théorie et les principes fondamentaux retenus pour le calcul.
La seconde partie présente le fonctionnement du programme et de chacune des commandes autorisées.
La troisième partie présente l’architecture du programme et s’adresse aux développeurs désireux de comprendre le fonctionnement du programme ou souhaitant apporter des commentaires et conseils pour les développements futurs.

Pour tout bug constaté ou tout problème lié à l’utilisation du programme (si l’aide n’apporte pas la réponse à la question), merci de me contacter.


## Théorie et principes du programme
### Diagramme d'interaction - qu'est-ce que c'est ?
Un diagramme d’interaction est une représentation graphique du domaine de validité des torseurs applicables à une section.
Si pour une section un torseur est défini par deux variables (N et My par exemple), alors le domaine de validité des torseurs sera analogue à une surface délimitée par un contour.
Si un torseur est défini par trois variables (N, My et Mz par exemple), alors le domaine de validité des torseurs sera analogue à un volume délimité par une surface.

Le domaine de validité des torseurs applicables à une section est calculé à partir du domaine de validité des états de déformation de la section.
Les états de déformation de la section sont limités par les critères portant sur les matériaux :

*	limitation en déformation (généralement aux ELU) ;
*	limitation en contrainte (généralement aux ELS).

## Hypothèses
_en cours de rédaction_

<!--

« addition » des éléments de section. (ex : si des aciers se trouvent dans un contour, le béton au droit des armatures n’est pas supprimé).

-->

# Traitement des contours
_en cours de rédaction_

<!--
sens important
-->

## Fonctionnement du programme
### Repère, convention de signe et unités
La section étudiée est décrite dans le plan (Oyz).

La convention de signe sur les déformations est la suivante :

* déformation positive = raccourcissement ;
* déformation négative = allongement.

Les torseurs (N, My, Mz) sont calculés comme suit :
N=∫▒〖σ.dS〗  ; M_y=∫▒〖σ.z.dS〗  ; M_z=∫▒〖σ.(-y).dS〗

Ainsi, si l’on renseigne des lois de comportements qui traduisent une compression lorsque la déformation est positive, on obtient la convention de signe suivante sur les torseurs calculés :

* un effort normal N positif correspond à une compression ;
* un moment My positif comprime la fibre z+ ;
* un moment Mz positif comprime la fibre y-.

            ^z
            |
      x     |        
      <-----+     (N>0; My>0 et Mz >0 si sens direct)
             \
              \
               y


Le programme est adimensionnel mais les unités suivantes sont recommandées :

* MPa pour les contraintes ;
* MN, MN.m pour les efforts et moments ;
* m, m2, m4 pour les dimensions/positions, surfaces et inerties.

Tout autre système d’unités est autorisé mais doit être cohérent sur l’ensemble du fichier pour que les résultats aient un sens (par exemple kN avec mm).

Par ailleurs, les déformations sont sans unité et les angles sont en radians.

### Présentation générale des lignes de commandes

Les commandes sont saisies dans les 5 premières colonnes d’une feuille de calcul Excel.
Ces commandes doivent être saisies entre une balise de début d’instructions DI et une balise de fin d’instruction EOF (End Of File). Rien de ce qui sera écrit avant DI ou après EOF ne sera interprété par le programme.

La colonne A ne peut contenir que les commandes autorisées ; toute autre saisie renvoie une erreur et force l’arrêt du programme.
Les colonnes B, C, D et E contiennent les paramètres associés à chacune des commandes.

**Bloc d'instructions**

Certaines commandes peuvent nécessiter plusieurs lignes pour leur définition (contours, nuages de points, loi déformation-contrainte, pivots) ; ces lignes sont interprétées par le programme jusqu’à l’apparition d’une ligne vide ou d’une nouvelle commande en colonne A.
On appellera par la suite « bloc d’instruction », des lignes définies par le même mot-clef en colonne A.

Bloc d'instruction type :

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `COMMANDE` | _Param 1_ | _(Param 2)_ | / | / |
| X | _y1_ | _z1_ | / | / |
| X | _y2_ | _z2_ | / | / |
| X | ... | ... | / | / |


| Légende | Description |
|:---:|:---|
| `COMMANDE` | Cellule contenant le nom de la commande, insensible à la casse |
| _Param 1, y1, ..._ | Cellule contenant des paramètres relatifs à la commande appelée |
| _(Param 2)_ | Cellule contenant un argument optionnel |
| X | Cellule interdite à l'écriture |
| / | Cellule non lue par le programme, peut contenir des commentaires |

Les lignes vides entre deux commandes sont autorisées, voire recommandées pour plus de clarté.

Les symboles suivant dans les titres des description des fonctions définissent leur caractère obligatoire :

* (\*) désigne une commande obligatoire ;
* (+) signifie qu'au moins une commande de ce type est obligatoire (CONTOUR / POINTS) ;
* (-) désigne une commande facultative (paramètres de calcul pris alors par défaut).


### Description des fonctions
#### (\*)DI


La commande `DI` précède l’ensemble des instructions que le programme interprète. Les lignes précédant la commande DI ne sont pas interprétées par le programme et sont accessibles à l’utilisateur.


| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `DI` | / | / | / | / |

#### (\*)LOI

La commande `LOI` permet de définir une loi déformation-contrainte.

Une loi est définie par une série de points (déformation ; contrainte) saisie dans l’ordre croissant des déformations.

La série doit contenir au moins 2 points et n’est pas limitée supérieurement.

La définition de la loi doit permettre de « passer » par l’origine du repère (0 ; 0).

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | _(mot-clef)_ | / | / |

**Loi polygonale**

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | / | / | / |
| X | _ε1_ | _σ1_ | / | / |
| X | _ε2_ | _σ2_ | / | / |
| X | ... | ... | / | / |
| X | _εn_ | _σn_ | / | / |

* _id\_loi_ : identifiant de loi sous forme d’une chaine de caractères ;
* _(εi ; σi)_ : couples de valeurs définissant la loi.

L’interprétation par le programme est la suivante :

* la présence d’une nouvelle commande en colonne A ou l’absence d’une valeur en colonne B provoque la fin de définition de la série ;
* l’absence de valeur en colonne C est interprétée comme σi=0 ;
* interpolation linéaire entre les points ;
* renvoie σ1 si la déformation calculée est inférieure à ε1 ;
* renvoie σn si la déformation calculée est supérieure à εn.


**Loi EC2-3.2 - Béton - Loi de Sargin**

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | **EC2-3.2** | / | / |
| X | _fck_ | / | / | / |


**Loi EC2-3.3 - Béton - Loi parabole-rectangle**

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | **EC2-3.3** | / | / |
| X | _fck_ | _γc/αcc_ | / | / |


**Loi EC2-3.4 - Béton - Loi bilinéaire**

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | **EC2-3.4** | / | / |
| X | _fck_ | _γc/αcc_ | / | / |


**Loi EC2-3.8 - Aciers passifs - palier incliné**

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | **EC2-3.8** | / | / |
| X | _fyk_ | _γs_ | _Es_ | _k_ |
| X | _εud_ | _εuk_ | / | / |


**Loi BAEL-Béton - Loi parabole-rectangle**

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | **BAEL-beton** | / | / |
| X | _fcj_ | _γb×θ/0.85_ | / | / |


**Loi BAEL-Acier - Loi bilinéaire**

| A | B | C | D | E |
|:---:|:---:|:---:|:---:|:---:|
| `LOI` | _id\_loi_ | **BAEL-acier** | / | / |
| X | _fy_ | _γs_ | _Es_ | _εlim_ |




----
_to be continued ..._















