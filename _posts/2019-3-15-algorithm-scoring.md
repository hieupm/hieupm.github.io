---
published: true
title: "Scoring des algorithmes"
excerpt: "Datasets, scores, courbes, taux"
toc: true
toc_sticky: true
toc_label: "Algorithm Metrics"
toc_icon: "ruler-combined"
comments: true
author_profile: false
header:
  overlay_image: "assets/images/covers/cover4.jpg"
  teaser: "assets/images/covers/cover4.jpg"
categories: [metrics, scoring]
---

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

Quand on travaille avec les algorithmes, on finit toujours par devoir évaluer leur performances, notamment pour :
- Evaluer la performance de notre modèle avec differents hyper-paramètres (paramètres de l'algorithme).
- Connaitre les performances du modèle, et comparer le résultat avec d'autres personnes, comme dans les challenge Kaggle.

Dans cet article nous verrons :
1. Comment **segmenter les données** pour calculer les scores.
2. La matrice de confusion pour la **classification binaire**.
3. Mesures de performance pour les problèmes de **classification**.
4. Mesures de performance pour les algorithmes de **régression**.

# Segmentation des données pour l'évaluation

## Dilemme biais-variance

Dans la segmentation du dataset en général, il est important de garder à l'esprit la recherche d'équilibre biais-variance pour ne pas tomber dans le sous/sur-apprentissage.

C'est un problème bien connu des data scientists, il s'agit de trouver un compromis pour **minimiser deux sources d'erreurs** liées entre elles :

- **Le biais** : Il est souvent causé par un manque de relations pertinentes dans les données, on parle de **sous-apprentissage**. L'algorithme ne dispose pas d'assez de données pour établir les liens pertinents.
- **La variance** : C'est lorsque l'algorithme se colle trop aux données d'apprentissage, il modélise alors des variations non significatives, c'est le **sur-apprentissage**. Lorsque l'algorithme est ensuite confronté aux données de test qu'il n'a jamais vu, les prédictions seront moins précises.

## Split : train, validation, test
C'est la première étape quand on commence à travailler avec un ensemble de données. Cela consiste à séparer nos données en deux sous-ensembles :
- Un set d'entrainement, le **train set**. Qui sera composé de la majorité des données, servant à entrainer notre modèle.
- Un set de validation, le **validation set**. Qui est utilisé dans le cycle de développement, permet de fournir une évaluation non-biaisée du modèle lors de la recherche des hyper-paramètres.
- Un set de test, le **test set**. Il s'agit d'une partie du dataset que l'on va cacher à l'algorithme et représentera 20% du volume de données disponible. C'est sur cet ensemble que l'algorithme sera évalué à la toute fin.

Le train/validation/test split est une technique très répandue, qui fonctionne bien lorsque l'on dispose d'une **grande quantité de données**, car il faut "sacrifier" un pourcentage des données, pour l'évaluation du modèle.

<div align="center">
    <img src="https://qph.fs.quoracdn.net/main-qimg-e4755860eefa095dcab79659e356cf56" alt="Evaluation Flow-chart" vspace="10">
</div>

## Validation croisée (K-fold)

La validation croisée est une extension du train/test split. On parle aussi de **cross-validation** ou **cv** dans les paramètres des algorithmes. Elle est utilisée quand la **quantité de données est limitée**, ou bien quand on dispose des ressources/temps nécessaires pour effectuer **plusieurs passages** sur les données.

Le principe de la validation croisée est de faire une boucle pour que tout le dataset d'entrainement ait servi d'ensemble de validation. On parle de **folds** et on en utilise généralement 5 ou 7. C'est à dire que le dataset d'entrainement sera séparé en 7 parties et qu'on effectue **7 passages** (entrainement et évaluation sur validation set).

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/machine-learning/cross-validation.png){: .align-center}

Les **métriques** d'évaluation calculées sur chacun des sets de validation, sont finalement **moyennées** pour présenter une métrique de validation.

La cross-validation est une **bonne pratique** qu'il faut prendre comme habitude. Elle permet d'évaluer de manière plus précise le modèle, et de mieux se protéger du sur-apprentissage.

Dans le K-fold, on peut donc faire varier K de 2 à N :
- Pour *K=2*, on sépare le dataset d'entrainement en deux. C'est la séparation la plus grossière possible.
- Pour *K=N*, avec N le nombre de données d'entrainement. On cache uniquement une valeur à chaque passage, on parle de *Leave-one-out*. C'est un traitement généralement très couteux en temps de calcul.
- Les valeurs de K utilisées sont généralement **5, 7, 10**, qui sont des valeurs acceptables, mais il n'y a pas de règle formelle.

> Plus on augmente K, plus la difference entre les sets entrainement/validation est grande, on se rapproche d'un set d'entrainement complet. 

En se rapprochant du set d'entrainement complet, le **biais introduit est plus faible**, car on lui cache moins de données, mais sa **variance** sera plus importante. 
- Lorsque toutes les données ne sont pas visibles par l'algorithme, il va devoir effectuer des approximations, c'est le **biais**.
- Lorsque l'algorithme décrit une fonction trop complexe qui est trop proche des données d'apprentissage, sa **variance** est élevée, car il ne saura pas bien généraliser à d'autres données.

## Variantes de la validation croisée

- On a déjà vu plus haut la **Leave-one-out cross-val**.
- La **stratified k-fold** cross validation, est une version pour les jeux de données **déséquilibrés**. On va introduire une contrainte d'équilibre des classes soit conservé (de 0 et de 1 par exemple).
- La validation croisée **répétée** : On effectue plusieurs fois le process de KFold, avec un tirage aléatoire lors de la constitution des folds à chaque itération.

# Mesures pour la classification binaire

Dans la partie précédente, on a évoqué la mesure de scores sur les différents dataset. Dans cette partie, on verra que la matrice de confusion nous permet d'obtenir diverses mesures de performance pour la classification binaire.

## La matrice de confusion

Dans cette partie, on traite un problème de **classification binaire** (prédiction de 0 ou 1). La matrice de confusion va présenter les résultats en confrontant la dimension **Vérité (Actual)** et **Prédiction (Predicted)**.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/machine-learning/confusion-matrix.png){: .align-center}

Prenons l'exemple de la reconnaissance de chat sur une image : 0 ce n'est pas un chat, 1 c'est un chat.

La diagonale des réussites en vert :
- Les **Vrai positifs (VP)** : L'algorithme a reconnu un chat sur l'image, et c'était réellement un chat.
- Les **Vrai négatifs (VN)** : L'algorithme n'a pas trouvé de chat sur l'image, et il n'y en avait pas.

La diagonale des erreurs en rouge :
- Les **Faux positifs (FP)** : L'algorithme a reconnu un chat, alors que c'était un chien.
- Les **Faux négatifs (FN)** : L'algorithme n'a pas trouvé de chat dans l'image, alors qu'il y en avait un.

Selon les cas d'usages, on va chercher à minimiser une erreur ou l'autre :
- Si notre algorithme classifie des mails comme spam ou non, on peut souhaiter minimiser les **faux positifs**, c'est à dire ne pas envoyer un mail important dans les spams.
- Si notre algorithme analyse des patients pour détecter des cas de cancer avant étude par un médecin, on souhaite détecter tout les cas de cancer, on ne veut pas avoir de **faux négatifs**. En contre-partie on augmentera le taux de faux positifs, qui seront alors écartés lors d'étude par un médecin.

## Accuracy / Justesse

Une mesure très courante que l'on tire de la matrice de confusion est **la justesse / l'accuracy**. C'est la rapport entre les prédictions justes, sur le nombre total de prédictions.

Pour que cette mesure soit pertinente, il faut que le jeu de données soit globalement **équilibré**.

L'accuracy ne convient pas du tout pour un dataset deséquilibré, en effet, si 5% des valeurs sont de la classe 1 et qu'on prédit uniformément la classe 0 pour tout les exemples, on aura une accuracy de 95%, ce qui n'est pas représentatif.

> Dans quelle proportion des cas l'algorithme fait une bonne prédiction.

$$Justesse =  \frac{Succès}{Total} = \frac{VP + VF}{VP + VF + FP + FN}$$

## Précision

La précision nous informe sur la performance pour une classe donnée : Sur toutes les images où l'algorithme a trouvé un chat, quelle proportion contenait réellement un chat.

> Pour les prédictions d'une classe donnée, quelle proportion est bien classifiée.

$$Justesse = \frac{VP}{VP + FP}$$

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/machine-learning/confusion-matrix-precision.png){: .align-center}

## Sensibilité (Recall)

> Sur toutes les images de chat, quelle est la proportion d'images où l'algorithme a identifié un chat.

$$Recall =  \frac{Succès(Classe1)}{Total(Classe1)} = \frac{VP}{VP + FN}$$

Le choix entre **précision et sensibilité** se fera sur l'importance attachée aux faux négatifs ou faux positifs.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/machine-learning/confusion-matrix-recall.png){: .align-center}

## F1-Score

On ne veut pas toujours exprimer la précision et la sensibilité : La mesure du F1 score permet d'**embarquer ces deux mesures** dans un même score. On pourrait bien sûr faire une simple moyenne, mais ce serait peu représentatif. Le F1 score donne des résultats pertinents pour des classes déséquilibrées, mais il faudra creuser avec d'autres mesures pour en savoir plus.

> Score prenant en compte la sensibilité et la précision, qui traite bien le déséquilibre de classes.

On utilise donc la moyenne harmonique, qui favorise la plus petite valeur quand les deux valeurs sont éloignées :

$$F1 Score = \frac{2 \times Precision \times Recall}{Precision + Recall}$$

## Area Under Curve (AUC)

Maintenant que l'on a vu quelques exemples autour de la matrice de confusion, passons à une des mesures les plus **largement utilisées** pour l'évaluation de modèles de classification binaire : La mesure AUC.

En détail, il s'agit de l'**aire sous la courbe ROC** (Receiver Operating Characteristic) que l'on mesure. On compare alors notre courbe à celle d'un classifieur aléatoire, qui est la courbe en rouge ci-dessous. Idéalement, on souhaite avoir la courbe la plus haute possible vers le coin superieur gauche, c-à-d avoir un **taux maximal de vrais positifs**.

> L'AUC permet de quantifier à quel point l'algorithme s'éloigne d'un classifieur aléatoire.

![image-center](https://docs.eyesopen.com/toolkits/cookbook/python/_images/roc-theory-small.png){: .align-center}

En pratique, on va chercher à determiner les taux de **vrai et faux positifs** (tpr et fpr) pour établir cette mesure.

## Perte Logarithmique

La perte Logarithmique, ou **Log Loss**, est une fonction très répandue pour évaluer un classifieur multi-classe :

- Pour N exemples d'entrainement, pour M classes différentes.
- $$y_{ij}$$ : Indique si l'échantillon **i** appartient à la classe **j**, c'est la valeur observée, la vérité.
- $$p_{ij}$$ : La probabilité que l'échantillon **i** appartienne à la classe **j**.

$$LogarithmicLoss = -\frac{1}{N}\sum\limits_{i=1}^N\sum\limits_{j=1}^M y_{ij} * \log(p_{ij})$$

Une LogLoss **proche de zéro** indique une bonne précision de l'algorithme. Cette fonction est souvent utilisée comme **fonction objective** (celle que l'on veut minimiser), par exemple dans les réseaux de neurones, et donne des classifieurs plus précis.

# Evaluer les régressions

Pour les algorithmes de régression, les deux principales mesures sont RMSE et MAE.

## Root Mean Square Error

La mesure RMSE est mesurée sur l'ensemble des n estimations. C'est la moyenne du carré des écarts entre la vraie valeur $$y_{j}$$, et la valeur prédite $$\hat y_{j}$$.

$$RMSE = \sqrt{\frac{1}{n}\sum\limits_{j=1}^n (y_{j}-\hat y_{j})^2}$$

## Mean Absolute Error

La **Mean Absolute Error** est la différence absolue entre la valeur observée et la valeur prédite. C'est un score linéaire : chaque différence individuelle est prise en compte avec le même poids.

$$MAE = \frac{1}{n}\sum\limits_{j=1}^n \mid y_{j}-\hat y_{j} \mid$$

## Laquelle choisir ?

Il est facile de voir que la MAE prend la moyenne des écarts, alors que RMSE pénalise plus les grands écarts. Généralement, la RMSE est supérieure ou égale à la MAE.

Bien que la **RMSE** soit plus complexe à calculer, et qu'elle soit biaisée envers les grands écarts, c'est la métrique la plus **largement utilisée** pour les cas de régression. Utilisée comme fonction de perte, on va préférer une **fonction carré**, plus facilement *dérivable*.

## R2 et R2 ajusté

Les mesures de **R2** et **R2 ajusté** sont souvent utilisées pour montrer si les variables choisies expliquent la variabilité de la valeur observée.

$$R^{2} = 1 - \frac{\sum\limits_{j=1}^n (y_{j}-\hat y_{j})^2}{\sum\limits_{j=1}^n (y_{j}-\overline{y}^{2})}$$

La mesure du **R2** correspond au ratio entre deux écarts : l'écart entre les prédictions et la vraie valeur, et l'écart entre la vraie valeur et la valeur moyenne des prédictions.

$$R^{2}_{ajusté} = 1 - \bigg[ \frac{(1-R^2)(n-1)}{n-k-1} \bigg]$$

- Avec $$n$$ le nombre d'observations, $$k$$ le nombre de predicteurs.
- Le $$R^2_{ajusté}$$ sera toujours inférieur au $$R^2$$.

Lorsque l'on cherche à **comparer plusieurs modèles, utilisant un nombre différent de variables**. Le $$R^2$$ ajusté arrive à determiner si les nouvelles variables ont eu un **impact** positif, si elles ne sont pas utiles, le $$R^2_{ajusté}$$ sera plus faible et on saura que nos variables supplémentaires n'auront pas amélioré le modèle.


# Sources
- [TowardsDataScience - Evaluation de modèles](https://towardsdatascience.com/metrics-to-evaluate-your-machine-learning-algorithm-f10ba6e38234)
- [Dezyre - Performance Metrics for ML](https://www.dezyre.com/data-science-in-python-tutorial/performance-metrics-for-machine-learning-algorithm)
- [Machine Learning Mastery - Cross validation](https://machinelearningmastery.com/k-fold-cross-validation/)
- [Confusion Matrix explained](https://medium.com/thalus-ai/performance-metrics-for-classification-problems-in-machine-learning-part-i-b085d432082b)
- [Medium - Choosing the right metrics](https://medium.com/usf-msds/choosing-the-right-metric-for-machine-learning-models-part-1-a99d7d7414e4)
- [Wikipedia - Dilemme biais-variance](https://fr.wikipedia.org/wiki/Dilemme_biais-variance)
