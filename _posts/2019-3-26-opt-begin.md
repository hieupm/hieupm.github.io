---
published: true
title: "Premières étapes de conception"
excerpt: "Itération, Regularisation, Normalisation"
toc: true
toc_sticky: true
toc_label: "First steps"
toc_icon: "microchip"
comments: true
author_profile: false
header:
  overlay_image: "assets/images/covers/cover6.png"
  teaser: "assets/images/covers/cover6.png"
categories: [deep-learning]
---

<script type="text/javascript" async
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Approche de conception

## Démarche itérative

La conception de réseaux de neurones est un processus iteratif. Comme il n'y a pas de valeur prédéfinie pour chaque type de problème, il faut expérimenter et tirer des conclusions. A chaque itération, on peut être amené à repenser :

- Le nombre de couches du réseau
- Le nombre de neurones par couche, leur fonctions d'activation
- Le pas d'apprentissage

## Train-test split

Avant toute chose, il est important de **séparer les données** en ensemble d'entrainement, de validation, et de test.

Il faut également s'assurer que les données utilisées ont des distributions cohérentes. Il faut par exemple entrainer et tester le réseau sur le même type d'images

*Exemple : Si on souhaite utiliser du deep learning dans une application mobile, où les utilisateurs fournissent leur propres images, et que l'on entraine le réseau sur des images parfaitement cadrées et professionnelles. On se doute que les performances dans l'application n'auront rien à voir car les utilisateurs donneront au réseau des imgaes de mauvaises qualité, pas forcément bien cadrées.*

Ces notions sont abordées plus en détail dans [cet article](https://alexpeterbec.github.io/metrics/scoring/algorithm-scoring/#split--train-validation-test).

## Compromis biais-variance

Le biais et la variance sont deux caractéristiques qui dégradent les performances du réseau. On cherche à les minimiser.

Au final, on veut : 
- Que notre algorithme ait une bonne performance sur les données d'entrainement. 
- Que les performances soient stables sur un ensemble de cas d'usages, on veut que le réseau puisse "généraliser" sur de nouvelles données.

Ces notions sont abordées plus en détail dans [cet article](https://alexpeterbec.github.io/metrics/scoring/algorithm-scoring/#dilemme-biais-variance).

## Schematisation de la démarche

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/nn1/nn-iter.png){: .align-center}

Avec les réseaux de neurones, on dispose de plus d'outils pour résoudre les problèmes de biais et de variance qu'avec les algorithmes classiques.

# Régularisation du modèle

Pendant l'apprentissage, le réseau minimise la fonction de coût $$J(w, b)$$, c'est la moyenne sur les données d'entrainement des fonctions de perte pour chaque exemple d'entrainement. L'application de techniques de régularisation a pour but **d'éviter le sur-apprentissage et de réduire la variance**. 

## Régularisation L2

Les régularisations L2 consiste à ajouter un terme de pénalisation de la fonction de coût :

$$J(w, b) = \frac{1}{m} \sum_{1}^{m} L(\hat y^{(i)},y^{(i)}) + \frac{\lambda}{2m} \sum_{l=1}^{L} \|w^{[l]}\|^{2}_{F}$$

Avec $$\|w^{[L]}\|^{2}_{F}$$ la norme de Frobenius de la matrice des poids de la couche L, et $$\lambda$$ le paramètre de régularisation (un nouvel hyperparamètre).

**La norme de Frobenius**

C'est la somme des valeurs au carré de la matrice des poids, qui est de dimension $$(n^{[l]}, n^{[l-1]})$$.

$$\|w^{[l]}\|^{2}_{F} = \sum_{i=1}^{n^{[l-1]}} \sum_{j=1}^{n^{[l]}} (w_{ij}^{[l]})^{2}$$.

**Pourquoi uniquement effectuer la régularisation sur le vecteur w des poids ?**

On pourrait inclure un terme prenant en compte la somme des éléments du vecteur de biais, mais cela inclue peu d'informations. La matrice des poids contient bien **plus d'informations** car sa dimension est pls grande.

## Influence sur la back-propagation

Avec le nouveau terme de régularisation, on modifie l'étape de mise à jour des paramètres :

$$w^{[l]} = w^{[l]} - \alpha [ dw_{backprop} + \frac{\lambda}{m} w^{[l]}]$$

En développant cette expression, on se rend dompte que le terme de régularisation se soustrait à la matrice des poids, et supprime ainsi certaines composantes.

**En quoi la régularisation limite le sur-apprentissage ?**

Le terme de régularisation vient directement impacter la valeur de Z pour chacun des neurones. En augmentant la valeur du terme de régularisation $$\lambda$$, on réduit la valeur de W, et Z est proportionnel à W.

Ainsi, avec des valeurs de Z plus petites, on se ramène dans la partie linéaire de la fonction d'activation du neurones. De manière globale, on **réduit la complexité des règles apprises** par notre réseau en le limitant à des fonctions plus simples.

## Le Drop-out

Le DropOut consiste à déclarer une probabilité de désactiver un neurone. Ici encore, on simplifie le réseau en le rendant plus petit. On utilise uniquement le drop-out avec le jeu de données d'entrainement, pour éviter le sur-apprentissage.

Avec le drop-out, la fonction de coût est alors mal définie, il faut d'abord s'assurer que le modèle converge sans drop-out.

## Autres techniques

- On a vu plus haut que pour améliorer les performances, on peut fournir plus de données au réseau lors de l'entrainement. Une technique consiste à **générer de nouvelles données à partir de celles dont on dispose**. Par exemple par des rotations, des randoms crops...
- On peut également **arrêter l'entrainement du modèle prématurément** avant que les performances entre le train et le test soient trop importantes.

# Premiers traitements : 

## Normalisation des données

La normalisation des données, c'est super important, grâce à cette étape, on s'assure que le réseau apprend dans les meilleurs conditions.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/nn1/center-norm.png){: .align-center}

0. Sur le premier repère, on voit que les données sont réparties un peu trop librement dans l'espace. Notre but est de ramener leur répartition autour de zéro, afin d'interpréter chaque variable de la même manière.
1. On calcule la moyenne de l'ensemble des valeurs, et on la retranche à toutes les valeurs. Comme illustré sur le second repère, on centre les données.
2. On calcule la variance des données, et on divise l'ensemble des données par la variance. Comme illustré sur le troisième repère, on réduit l'amplitude des données.

**Evolution de l'allure de la fonction de coût avec la normalisation :**

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/nn1/cost-function.png){: .align-center}

Avec la normalisation des données, on passe du cas de gauche au cas de droite. On facilite ainsi la tâche pour la descente de gradient.

## Vanishing / Exploding gradient

Si on remonte à l'expression de $$\hat y$$ en fonction des matrices de poids des L couches successives, de manière simplifiée, sans tenir compte des termes de biais, on obtient :

$$\hat y = w^{[L]}w^{[L-1]}w^{[L-2]}...w^{[3]}w^{[2]}w^{[1]}.X$$

En developpant, on se rend compte que la matrice des poids est mise à la puissance L-1. 

- Si la matrice des poids comporte des petites valeurs, on parle donc de **vanishing gradient**. Par exemple $$0.9^L$$ diminue exponentiellement avec le nombre de couche L.
- Si la matrice des poids comporte de grandes valeurs, on parle d'**exploding gradient**. Idem, pour l'explosion exponentielle des valeurs.

Avec les réseaux actuels comportant 100, voire 200 couches, on devine facilement l'effet néfaste.

## Initialisation des poids

Pour résoudre partiellement le problème du vanishing/exploding gradient, on utilise diverses méthodes d'initialisation des poids de la matrice W. 

> L'idée est d'initialiser la matrice $$w^{[L]}$$ avec des valeurs autour de 1,et une variance de valeur $$\frac{1}{n}$$.

Pour les fonction tanh (Xavier init) :

$$w^{[l]} = np.random.randn(shape) * np.sqrt(\frac{1}{n^{[l-1]}})$$

Pour les fonctions ReLU :

$$w^{[l]} = np.random.randn(shape) * np.sqrt(\frac{2}{n^{[l-1]}})$$

# Sources

- [Coursera - Deep Learning](www.coursera.org/learn/neural-networks-deep-learning)
