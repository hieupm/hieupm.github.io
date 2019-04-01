---
published: true
title: "Paramètres des neurones"
excerpt: ""
toc: true
toc_sticky: true
toc_label: "Fonctions d'activation"
toc_icon: "microchip"
comments: true
author_profile: false
header:
  overlay_image: "assets/images/covers/cover2.jpg"
  teaser: "assets/images/covers/cover2.jpg"
categories: [deep, learning, activation]
---

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

Un choix important dans la conception de réseaux de neurones est le choix des fonctions d'activation dans chacune des couches du réseau. Elles ne sont pas forcément les mêmes partout.

Dans tout les articles précédents, la fonction Sigmoïde était utilisée, parfois d'autres choix sont bien meilleurs.

> Une **fonction d'activation** sert à obtenir la valeur de sortie d'un neurone. On peut aussi l'appeler fonction de transfert.

# Fonctions d'activation

Dans notre réseau, si on utilise des fonctions d'activation linéaires dans la couche cachée, on obtient en sortie une fonction linéaire de l'entrée.

Dans un réseau avec plusieurs couches cachées, on peut montrer qu'avoir plusieurs couches cachées avec des fonctions linéaires, revient à ne pas avoir de couches cachées du tout.

Exemple : Si on veut prédire un prix, une fonction linéaire serait adaptée pour la couche de sortie. Mais à ce moment là il faudrait quand même préférer une fonction ReLU, de sorte à bloquer les valeurs négatives, qui n'auraient pas de sens.

## Sigmoïde

Dans chacun des neurones, on utilisait pour le calcul de **a** la fonction sigmoïde $$\sigma(z^{[1]})$$ qui s'exprime ainsi :

$$ a = \frac{1}{1 + e^{-z}}$$

![image-center](https://upload.wikimedia.org/wikipedia/commons/thumb/6/66/Funci%C3%B3n_sigmoide_01.svg/800px-Funci%C3%B3n_sigmoide_01.svg.png){: .align-center}

Dans le cas de la classification binaire 0/1, il vaudra mieux prendre une fonction sigmoïde qui fournira directement le bon résultat.

## Tangente Hyperbolique

Un choix souvent mieux que la fonction sigmoïde est la **tangente hyperbolique (tanh)**, qui donne des valeurs de **a = tanh(z)** entre -1 et 1 , qui s'exprime :

$$a = tanh(z) = \frac{e^{z}-e^{-z}}{e^{z}+e^{-z}}$$

![image-center](https://upload.wikimedia.org/wikipedia/commons/thumb/8/87/Hyperbolic_Tangent.svg/1200px-Hyperbolic_Tangent.svg.png){: .align-center}

Il s'avère que pour les couches cachées, si on prend une fonction d'activation $$g(z)$$, ça fonctionne quasiment toujours mieux que la fonction sigmoïde.

## Limitation 

Une limitation des fonctions précédentes apparait pour des valeurs très grandes (positif/négatif). En effet, pour les grandes valeurs, la valeur est quasi constante, et cela empêche la descente de gradient de se faire efficacement.

## Rectified Linear Unit

### ReLU

La fonction **ReLU** vaut $a = max(0, z)$. Sa dérivée vaut zéro ou 1 (sauf en zéro où c'est théoriquement plus compliqué), cela rend les calculs plus faciles. C'est maintenant le choix à privilégier.

![image-center](https://cdn-images-1.medium.com/max/1600/1*DfMRHwxY1gyyDmrIAd-gjQ.png){: .align-center}

La pente de cette fonction est très différente de zéro sur un grand intervalle. En pratique, cette fonction permet au réseau d'apprendre plus vite qu'avec une sigmoïde ou une fonction tanh.

Le problème avec la fonction ReLU est que le neurone meurt pour une grande plage de valeurs, car sa dérivée vaut zéro. Ce problème a provoqué l'apparition du Leaky ReLU.

### Leaky ReLU

La fonction Leaky ReLU vaut $max(0.01z, z)$, il y a une pente de 0.01 pour les valeurs négatives. La valeur de la pente est également une valeur paramétrable lors de l'optimisation du réseau.

![image-center](https://cdn-images-1.medium.com/max/1600/1*ypsvQH7kvtI2BhzR2eT_Sw.png){: .align-center}

# Dérivées des fonctions d'activation

Au moment de la **back-propagation**, on calcule les dérivées successives dans notre réseau de neurones. Bien sûr la *dérivée de la fonction d'activation* joue un rôle important dans ce calcul.

## Sigmoïde

On a vu plus haut la fonction sigmoïde et son expression : $ g(z) = \frac{1}{1 + e^{-z}}$

La dérivée est égale à :

$$g'(z) = \frac{1}{1+e^{-z}}(1-\frac{1}{1+e^{-z}}) = g(z)(1-g(z))$$

Pour les grandes valeurs de z, positif et negatif, la dérivée est nulle. Elle atteint son maximum en 0 où elle vaut $0.25$.

## Tangente Hyperbolique

Pour la fonction $tanh$ , la dérivée s'exprime : $g(z) = 1-tanh(z)^2$

Pour les grandes valeurs de z, on a également la dérivée qui vaut zéro, avec un maximum en zéro.

## ReLUs

- Pour la ReLU $g(z)=max(0, z)$, la dérivée prend la valeur 0 ou 1, et elle n'est pas définie au point 0.
- Pour la Leaky ReLU $g(z)=max(0.01z, z)$, la dérivée prend la valeur 0.01 ou 1, et non définie au point 0.

# Initialisation des poids

Quand on entraine un réseau de neurones, il est important d'initialiser les poids aléatoirement. Pour le cas basique de la régression logistique, initialiser à zéro ne posait pas problème, mais pour un réseau de neurones ça ne fonctionne pas.

## Initialisation à zéro

Prenons l'exemple d'un réseau avec :
- Deux variables d'entrée
- Deux neurones sur la couche cachée
- Un neurone d'activation

Avec une initialisation à zéro, la matrice des poids pour la couche cachée, ne contiendra que des zéros. Les poids étant identiques pour les deux neurones, ils vont donner la même valeur en sortie.

Il se trouve qu'au bout de quelques itérations, les neurones de la couche cachée calculent **exactement la même fonction**, car la mise à jour de leur poids est aussi identique. Ainsi de suite, les deux neurones donnent la même valeur à chaque itération. 

Il n'y a donc aucun intérêt à avoir plus d'un neurone dans la couche cachée, puisque chacun aura exactement la même influence sur la couche de sortie. 

> Il est donc important que les poids soit différents pour chacun des neurones, afin qu'ils "apprennent" de manière différente.

## Initialisation aléatoire

Création d'un vecteur de poids aléatoires, suivant une loi gaussienne aléatoire :

```python
w1 = np.random.randint((2, 2)) * 0.01
```

On multiplie par une très petite valeur pour s'assurer que les valeurs sont faibles, et être proche de zéro. Ainsi, on se situe dans la zone d'activation de la fonction non linéaire d'activation (sigmoid, tanh, relu). 

Si les valeurs d'initialisation des poids sont trop grands, on se situe dans la zone plate de la fonction d'activation, ce qui ralentirait fortement la vitesse de convergence de la descente de gradient.

Pour le **biais**, le problème de symétrie n'apparait pas et ça ne pose pas de problème de tout initialiser à zéro.

```python
b = np.zero((2, 1))
```

L'initialisation par des valeurs aléatoires de l'ordre de 0.01, peut bien fonctionner pour un réseau de faible profondeur. Pour les réseaux plus grands, il y a des techniques pour avoir une meilleure convergence de la descente de gradient. 

# Sources
- [TowardsDataScience - Sagar Sharma - Activation Functions in Neural Networks](https://towardsdatascience.com/activation-functions-neural-networks-1cbd9f8d91d6)
- [Coursera - Deep Learning](www.coursera.org/learn/neural-networks-deep-learning)
