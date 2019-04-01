---
published: true
title: "Représentations deep"
excerpt: "Pourquoi on les utilise, comment les représenter"
toc: true
toc_sticky: true
toc_label: "Deep representations"
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

# Pourquoi "deep" ?

## Approche conceptuelle

On peut d'abord voir les réseaux profonds comme des algorithmes capables d'apprendre des règles très complexes. Plus le réseau est profond, plus il est capable d'apprendre des notions complexes.

### Detection de visages

- **Couche 1** : Détection de features basiques, détection des bords du visage.
- **Couche 2** : Assemblage de plusieurs bordures pour reconnaitre un visage, reconnaissance de features tels que le nez ou les yeux. 
- **Couche 6** : Assemblage des features complexes et distinction de types de visages. 

### Traitement audio

- **Couche 1** : Perception de variations audio bas niveau. 
- **Couche 2** : Détection de phonèmes "C", "A"..
- **Couche 3** : Assemblage de phonèmes en mots.
- **Couche 4** : Assemblage de mots en phrases.

## Intérêt calculatoire

D'un point de vue calculatoire on peut montrer qu'un réseau profond permet de **réduire la complexité** de calcul d'un XOR sur n variables.

Si on utilise un réseau de **plusieurs couches** où chaque neurone effectue l'operation XOR, on arrive à une complexité de calcul de $$O(log n)$$.

En revanche, si on utilise une **couche unique**, on serait contraint d'utiliser $$2^n$$ neurones afin de modéliser toutes les combinaisons possibles.

# Représentation en blocs

Le diagramme ci-dessous reprend de manière condensée l'évolution des variables au sein d'un réseau de neurones, dans les deux sens : forward et backward propagation.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/nn1/nn-blocks.png){: .align-center}

# Paramètres, Hyperparamètres

Il est important de faire la distinction entre les **paramètres** et les **hyper-paramètres**.

- **Paramètres** : $$w^{[1]}, b^{[1]}...$$, ce sont les paramètres de poids, de biais, du réseau, qui évoluent au cours de l'entrainement du réseau de neurones.
- **Hyper-paramètres** : Ce sont tout les autres paramètres, qui vont influer sur l'évolution des paramètres : Learning rate, Nombre d'iterations (epoch), Nombre de couches, Nombre de neurones par couche, fonctions d'activation, batch..

# Sources

- [Coursera - Deep Learning](www.coursera.org/learn/neural-networks-deep-learning)
