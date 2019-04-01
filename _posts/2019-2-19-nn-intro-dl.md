---
published: true
title: "Qu'est-ce qu'un réseau de neurones ?"
excerpt: "Principe et classification binaire"
classes: wide
comments: true
author_profile: false
header:
  overlay_image: "assets/images/covers/cover1.jpg"
  teaser: "assets/images/covers/cover1.jpg"
categories: [nn]
---

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

Parmi les exemples d'usages classiques des algorithmes de machine learning on peut citer : la prédiction du prix d'une maison avec sa surface habitable, la detection de chats sur une image... Nous verrons que pour chaque problème, on trouve des algorithmes particuliers dans la famille des réseaux de neurones.

Pour introduire la notion de réseau de neurones, commençons par prendre l'exemple d'un neurone seul : C'est le Perceptron, inventé en 1957 par Frank Rosenblatt. Le Perceptron est une seule cellule, un seul neurone, qui permet de résoudre les problèmes de classification binaire : prédire 0 ou 1. Le neurone va trouver la frontière séparant les données 0 et 1.

Le Perceptron est à la base de l'analogie avec les neurones:
- Il s'agit d'une cellule qui reçoit des informations en entrée, tout comme un neurone humain capte des signaux par les *Dendrites*.
- Les variables en entrée sont passées dans une fonction d'activation (de décision), comme les signaux captés sont rassemblés dans le corps cellulaire, ou *Soma*, décide de transmettre l'information à d'autres neurones du cerveau.

Le Perceptron est une fonction mathématique, prenant des variables d'entrée pondérées par des poids, et un terme de biais. Dans un espace à deux dimensions, ceux-ci sont comparables au coefficient directeur, et à l'ordonnée à l'origine.

**Dans un espace à deux dimensions, le Perceptron est capable de tracer une droite pour séparer des points entre deux classes.**

![image](/assets/images/nn1/perceptron.png){: .align-center }


# Apprentissage supervisé avec les Neural Nets

### Exemples d'applications

- Caractéristiques d'une maison, prediction de son prix. (NN)
- Publicités et données utilisateur, prédire le clic. (NN)
- Images avec labels, trouver les objets sur l'image. (CNN)
- Audio, transcription de texte. (RNN)
- Image et radar, navigation véhicules. (Custom, Hybride)

### L'apprentissage supervisé

En entrée des réseaux, on trouve des données **structurées** comme des tables de prix, des tables de données utilisateur : Des occurrences de données avec plusieurs champs renseignés.

On trouve également des données **non structurées** : les données audio, les images, les textes. C'est une tâche qui a été plus complexe à maitriser pour les algorithmes et qui connait un succès récent.

# Comment expliquer le succès du Deep Learning ?

Historiquement, les **algorithmes classiques** ne donnaient pas de meilleures performances lorsqu'ils disposent de plus de données. On a désormais accès à de plus en plus de données avec la digitalisation.

Les **réseaux de neurones** ont permis de tirer avantage de ces plus grandes quantités de données et d'augmenter les performances.

C'est donc un **effet d'échelle** qui a permis le progrès des réseaux de neurones, l'échelle des données, mais aussi l'échelle du réseau de neurones : la profondeur du réseau a augmenté au cours du temps.

# Classification binaire

Il s'agit de classifier le contenu d'une image, il y a un chat (1) ou pas de chat (0).

On part de l'image d'entrée en dimension 64x64, qui est en réalité un ensemble de 3 matrices 64x64, pour les trois intensités couleurs Rouge, Vert, Bleu. Notre matrice de données en entrée est finalement une concaténation de toutes les intensités dans un grand vecteur vertical.

$$(x, y) x \in \mathbb{R}$$ m exemples d'entrainement $${(x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)}), ..., (x^{(n)}, y^{(n)})}$$

La matrice X d'entrainement est de dimension (Nx, m), avec Nx le nombre d'éléments dans un vecteur de données, et m le nombre d'exemples d'entrainement. $$X \in \mathbb{R}^{Nx . m}$$

En sortie, on a un vecteur de labels $$Y = [y^{(1)}, y^{(2)}, ..., y^{(n)}] \in \mathbb{R}^{1 \times m}$$

## Problème vu comme une Regression Logistique

La regression logistique est utilisée pour la classification binaire supervisée. Pour un vecteur d'entrée X (une image), on considère la prédiction et on l'appelle Y chapeau. Y chapeau, est la probabilité d'obtenir 1, sachant l'entrée X (Quelle est la chance d'avoir un chat sur l'image).

On pourrait tenter une approche de régression linéaire en estimant $$\hat y = w^{T} + b$$, mais cela ne donnerait pas une grandeur entre 0 et 1, on recherche une probabilité.

On va donc appliquer une fonction d'activation, la fonction sigmoïde, qui renvoie les valeurs dans l'intervalle [0, 1] : $$\hat y = \sigma(w^{T} + b)$$. La fonction sigmoïde s'exprime ainsi : $$\sigma(z) = \frac{1}{1+e^{-z}}$$

On revient finalement au problème d'estimation des paramètres w et b pour que $$\hat y$$ soit un bon estimateur de la probabilité que Y vale 1.
- W est un vecteur de taille $$n_{x}$$
- b est un réel

## Fonctions de perte et de coût associées à la régression logistique

Pour entrainer les paramètres W et b, il faut définir une fonction de perte. Pour un ensemble d'entrainement $${(x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)}), ..., (x^{(m)}, y^{(m)})}$$, on veut notre estimation $$\hat y^{i}$$ la plus proche $$y^{i}$$.

La **fonction de perte (Loss)** est définie par :

$$\mathfrak{L}(\hat y, y) = -(y\log \hat y+(1-y)\log(1-\hat y))$$

Cette fonction de Loss est adaptée car elle est convexe et la descente de gradient arrivera à trouver un minimum. La procédure d'apprentissage va consister à minimiser cette fonction, elle est en effet minimale pour $$\hat y^{i}$$ égal à $$y^{i}$$.

La **fonction de  coût (Cost)** est définie par l'application de la fonction de perte à chacun des exemples d'entrainement :

$$J(w, b)=\frac{1}{m}\sum_{1}^{m}\mathfrak{L}(\hat y^{(i)},y^{(i)})$$

# Sources

- <a href="https://www.coursera.org/learn/neural-networks-deep-learning/home/welcome" target="_blank">Deep Learning course</a> (Coursera - Andrew Ng)
