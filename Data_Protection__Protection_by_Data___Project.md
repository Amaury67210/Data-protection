# Data Protection, Protection by Data – Project (SENSENBRENNER Amaury SDSC / WEISS Nicolas SDSC / DIDON Valère SDSC)

Notre projet consiste à développer un module Python de détection d'intrusion. Le sujet propose deux scénarios différents, nous avons choisi de travailler sur le sujet numéro 1 qui est de détecter les logiciels malveillants lors de l'exécution.

### <u>Observation des données :</u>

Dans un premier temps, nous allons observer les données. On peut voir que le dataframe contient beaucoup de colonnes  (57). On peut observer que toutes les colonnes sont des colonnes numériques (float64 ou int64) hormis les colonnes `'Category'` et `'Class'` qui sont des colonnes objets. Après l'utilisation de `data['Class'].unique()` et de `data['Category'].unique()`, nous pouvons voir que la colonne `'Class'` contient uniquement deux valeurs : `'Benign'` et `'Malware'`. Cela va nous servir à effectuer une classification binaire. Après observation de la colonne `'Category'`, on peut voir que celle-ci contient un grand nombre de valeurs uniques qui correspondent à un type d'attaque spécifique. Cependant, ces objets sont conçus pour que le premier mot de la colonne corresponde à un type d'attaque spécifique puis le deuxième mot, après le tiret, corresponde à un sous-groupe de cette attaque spécifique. C'est grâce à cela, qu'on va mêler à une analyse sémantique, qui va nous permettre de faire une classification multi-classes soit en 4 classes (`'Benign'`, `'Ransomware'`, `'Spyware'`, `'Trojan'`) ou soit en 16 classes (`'Benign'`, `'Ransomware-Ako'`, `'Ransomware-Shade'`, etc...).

### <u>Nettoyage des données :</u>

Pour le nettoyage des données, nous allons voir dans un premier le nombre de valeurs manquantes de l'ensemble des valeurs du jeu de données. Nous allons utiliser la portion de code ci-dessous pour connaître le pourcentage de valeurs manquantes dans chaque colonne du Dataframe.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-24 à 01.43.22.png)

Comme nous pouvons le voir sur les résultats disponibles dans le Kaggle, il ne manque aucune valeur. Nous n'allons pas opérer de modifications dans ce contexte.

Dans un second temps, nous allons voir les différentes valeurs que peuvent prendre les différentes colonnes de notre Dataframe. La portion de code `len(data[element].unique()) < 2` nous permet d'obtenir les colonnes ainsi que la valeur unique de cette colonne (voir notebook pour plus de détails). On peut voir que 3 colonnes possèdent une unique valeur pour toutes les lignes du Dataframe, on peut donc décider de supprimer ces colonnes car elles ne vont nous apporter aucune information lors d'une analyse plus poussée du Dataframe.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-24 à 01.49.01.png)

Par la suite, pour supprimer un certain nombre de colonnes moins utiles que d'autres, nous avons utilisé la méthode LOFO (Leave One Feature Out). Cette méthode calcule l'importance des features basé sur une métrique, ici nous avons fait le choix d'utiliser la métrique AUC-ROC. Le principe consiste à calculer la performance d'un modèle (ici celui par défaut est le LightGBM) en ayant toutes les features puis de calculer sa performance mais sans une feature qui est enlevée le temps de ce calcul afin d'obtenir l'importance de cette dernière. Pour mettre en place cela, nous avons utilisé la librairie lofo-importance, en important les modules LOFOImportance et Dataset. En effet, il a fallu tout d'abord créer un dataset contenant le dataframe entier (contenant l'ensemble des features et la classe à détecter), le nom de la classe à détecter et l'ensemble des noms des autres features. Nous passons ensuite en paramètres ce dataset, une stratégie de division pour la validation croisée et la métrique sur laquelle nous basons le calcul de l'importance des features dans la fonction LOFOImportance. Cette fonction retourne alors un dataframe contenant beaucoup de valeurs utiles concernant l'importance calculée pour chacune des features (une ligne équivaut à une feature). Ce qui nous a intéressé ici a été la valeur "importance_mean", et nous avons décidé de ne garder que les features ayant cette valeur supérieure à 0.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 20.54.19.png)

### <u>Segmentation du jeu de données en 2 et 4 classes :</u>

#### Classification binaire

Dans un premier temps, nous avons segmenté notre jeu de données de façon binaire : les données saines et les données malsaines. La colonne `'Class'` nous permet déjà d'avoir cette segmentation des données pour chaque ligne du dataframe avec les attributs `'Benign'` et `'Malware'`. On peut voir que le jeu de données est équilibré car il contient autant de données saines (29 298) que des données malsaines (29 298).

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.03.43.png)

Dans ce cas là, on utilise la colonne `'Category'` car pour une classification binaire, il n'y a pas de différences.

#### Classification multi-classes

Dans un second temps, nous avons un peu manipulé notre jeu de données afin que la colonne `'Class'` possède 4 valeurs différentes au lieu de 2 initialement. Nous avons donc effectué une analyse sémantique de la colonne `'Category'`. En effet, celle-ci possède le nom complet de la catégorie d'attaque de la donnée. Grâce à la description des données dans le sujet, nous pouvons voir que notre jeu de données va se décomposer en 4 classes : `'Benign'`, `'Ransoware',` `'Spyware'` et `'Trojan'`. Nous allons donc utiliser le code ci-dessous (exemple pour données Trojan) afin de regrouper dans chaque catégorie les données associées pour constituer un dataframe. Dans ce dataframe, nous allons modifier la valeur de la colonne `'Class'` par une valeur plus spécifique qui définit la catégorie (ici `'Malware_Trojan'` pour les données Trojan). Et pour finir, nous allons regrouper tous les dataframes pour en former plus qu'un avec les modifications apportées à la colonne `'Class'` afin que les modèles de classification puissent faire de la classification multi-classe.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.16.35.png)

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.16.54.png)

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.17.10.png)

### <u>Matrice de corrélation :</u>

Avant d'effectuer nos premiers tests de classification, nous avons regardé la matrice de corrélation pour les données saines et les données malsaines.

#### Données 'Benign'

![](/Users/amorce/Downloads/benign_corr (2).png)

#### Données 'Fraud'

![](/Users/amorce/Downloads/fraud_corr (1).png)



### <u>Classification :</u>

Dans un premier temps, il nous est demandé de faire une comparaison entre les modèles suivants : KNN, CART, Random Forrest, XGBoost, SVM et MLP. Nous allons tout d'abord segmenter nos modèles en deux groupes : ceux qui prennent en charge la classification multi-classes et ceux qui font de la classification binaire.

Pour le rendu intermédiaire, il nous a été demandé d'effectuer nos tests sur un jeu de données avec 10^5 entrées. Le jeu de données que nous utilisons pour effectuer nos entrainements ainsi que nos tests possèdent un nombre d'entrées de 58596.

Pour évaluer la consommation des ressources nécessaires, nous avons décidé d'utiliser un environnement disponible sur le cloud qui est "Kaggle" afin d'avoir les mêmes résultats sur chacun de nos ordinateurs si on réeffectue des tests. En effet, nous ne possédons pas tous le même ordinateur au sein du groupe donc du matériel différent (CPU, RAM) ce qui ne nous permet pas d'effectuer ces tâches sur nos propres ordinateurs car nous obtiendrons des résultats différents.

Le découpage de notre jeu de données pour le jeu d'entrainement et le jeu de test se fait de la manière suivante : 2/3 des données pour le jeu d'entrainement et 1/3 des données pour le jeu de test. Le découpage est le même pour le dataframe qui contient une classification multi-classes.

Pour chaque test effectué sur nos algorithmes, nous avons les informations suivantes : temps d'entrainement du modèle, temps de prédiction du modèle, prédiction, probabilité de chaque prédiction, matrice de confusion, classification du modèle (precision, recall, f1-score), "balanced accuracy report", "Matthews Correlation Coefficient", "True negative rate" et "True positive rate". Pour voir en détail chaque résultat obtenu pour un modèle précis, veuillez regarder le notebook ci-joint. Ci-dessous, vous pouvez observer la fonction que nous avons utilisé pour chaque algorithme de classification. La fonction est légèrement différente pour la classification multi-classes.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.50.19.png)

<u>Voici un exemple de résultat obtenu pour un algorithme de classification sur les jeux de test du dataframe binaire :</u>

<img src="/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.53.49.png" style="zoom:50%;" />

<img src="/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.54.03.png" style="zoom:50%;" />

<u>Voici un exemple de résultat obtenu pour un algorithme de classification sur les jeux de test du dataframe multi-classes :</u>

<img src="/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.56.21.png" style="zoom:50%;" />

<img src="/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 21.56.33.png" style="zoom:50%;" />



Par la suite, nous avons entrainé nos modèles sur les mêmes jeux de données et effectué des tests sur le même jeu de test afin de pourvoir effectuer une meilleure comparaison. Voici les résultats obtenus pour la classification binaire des jeux de test :

- CART : 99,96%
- XGBoost : 99,98%
- Random Forest : 100%
- MLP : 99,63%
- KNN : 99,91%
- SVM : 99,78%

Les résultats pour la classification multi-classes sont moins bons que ceux de la classification binaire. En effet, c'est plus compliqué de prédire une variable sur quatre classes qu'une variable sur deux classes. Voici les résultats obtenus pour la classification multi-classes sur nos algorithmes de classification avec le même jeu de test :

- CART : 84,19%
- XGBoost : 86,90%
- Random Forest : 87,12%
- MLP : 69,07%
- KNN : 81,04%
- SVM : 71,80%

### <u>Temps de fit :</u>

Nous avons donc comparé les temps d'entraînement des modèles obtenus sur le même jeu de données :

<u>Classification binaire :</u>

<img src="/Users/amorce/Desktop/Capture d’écran 2022-10-14 à 16.09.39.png" style="zoom:50%;" />

<img src="/Users/amorce/Desktop/Capture d’écran 2022-10-14 à 16.15.45.png" style="zoom:50%;" />

On peut apercevoir que certains modèles ont un temps d'entrainements beaucoup moins long que d'autres modèles. Par exemple, les algorithmes KNN et CART s'entrainent très vite sur le jeu de données proposé alors que d'autres algorithmes comme MLP ou Random Forest sont beaucoup plus lent (environ 6 à 8 fois plus de temps). Puis, nous avons l'algorithme SVM qui est beaucoup plus lent que tous les algorithmes cités précédemment, il est environ 400 fois plus lent que le reste des algorithmes.

<u>Classification binaire et multi-classes :</u>

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 23.41.31.png)

En comparant les modèles multi-classes et binaire, on peut voir que de manière générale le temps d'entraînement sur les données est moins long avec une classification binaire qu'une classification multi-classes (sauf pour KNN et MLP).

### <u>Temps de prédiction :</u>

<u>Classification binaire :</u>

Nous avons aussi, par la suite, comparé les temps de prédiction de chaque modèle sur le même jeu de test :

<img src="/Users/amorce/Desktop/Capture d’écran 2022-10-14 à 16.18.54.png" style="zoom:50%;" />

On peut apercevoir que les modèles ont un temps de prédiction très rapide sauf un qui est environ 12 fois plus lent que tout les autres, cet algorithme est KNN.

<img src="/Users/amorce/Desktop/Capture d’écran 2022-10-14 à 16.44.03.png" style="zoom:50%;" />

Lorsqu'on enlève l'algorithme KNN afin de comparer nos temps de prédiction sur chaque modèle, on peut voir que les modèles les plus rapides en temps de prédiction sont XGBoost, MLP et CART. L'algorithme SVM est environ 4 fois plus lent que les modèles cités précédemment et l'algorithme Random Forest est environ 8 fois plus lent que ces mêmes modèles.

<u>Classification multi-classes et binaire :</u>

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 23.47.56.png)

Concernant la comparaison du temps de prédiction vis à vis du multi-classes et du binaire. En règle général, le temps de prédiction est plus long pour la classification multi-classes que pour la classification binaire (sauf pour KNN). On peut aussi ajouter qu'on observe une grosse différence de temps de prédiction entre le SVM binaire et le SVM multi-classes (temps de prédiction doublé) alors que sur les autres algorithmes, le rapport est moindre.



### <u>Antiscore :</u>

<u>Classification binaire :</u>

Puis pour finir, nous avons évalué l'antiscore de chaque modèle :

<img src="/Users/amorce/Desktop/Capture d’écran 2022-10-14 à 16.20.55.png" style="zoom:50%;" />

On peut voir que l'antiscore se situe entre 0.0000 et 0.0010 pour la plupart des modèles hormis l'algorithme MLP qui possède un antiscore supérieur à 0.0035.

<u>Classification multi-classes :</u>

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 23.51.14.png)

Concernant l'antiscore des modèles multi-classes, il est largement plus élevé que la classification binaire. Les modèles multi-classes apprennent moins bien que les modèles binaire. Nous pensons que cela est aussi dû aux faibles nombres du dataframe. Il faudrait plus de données pour que nos modèles de classifications apprennent mieux.

### <u>Comparaison des paramètres des modèles :</u>

Dans cette section, nous sommes à la recherche des meilleurs paramètres afin de développer notre modèle de classification. Chaque modèle de classification est testé avec chaque variante de paramètres qui est connu. Nous avons effectué ces tests de paramètres à la fois sur la classification binaire et sur la classification multi-classes. Il se peut fortement que les meilleurs paramètres ne soient pas les mêmes pour les deux classifications.

<u>Exemple du code utilisé pour faire varier les paramètres pour CART :</u>

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 23.58.35.png)

![](/Users/amorce/Desktop/Capture d’écran 2022-11-28 à 23.59.28.png)

Nous vous conseillons d'aller voir le Notebook Jupyter disponible ci-joint afin de voir les différents résultats obtenus. Les résultats prendraient trop de place dans ce rapport si nous les plaçons ici. Nous avons fait la même manipulation pour tous les algorithmes de classification que nous avons utilisé ci-dessus.

### <u>Analyse du meilleur modèle :</u>

Après avoir obtenus des centaines et des centaines de mesures différentes, nous les avons enregistrées afin qu'elles soient plus facilement accessibles. En effet, il ne sera pas obligatoire de recharger tout le notebook afin d'obtenir les derniers résultats car cela prend trop de temps.

Après analyse de chaque modèle, nous avons décidé de mettre en valeur le modèle ayant eu le meilleur antiscore, le modèle ayant eu le meilleur temps de fit et le modèle ayant eu le meilleur temps de prédiction.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 00.05.26.png)

Nous pouvons voir ci-dessous, les modèles ayant obtenus les meilleurs résultats dans leur domaine.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 17.56.08.png)

Nous allons par la suite (voir photo ci-dessous), mettre tous les meilleurs modèles de chaque algorithme de classification (binaire et multi-classes) ainsi que leurs paramètres dans un dataframe.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 17.59.07.png)

Nous allons observer la corrélation entre les différents lignes du dataframe afin de voir si nous pouvons en dégager quelques informations supplémentaires ainsi qu'une conclusion.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 18.01.17.png)

<u>Exemple de résultat (beaucoup de résultats disponible sur le Notebook) :</u>

![](/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 18.01.25.png)

### <u>Tensorflow :</u>

A l'aide de Tensorflow, nous allons essayer de trouver les meilleurs paramètres pour un algorithme de classification spécifique afin que cela soit plus rapide que d'analyser et de récolter toutes les possibilités des modèles de classification.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 00.07.17.png)

<u>Voici donc les meilleurs paramètres (en théorie) utilisables pour le modèle de classification MLP :</u>

<img src="/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 13.36.50.png" style="zoom:50%;" />

Nous allons donc utiliser les données des différents paramètres dans notre fonction de classification afin de voir si le modèle est le meilleur en terme de performance.

![](/Users/amorce/Desktop/Capture d’écran 2022-11-29 à 13.39.19.png)

Comme vous pouvez le voir ci-dessus, nous avons testé les performances du modèle MLP avec les paramètres que nous avons obtenus grâce à la fonction un peu plus haute. On peut en conclure que la combinaison des meilleurs paramètres n'est pas forcément le bon choix pour obtenir le modèle le plus optimisé possible. En effet, d'après les résultats obtenus ci-dessus, on peut très vite distinguer que nous avons des modèles qui sont meilleurs dans les 3 domaines (Temps de fit, Temps de prédiction et antiscore) que le modèle présent ci-dessus. L'expérience n'est donc pas un succès.

### <u>Conclusion :</u>

En conclusion, nous avons récolté le plus de métriques possibles sur tout les algorithmes de classification (binaire et multi-classes) en faisant varier leurs paramètres  afin de constituer une base de données. Nous pouvons déjà dire que les modèles de classification sont toujours plus efficaces en terme de performance pour de la classification binaire que pour de la classification multi-classes. À l'aide de celle-ci, nous avons pu extraire le meilleur modèle en terme de temps de prédiction, le meilleur modèle en terme de temps de fit et le meilleur modèle en terme d'antiscore. Mais nous pouvons observer que même si ils ont obtenus les meilleurs résultats dans leurs domaines, les autres résultats ne sont pas très compétitifs. Nous avons alors décidé de trouver les meilleurs paramètres avec l'aide d'une fonction et de la librairie python Tensorflow, les meilleurs paramètres pour un modèle donné. Après l'effectuation de nos tests, on peut constater que cette méthode n'est pas efficace.