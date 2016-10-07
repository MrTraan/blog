---
title: Une histoire sans fin
date: 2016-10-06 11:37:59
tags:
---
>*Un algorithme récursif est un algorithme qui résout un problème en calculant des solutions d'instances plus petites du même problème.*

La première fois fois que j'ai dû utiliser la récursivité dans un programme, c'était pendant le concours d'entrée de mon école (42). A l'époque, j'avais trouvé ça facile à implémenter, mais j'avais pas compris grand chose.

Un an après... J'en suis toujours à peu près au meme point. Je me suis pas servi très souvent d'algorithmes récursifs dans mes programmes et j'ai pas eu l'impression de m'en servir de la bonne façon.
Alors du coup j'ai voulu reprendre les bases. Un des premiers problèmes qu'on nous avait donné, comme je pense dans la plupart des écoles d'informatique sur ce sujet, c'était d'écrire un programme pour calculer la valeur d'un rang N de la suite de Fibonacci.

La suite de Fibonacci c'est l'histoire de lapins qui se reproduisent indéfiniment. Une sombre histoire d'un italien un peu lubrique, mais qui a des résultats assez cool puisqu'elle permet de faire une approximation assez précise du [nombre d'or](https://fr.wikipedia.org/wiki/Nombre_d%27or).

Je copie les explications du problème depuis [wikipedia](https://fr.wikipedia.org/wiki/Suite_de_Fibonacci):
> Au début du premier mois, il y a juste une paire de lapereaux.
> Les lapereaux ne procréent qu'à partir du début du troisième mois. Chaque début de mois, toute paire susceptible de procréer engendre effectivement une nouvelle paire de lapereaux.
Les lapins ne meurent jamais.

La suite de Fibonacci c'est la représentation mathématiques de cette situation.
Je vous passe la demonstration mais on trouve que:
`f(n) = f(n - 1) + f(n - 2)`
En gros, le nombres de lapins en avril, c'est le nombre de lapin de mars + le nombre de lapin de février (c'est que ca se reproduit vite ces chose là).

En programmation c'est facile à représenter. Je vais le faire en C mais vous trouverez vite un implémentation quasiment similaire dans tous les languages du monde. On lancera le programme en passant en argument la nombres de mois a calculer.

```c
#include <stdlib.h>
#include <stdio.h>

int fibonacci(int n) {
	// On retourne 2 si n correspond au premier
	//  ou au deuxieme mois
	if (n == 1 || n == 2)
		return (2);

	// Sinon on retourne la somme des
	// deux rangs precedents 
	return fibonacci(n - 1) + fibonacci(n - 2);
}

int main(int ac, char **av) {
	// On quitte s'il n'y a pas d'argument
	if (ac != 2)
		return (1);
	
	// On convertit en nombre l'argument
	int n = atoi(av[1]);

	// On calcule la valeur de la suite pour le rang n
	// et on affiche le resultat
	int val = fibonacci(n);
	printf("La valeur de la suite de fibonacci pour le rang %d est: %d\n", n, val);
	return (0);
}
```

Bon du coup, ca marche. Si on veut aller plus loin, il va deja falloir stocker la valeur de la suite dans un type plus gros, parce que le resultat grimpe très vite.

```c
#include <stdlib.h>
#include <stdio.h>

// J'utilise un typedef pour clarifier le code
// ull devient un racourcis pour le type
// unsigned long long
typedef unsigned long long ull;

ull fibonacci(int n) {
	if (n == 1 || n == 2)
		return (2);

	return fibonacci(n - 1) + fibonacci(n - 2);
}

int main(int ac, char **av) {
	if (ac != 2)
		return (1);
	
	int n = atoi(av[1]);
	ull val = fibonacci(n);
	printf("La valeur de la suite de fibonacci pour le rang %d est: %llu\n", n, val);
	return (0);
}
```

J'ai testé le programme pour le rang 50:
on trouve un resultat de 25 172 538 050 lapins. 25 milliards et des poussieres de lapins en 50 mois. Au calme. Mais on est encore loin de la limite des unsigned long long, qui sont encodés sur 64 bits et qui vont donc jusqu'a 1 << 64 = 18 446 744 073 709 551 616.
Pourtant ca prend deja pas mal de temps a calculer, sur un mac book pro j'en suis a 1 min 8s de calcul pour le rang 50 et ca monte de facon exponentielle:
![performance chart](http://images.mrtraan.love/fibo_chart1.png)

Si ca prend autant de temps, c'est a cause du nombre d'appel de fonctions réalisé, qui suit la meme logique que la suite de Fibonacci. Pour les valeurs n = 1 ou n = 2, on appellera la fonction recursive une fois (puisqu'elle retournera directement une valeur au lieu d'appeler d'autres fonctions). Ensuite, puisqu'on appelle f(n - 1) et f(n - 2), on appelera autant de fonctions que la somme des appels des deux rangs precedents. Donc
`nbr_appels(n) = fibonacci(n) - 1`
Le -1 est la car le terme constant pour les rangs 1 et 2 vaut desormais 1 appel de fonction, au lieu de nos 2 lapins.

Ca veut donc dire que pour calculer le rang 50 de la suite de fibonacci, j'ai appelé 25 172 538 049 de fois ma fonction recursive.

J'ai décidé d'implémenter un systeme de cache qui stockerai les valeurs déjà calculées, au lieu de les recalculer a chaque fois:
```c
#include <stdlib.h>
#include <stdio.h>
#include <time.h>
#include <string.h>

typedef unsigned long long ull;
static ull function_calls = 0;

ull fibonacci(int n, ull *cache) {
	function_calls++;
	if (n == 1 || n == 2)
		return (2);

	// Si la valeur courante est en cache on la retourne
	if (cache[n] != 0) {
		return cache[n];
	}

	// Sinon on la calcule puis on la mets en cache avant de la retourner
	ull current_value = fibonacci(n - 1, cache) + fibonacci(n - 2, cache);
	cache[n] = current_value;
	return current_value;
}

int main(int ac, char **av) {
	if (ac != 2)
		return (1);
	clock_t tic = clock();

	int n = atoi(av[1]);

	// On cree un tableau de n + 1 unsigned long long
	// et on l'initialise a 0 sur tous les elements
	ull *cache = malloc(sizeof(ull) * (n + 1));
	memset(cache, 0, sizeof(ull) * (n + 1));

	ull val = fibonacci(n, cache);
	clock_t toc = clock();
	printf("%d;%f;%llu;%llu\n", n, (double)(toc - tic) / CLOCKS_PER_SEC, function_calls, val);

	free(cache);
	return 0;
}
```
*En plus du cache, j'ai rajouté quelques lignes pour compter le nombre d'appels de fonctions et pour chronométrer mon programme.*

Le programme peut sembler inutilement compliqué si vous ne faites pas du C, c'est juste que j'utilise la fonction `malloc` pour déclarer un tableau pour stocker mes valeurs.

Concernant les performances, ca n'a plus rien a voir. Tous les calculs sont instantanés:
![performance chart](http://images.mrtraan.love/fibo_chart2.png)
*Les variations sont normales, le temps d'execution est tellement court que le moindre cycle de cpu a un impact sur la performance.*

Et oui l'echelle est bien en micro secondes, soit un millionième de secondes. Je suis allé jusqu'au rang 200 cette fois, mais ca n'a pas beaucoup de sens puisqu'a partir du rang 92 on atteint la valeur de 15 080 227 609 492 692 858 (cé bocou) et que du coup on part en overflow et les resultats deviennent faux.

J'ai constaté que le nombre d'appel de fonction, a partir du rang 3, correspond a `n * 2 - 3` (je n'ai pas su démontrer pourquoi). Ce nombre évolue donc de façon linéaire cette fois, avec un facteur minuscule par rapport aux valeurs calculées.

Mais du coup qu'est-ce que ca veut dire tout ca?

En prenant un problème de base en programmation recursive, on peut optimiser le temps de calcul de façon délirante en mettant en place un système de cache, qui en plus ici a un impact négligeable sur la mémoire utilisée.
J'ai eu tendance a éviter d'utiliser des algorithmes récursif, et je constate ici qu'une petite variation dans l'implémentation peut changer la puissance du programme du tout au tout.
Alors certes, on reste ici sur un probleme purement théorique. Mais je garderai cet exemple en tête lorsque je travaillerai sur des cas plus concret, comme un parser de RegExp qui utiliserai sans doute des fonctions récursives.
