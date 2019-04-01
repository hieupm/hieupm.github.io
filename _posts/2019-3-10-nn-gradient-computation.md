---
published: true
title: "Descente de gradient, Graphes de calcul"
excerpt: "NN Basics"
toc: true
toc_sticky: true
toc_label: "NN Basics"
toc_icon: "microchip"
author_profile: false
comments: true
header:
  overlay_image: "assets/images/covers/cover1.jpg"
  teaser: "assets/images/covers/cover1.jpg"
categories: [ml, graph, algebre]
---

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Principes

Nous verrons ici comment utiliser la descente de gradient pour apprendre les paramètres w et b sur l'ensemble de données d'apprentissage.

Fonction de coût :
$$J(w, b)=\frac{1}{m}\sum_{1}^{m}\mathfrak{L}(\hat y^{(i)},y^{(i)})$$

## Minimisation de J

On veut trouver les paramètres w et b qui **minimisent** la fonction J(w, b). Si il était possible de représenter pour toutes les valeurs de l'espace des couples (w, b), on verrait facilement pour quelles valeurs la fonction de perte est minimale. Or, trouver un seul point de la valeur de la fonction de perte necessite du **temps de calcul** (et augmente avec la complexité de notre algorithme).

## La descente de gradient
L'algorithme de **descente de gradient** est largement utilisé dans le machine learning pour éviter d'évaluer les performances pour un grand nombre de paramètres. Cet algorithme permet de faire évoluer nos paramètres (w, b) dans le bon sens (le sens qui induit une perte plus petite).

On voit sur l'image ci-dessous le type d'évolution que l'on souhaite faire suivre à nos paramètres, on veut arriver sur les points les plus bas du paysage, qui correspondent aux valeurs basses de la fonction de coût. Les paramètres theta sont nos paramètres w et b.

<img src="https://cdn-images-1.medium.com/max/1600/1*f9a162GhpMbiTVTAua_lLQ.png" alt="" class="center">

## Utilisation des graphes de calcul
Pour la descente de gradient, on a vu que l'on a besoin de calculer les dérivées selon chacune des variables d'entrée. Dans un réseau de neurone, on peut voir le problème comme un **graphe de calcul** et calculer les dérivées en chaine.

# La descente de gradient

Les étapes de l'algorithme :
- On initialise nos paramètres (w, b) à des valeurs aléatoires.
- A chaque étape, les paramètres sont mis à jour dans la direction qui "descend" le plus fort.
- Au bout d'un nombre fini d'itérations, le minimum global est atteint si la fonction est convexe. Les performances de notre algorithme sont optimales.

On écrit ainsi la mise à jour des paramètres :

$$w = w = \alpha . \frac{\partial J(w, b)}{\partial w}$$

$$w = w = \alpha . \frac{\partial J(w, b)}{\partial b}$$

Le paramètre **alpha** est le **pas d'apprentissage** ou **learning rate**, permet d'ajuster la taille des changements que l'on effectue. Nous verrons que sa valeur est importante selon les cas, on souhaite aller plus vite, ou bien affiner nos paramètres par de toutes petites variations.

# Le graphe de calcul

Les étapes de calcul dans un réseau de neurones sont organisés en deux étapes :
- La **propagation vers l'avant** ou **forward propagation** : On calcule la sortie du réseau.
- La **back-propagation** où l'on se sert des résultats de la première étape pour calculer les gradients et dérivées, pour mettre à jour les paramètres.

## Principe (forward/backward)

Le graphe de calcul ou **computation graph** est une manière de représenter des opérations successives utilisant plusieurs variables. Ces graphes sont particulièrement utiles quand on souhaite optimiser la valeur finale.

En effet, puisqu'on va s'intéresser aux dérivées par rapport à nos différents paramètres, on va suivre les deux mêmes étapes évoquées plus haut.
- On effectue d'abord les opérations sur les variables pour arriver à la valeur de sortie du graphe (forward).
- On calcule ensuite les dérivées partielles pour mettre à jour nos variables dans le sens souhaité (backward).

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/nn1/computation-graph.png" alt="" class="center">

## Le calcul de dérivées dans le graphe

Une fois que l'on a atteint la valeur finale de notre graphe de calcul, on souhaite connaitre l'importance de chacune des variables. On peut alos utiliser les valeurs du graphe pour estimer les dérivées.

Ici on connait déjà les valeurs des coefficient, donc la dérivée semble évidente. Mais il est important de remarquer que cette valeur se retrouve en prenant le ratio de la variation de J et la variation du paramètre.

On peut ensuite remonter la **chaine des dérivées** pour trouver l'impact des variables de départ sur les variables intermédiaires puis sur la valeur de sortie.

Habituellement il y a une variable finale à la sortie de notre algorithme, et on va essayer de trouver l'impact individuel de chacune des variables. $$\frac{dVariableFinale}{dVar}$$

## Dérivées en chaine

Pour remonter à l'influence d'une variable sur la sortie, on va utiliser les dérivées successives. Dans le graphe en exemple nous avons les variables a, b, c, u, v et J la variable finale.

- Pour calculer l'influence de la variable v, on augmente la valeur de v et on regarde l'impact sur la variable J. Si on augmente v de 0.001, J augmente de 0.003. Par le ratio des variations on a :

$$\frac{dJ}{dv} = \frac{0.003}{0.001} = 3$$

- On remonte ensuite vers les variables d'entrée, pour **a** qui est plus simple :

$$\frac{dJ}{da} = \frac{dJ}{dv} . \frac{dv}{da} = \frac{0.003}{0.001} . \frac{0.003}{0.001} = 3 . 1 = 3$$

- Ainsi de suite, on peut décomposer chaune des étapes pour remonter à l'influence des variables d'entrée.

On a vu dans cet article le principe de la descente de gradient, et de dérivées en chaine. Ce sont des éléments très important pour les réseaux de neurones. On trouve quantité d'articles sur le sujet auprès des sources spécialisées, et cela constitue les bases pour l'enseignement du deep learning.

# Descente de gradient pour la regression logistique

## Hypothèses de départ

On reprend le cadre simple de la regression logistique avec les éléments suivants :
- $$z = w^\intercal \mathbf{x} + b$$ : L'équation de regression.
- $$ \hat y = a = \sigma(z)$$ : La prédiction par la fonction d'activation.
- $$\mathfrak{L}(a, y) = -(y\log a+(1-y)\log(1-a))$$ : La fonction de perte pour une prédiction a.

Ainsi, pour deux variables d'entrée $$x_{1}$$ et $$x_{2}$$, avec leurs poids associés $$w_{1}$$ et $$w_{2}$$ et le terme de biais b on a :
- $$z(x_{1}, x_{2}) = w_{1}x_{1}+w_{2}x_{2}+b$$ : L'équation de classification
- $$\hat y = a = \sigma (z)$$ : La prédiction a.
- $$\mathfrak{L}(a, y)$$ : La fonction de perte se calcule finalement sur la prédiction a, et la vraie valeur y.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/nn1/applied-schema1.png" alt="" class="center" width="700">

Le but de la descente de gradient est d'évaluer la fonction de perte L afin de mettre à jour les paramètres (dans la direction opposée au gradient) afin de minimiser L.

## Les dérivées successives

Comme vu dans le premier article sur la descente de gradient, on cherche à remonter successivement dans les dérivées. 

### Influence de a
La première dérivée qui nous interesse est celle de a : $$\frac{d\mathfrak{L}(a, y)}{da}$$. C'est **l'influence de a** sur l'évolution de la fonction de perte. Si on reprend l'expression de L(a, y), par la dérivation des fonctions Log on trouve : 

$$\frac{d\mathfrak{L}(a, y)}{da} = -\frac{y}{a} + \frac{1-y}{1-a}$$

### Influence de z
On remonte d'une étape dans le diagramme et on s'interesse à **l'influence de la variable Z** : 

$$\frac{d\mathfrak{L}(a, y)}{dz} = \frac{d\mathfrak{L}(a, y)}{da} . \frac{da}{dz}$$

Il s'agit d'un produit utilisant le terme trouvé à la première étape, il suffit de s'interesser à la fonction qui transforme Z pour arriver à A, c'est la fonction sigmoïde. Sa dérivée est égale à $$\frac{da}{dz} = a.(1-a)$$. Par produit on obtient : 

$$dz = \frac{d\mathfrak{L}(a, y)}{dz} = a - y$$

### Influence des variables d'entrée
L'étape finale du calcul est d'arrivée à **l'influence de chaque variable d'entrée** sur la fonction de perte grâce à l'expression précédente. Pour le paramètre $$x_{1}$$ on va modifier le poids $$w_{1}$$, on calcule donc : 

$$\frac{\partial L}{\partial w_{1}} = x_{1} . dz$$

L'expression est similaire pour le poids $$w_{2}$$. Pour le biais on a $$db = dz$$. Ce sont les quantités que l'on utilisera pour notre mise à jour des poids dans la descente de gradient :

$$w_{1} := w_{1} - \alpha dw_{1}$$

On a vu dans ce paragraphe comment effectuer la mise à jour des poids pour un seul exemple d'entrainement de notre algorithme. 

Dans la suite nous verrons comment appliquer le même raisonnement à **plusieurs données d'entrainement** : quand on dispose de nombreux couples $$(x_{1}, x_{2})$$.

# Descente de gradient sur m données d'entrainement

Dans les raisonnements précédents, on minimise la fonction de **perte L**, qui est l'évaluation de la **performance sur un couple prédiction / vraie valeur**. 

## Fonction de coût

On va maintenant s'intéresser à la fonction de **coût J** : C'est la moyenne des fonctions de perte, sur **l'ensemble des données d'entrainement**. C'est cette quantité que l'on va chercher à minimiser sur l'ensemble de nos données. 

$$J(w, b)=\frac{1}{m}\sum_{1}^{m}\mathfrak{L}(a^{(i)},y)$$

Sa dérivée par rapport au poids $$w_{1}$$ s'exprime ainsi :

$$\frac{\partial J(w, b)}{\partial w_{1}} = \frac{1}{m}\sum_{1}^{m}\frac{\partial L(a^{(i)}, y^{(i)})}{\partial w_{i}}$$

Comme on a déjà vu l'étape de calcul du terme à l'intérieur de la somme, il ne reste plus qu'à **moyenner** ce terme sur l'ensemble des données d'apprentissage pour effectuer la **mise à jour**.

## Pseudo-algorithme

Avec les éléments vus jusqu'à présent, on peut commencer à établir une version simple de l'algorithme de descente de gradient

```python
J=0, dw1=0, dw2=0, db=0     # Initialisation des variables
for i=1 to m:               # Iteration sur train-set
zi = wT.xi + b          # Calcul de Z(x1, x2)
ai = sigma(zi)          # Prediction
J += L(ai, y)           # Ajout de la fonction de perte
dzi = ai - yi           # Influence de Z
dw1 += x1i * dzi        # Influence du poids w1 (étape i)
dw2 += x2i * dzi
db += dzi
J/=m, dw1/=m, dw2/=m, db/=m  # Moyenne sur le train-set

# Mise à jour des paramètres 
w1 = w1 - alpha * dw1
w2 = w2 - alpha * dw2
b = b - alpha * db
```

- On utilise les quantités J, dw1, dw2 et db comme des **accumulateurs**, et on effectue une moyenne sur le nombre de données d'entrainement. 
- Tout ce pseudo algorithme constitue une seule étape de la descente de gradient, c'est à dire le **chemin entre deux croix** sur le graphique ci-dessous. Il faut donc effectuer plusieurs itérations afin de descendre au plus bas pour notre fonction de perte J.

<img src="https://cdn-images-1.medium.com/max/1600/1*f9a162GhpMbiTVTAua_lLQ.png" alt="" class="center">

## Optimisation

Le pseudo algorithme vu ci-dessus présente le défaut d'utiliser **deux boucles for** :
- Une boucle pour itérer sur l'ensemble du train set.
- Une boucle pour mettre à jour chacun des paramètres après le calcul de *dzi* (ici nous en avons uniquement deux donc cela n'apparait pas clairement).

En deep learning, avoir des boucles for est une entorse à l'efficacité des algorithmes. Éviter l'usage des boucles for permet de scaler plus facilement à des datasets plus gros.

Pour se débarasser de ces boucles, on utilisera des techniques de **vectorisation**, indispensables de nos jour vu les quantités de données utilisées.

# Sources
- <a href="https://towardsdatascience.com/its-only-natural-an-excessively-deep-dive-into-natural-gradient-optimization-75d464b89dbb" target="_blank">Nombreux articles sur la descente de gradient</a> (TowardsDataScience)
- <a href="https://www.coursera.org/learn/neural-networks-deep-learning/home/welcome" target="_blank">Deep Learning course</a> (Coursera - Andrew Ng)
