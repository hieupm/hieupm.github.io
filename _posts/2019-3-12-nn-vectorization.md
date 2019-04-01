---
published: true
title: "Python : La vectorisation"
excerpt: "Fini les boucles for"
toc: true
toc_sticky: true
toc_label: "Vectorization"
toc_icon: "infinity"
comments: true
author_profile: false
header:
    overlay_image: "assets/images/covers/cover1.jpg"
    teaser: "assets/images/covers/cover1.jpg"
categories: [ml, python]
---
<script type="text/javascript" async
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Vectorisation

La **vectorisation**, c'est la technique de base pour se débarasser des boucles *for* dans le code.

On a vu dans les premiers exemples de la regression logistique que la première opération consiste à multiplier deux vecteurs $$w$$ et $$x$$ dans la formule suivante :

$$z = w^\intercal \mathbf{x} + b$$

L'approche **classique**, non-vectorisée serait d'itérer sur chacun des indices de ces vecteurs avec une boucle for sur l'intervalle des indices.

Avec python, pour effectuer cette même opération de manière vectorisée, on utilise la fonction `np.dot(w, x) + b` , de la librairie numpy. 

Il se trouve que ce type d'opération est extrêmement **rapide** : Pour multiplier deux vecteurs de taille 1 million, la boucle for prend **500 fois plus de temps** que l'opération avec np.dot. 

En effet, les opérations de la librairie numpy sont capables de **paralleliser** les opérations sur le CPU/GPU au lieu de les effectuer séquentiellement (comme c'est le cas avec une boucle for).

# Quelques exemples

Multiplication de matrices/vecteurs :

```python
u = np.dot(A, v)
```

On dispose d'un vecteur v de dimension n, et on souhaite **appliquer une fonction à chacun des éléments** du vecteur :

```python
u = np.exp(v)
u = np.log(v)
u = np.abs(v)
u = np.maximum(v, 0)
```

Utilisation d'un **vecteur de poids** en entrée du réseau de neurones, plutôt qu'une variable pour chacun :

```python
dw = np.zeros((nx, 1))  # Initialisation
dw += xi * dzi          # Mise à jour
dw /= m                 # Moyenne train-set
```

# Regression logistique

Pour ne plus avoir besoin de boucles dans notre implémentation de la régression logistique, on va utiliser les formes matricielles : X est la matrice de données d'entrainement, contenant nx variables sur les lignes, et m exemples sur les colonnes.

La première étapes étant de calculer la fonction Z, on l'écrit comme un produit de matrices, et on obtient avec **une seule opération** toutes les valeurs de Z pour l'ensemble du train-set :

$$Z = [z^{(1)}, z^{(2)}, ..., z^{(m)}] = w^\intercal . X + [b, b, ..., b] $$

En python, cela revient à :

```python
Z = np.dot(w.T, X) + b
```

L'étape suivante est d'évaluer chacune des valeurs de Z par la fonction sigmoïde. Il suffira de déclarer la fonction sigmpïde dans une fonction python et l'appliquer à l'ensemble du vecteur.

# Calcul du gradient par vectorisation

## Calcul de dZ

On a vu que pour chaque exemple de notre train-set, on a besoin de calculer $$dz^{(i)}$$. Ici encore on va regrouper ces valeurs dans un vecteur de taille m :

$$ dZ = [dz^{(1)}, dz^{(2)}, ..., dz^{(m)}]$$

En définissant deux vecteurs de taille m pour les valeurs de A et de Y. On peut simplement calculer le vecteur dZ car pour chacun des termes, c'et la différence $$a^{(i)} - y^{(i)}$$. En python cela va simplement s'exprimer :

```python
dZ = A - Y
```

## Vectorisation des m exemples

Pour la grandeur db, qui est la somme des valeurs dans le vecteur dZ on peut faire :

```python
db = 1/m * np.sum(dZ)
```
Pour la grandeur $$dw = \frac{1}{m} . X . dz^\intercal$$ on peut ici aussi utiliser une ligne de code :

```python
dw = 1/m * np.sum(X, dZ.T)
```

En regroupant toutes les simplifications grâce aux matrices, on peut faire un **passage sur tout le train-set** (forward propagation) avec ces lignes :

```python
Z = np.dot(w.T, X) + b
A = sigmoid(Z)
dZ = A -Y
dw = 1/m * np.sum(X, dZ.T)
db = 1/m * np.sum(dZ)

w = w - r * dw
b = b - r * db
```

On a encore besoin d'une boucle pour effectuer **chacun des passages** de notre descente de gradient.

# Broadcasting

Le broadcasting est beaucoup utilisé dans les **opérations matricielles**. Lorsque l'on souhaite effectuer des opérations entre deux matrices (ici, des numpy arrays), on ne fournit pas toujours deux tables de mêmes dimensions.

Dans ce cas, numpy va chercher la **dimension commune** des deux matrics, et **étendre la plus petite**, dans la direction des lignes ou des colonnes, afin de les faire correspondre, et effectuer l'opération terme à terme.

# Vecteurs numpy

Lorsque l'on crée un vecteur aléatoire avec `np.random.randn(5)`par exemple, on obtient un vecteur de rang 1, de dimension `(5,)`, dont la transposée est égal au vecteur. Il vaudra mieux exprimer les dimensions complètes avec `np.random.randn(5, 1)`, qui donne réellement un vecteur colonne manipulable.

- Sommer verticalement : `A.sum(axis=0)`
- Verifier la dimension d'un vecteur : `assert(a.shape == (5, 1))`

# Avec plusieurs données d'entrainement

Comme on l'a fait pour l'exemple de la regression logistique, on va utiliser des tableaux numpy pour effectuer tout nos calculs d'un coup.

Pour la matrice X :
- Index horizontal : exemples d'entrainement
- Index vertical : les différentes features

Pour les matrices A et Z :
- Index horizontal : Exemples d'entrainement
- Index vertical : On descend avec la profondeur du réseau, correspond aux neurones des couches cachées.

## Stacking des données

Au debut de l'article on a vu comment effectuer les opérations de tout les neurones d'une couche avec les vecteurs numpy.

Pour se débarasser de la boucle FOR qui nous fait itérer sur les données d'entrainement, on va maintenant rassembler nos données dans une matrice numpy.

Notre matrice X des données d'entrainement :

$$X = \left[ \begin{array}{ccc} \\ x^{(1)} \space x^{(2)} \space x^{(3)} \space ... \space x^{(m)} \\ \space \end{array} \right]$$

On peut manipuler directement toutes nos données dans les produits de matrices suivants :

$$Z^{[1]} = W^{[1]}X + b^{[1]}

A^{[1]} = \sigma(Z^{[1]})

Z^{[2]} = W^{[2]}A^{[1]}+b^{[2]}

A^{[2]} = \sigma(Z^{[2]})$$

# Sources
- <a href="https://www.coursera.org/learn/neural-networks-deep-learning/home/welcome" target="_blank">Deep Learning course</a> (Coursera - Andrew Ng)
