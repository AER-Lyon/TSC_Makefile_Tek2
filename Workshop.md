# Makefiles

</br>

### Un exemple de projet : Hello World

</br>

```c
// hello.c

#include <unistd.h>

void say_hello(void)
{
    write(1, "Hello world !\n", 14);
}

```

```c
// hello.h

#ifndef H_HELLO
    #define H_HELLO

    void say_hello(void);

#endif

```

```c
// main.c

#include "hello.h"

int main(void)
{
    say_hello();
    return 0;
}

```

</br>

### La syntaxe des Makefiles

</br>

Les makefiles sont des fichiers composés de plusieurs règles suivant cette forme :

```
cible:	dépendances
	commandes
```

</br>

Lors de l'exécution de la commande make, la première règle rentrée, ou la règle spécifiée est évaluée. L'évaluation d'une règle se fait en suivant ces étapes :


- Les dépendances sont analysées, si une dépendance est la cible d'une autre règle du Makefile, cette règle est à son tour évaluée.

- Lorsque l'ensemble des dépendances est analysé et si la cible ne correspond pas à un fichier existant ou si un fichier parmi les dépendances est plus récent que le fichier cible, les différentes commandes sont exécutées.

</br>

    Attention, les commandes doivent impérativement précédées d'une tabulation.


</br>

### Un makefile minimal

</br>

On peut donc écrire une première version minimale de makefile pour notre mini projet :

```Makefile
# Makefile

hello: hello.o main.o
	gcc -o hello hello.o main.o

hello.o: hello.c
	gcc -o hello.o -c hello.c -Wall -Wextra -Werror

main.o: main.c hello.h
	gcc -o main.o -c main.c -Wall -Wextra -Werror
```

</br>

Découpons l'exécution lors de l'exécution de la commande `make` :

- La première règle évaluée est la première rencontrée, soit "hello".

- La première dépendance de cette règle fait référence à une autre règle du Makefile, elle va donc être évaluée.

- La dépendance de la règle "hello.o" n'est pas une autre règle, mais un fichier, deux cas de figures se présentent alors :

    - Soit le fichier hello.c est plus récent que le fichier cible hello.o, alors la commande sera exécutée.

    - Dans le cas contraire, la commande ne sera pas exécutéé.


- Les mêmes étapes sont appliquées pour la deuxième dépendance de la règle "hello" et elle-même, soit la commande ne sera exécutée que si un des fichiers hello.o ou main.o est plus récent que l'exécutable hello.

</br>

### Un Makfile plus évolué

</br>

Dans la version écrite précédemment, plusieurs choses posent problème :

- Il ne permet pas la création de plusieurs exécutables.

- Les fichiers temporaires restent présents (.o).

- Les exécutables ne peuvent pas être supprimés efficacement.

- Il n'est pas possible de forcer la génération intégrale du projet.

</br>

Ces problèmes trouvent leur solution dans l'ajout de plusieurs règles :

- La règle `all`, généralement la première du fichier, elle a pour dépendance, l'ensemble des exécutables à générer.

- La règle `clean`, elle permet de supprimer les fichiers temporaires.

- La règle `fclean`, elle permet de supprimer l'intégralité des fichiers générés, incluant les fichiers temporaires et les exécutables.

- La règle `re`, elle permet de regénérer l'ensemble du projet.

</br>

Voilà à quoi ressemble notre makefile maintenant :

```Makefile
# Makefile

all: hello

hello: hello.o main.o
	gcc -o hello hello.o main.o

hello.o: hello.c
	gcc -o hello.o -c hello.c -Wall -Wextra -Werror

main.o: main.c hello.h
	gcc -o main.o -c main.c -Wall -Wextra -Werror

clean:
	rm -rf *.o

fclean: clean
	rm -rf hello

re: fclean all
```

</br>

### Utilisation des variables

</br>

Il est en effet possible de définir des variables dans un makefile, cela les rend plus agréables à lire, mais aussi à faire évoluer.

Les variables se déclarent sous la forme NOM = VALEUR et s'utilisent via la syntaxe $(NOM).

Nous allons donc pouvoir rajouter des variables à notre makefile :

```Makefile
# Makefile

CC = gcc
CFLAGS = -Wall -Wextra -Werror
LDFLAGS =
EXEC = hello

all: $(EXEC)

hello: hello.o main.o
	$(CC) -o hello hello.o main.o $(LDFLAGS)

hello.o: hello.c
	$(CC) -o hello.o -c hello.c $(CFLAGS)

main.o: main.c hello.h
	$(CC) -o main.o -c main.c $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf hello

re: fclean all
```

</br>

On a ainsi défini plusieurs variables :

- CC compilateur utilisé pour la compilation du C.

- CFLAGS flags utilisés lors de la compilation du C.

- LDFLAGS flags utilisés lors de l'invocation du linker.

- EXEC contient le nom des exécutables à générer.


</br>

	Attention, les variables CC, CFLAGS et LDFLAGS sont des variables implicites utilisées par défaut par make. Pour plus de détaile, voir :

	https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html

</br>

### Les variables internes

</br>

Comme en shell script, il existe des variables internes au makefile :

|||
|:--:|:--:|
| $@ | Le nom de la cible |
| $< | Le nom de la première dépendance |
| $^ | La liste des dépendances |
| $? | La liste des dépendances plus récentes que la cible |
| $* | Le nom du fichier sans suffixe |

</br>

On peut donc simplifier notre makefile grâce à ces variables, ce qui nous donne :

```Makefile
# Makefile

CC = gcc
CFLAGS = -Wall -Wextra -Werror
LDFLAGS =
EXEC = hello

all: $(EXEC)

hello: hello.o main.o
	$(CC) -o $@ $^ $(LDFLAGS)

hello.o: hello.c
	$(CC) -o $@ -c $< $(CFLAGS)

main.o: main.c hello.h
	$(CC) -o $@ -c $< $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all
```

</br>

### Les règles d'inférence

</br>

La syntaxe des makefiles nous permet de déclarer des règles génériques, telles qu'une règle qui permet de définir la construction d'un fichier .o depuis un fichier .c :

```Makefile
%.o: %.c
	commandes
```

</br>

Il est alors possible de simplifier notre makefile :

```Makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror
LDFLAGS =
EXEC = hello

all: $(EXEC)

hello: hello.o main.o
	$(CC) -o $@ $^ $(LDFLAGS)

%.o: %.c
	$(CC) -o $@ -c $< $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all
```

</br>

Le problème de cette version est que le fichier main.o n'est plus reconstruit si le fichier hello.h est plus récent. Il est alors possible de faire fonctionner notre règle d'ingérence avec une règle permettant de spécifier la dépendance entre ces deux fichiers :

```Makefile
# Makefile

CC = gcc
CFLAGS = -Wall -Wextra -Werror
LDFLAGS =
EXEC = hello

all: $(EXEC)

hello: hello.o main.o
	$(CC) -o $@ $^ $(LDFLAGS)

main.o: hello.h

%.o: %.c
	$(CC) -o $@ -c $< $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all
```

</br>

### Le .PHONY

</br>

En parlant de dépendance, que ce passerait-il si un fichier ou un dossier nommé clean se trouvait au même endroit que notre makefile ? Et bien, la règle clean n'ayant pas de dépendance, le fichier ou le dossier serait considéré comme le plus récent et la règle ne serait jamais exécutée.

Pour palier à ce genre de problème, il existe la cible .PHONY. Les règles précisées comme dépendances de celle-ci seront exécutés de manières inconditionnelles, peu importe alors si un fichier existe avec le même nom.


```Makefile
# Makefile

CC = gcc
CFLAGS = -Wall -Wextra -Werror
LDFLAGS =
EXEC = hello

all: $(EXEC)

hello: hello.o main.o
	$(CC) -o $@ $^ $(LDFLAGS)

main.o: hello.h

%.o: %.c
	$(CC) -o $@ -c $< $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all

.PHONY: clean fclean re
```

</br>

### La construction des fichiers objets

</br>

Lors de la réalisation de projet de plus grande taille, on peut rapidement se retrouver avec de nombreux fichiers, il devient alors fastidieux de tous les lister dans la définition de nos règles de compilation. On va alors utiliser d'autre variable afin de résoudre ce problème :

- La variable SRC qui contiendra la liste de tous les fichiers source du projet.

- La variable OBJ qui contiendra la liste des fichiers objets.

La variable SRC se définit de manière assez simple :

```Makefile
SRC = hello.c main.c
```

Et si l'on réfléchit bien, le contenu de la variable OBJ est presque le même, à ceci près que, les fichiers se termineront en .o au lieu de .c. Or, il existe une syntaxe qui permet de faire cette conversion à partir de la variable SRC :

```Makefile
OBJ = $(SRC:.c=.o)
```

</br>

Voilà donc à quoi ressemble noter makefile avec ces deux nouvelles variables :

```Makefile
# Makefile

CC = gcc
CFLAGS = -Wall -Wextra -Werror
LDFLAGS =
SRC = hello.c main.c
OBJ = $(SRC:.c=.o)
EXEC = hello

all: $(EXEC)

hello: $(OBJ)
	$(CC) -o $@ $^ $(LDFLAGS)

main.o: hello.h

%.o: %.c
	$(CC) -o $@ -c $< $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all

.PHONY: clean fclean re
```

</br>

### L'utilisation de conditions

</br>

Lors de la phase de développement d'un projet, il est fortement recommendé d'utiliser les symboles de débogage pour pouvoir tester efficacement sont projets. Or, il ne faut pas que ceux-ci soient présents lors de la mise en production. Pour ce faire, on peut ajouter des conditions à notre makefile, pour que l'on puisse préciser si on est en phase de développement ou non :

```Makefile
# Makefile

DEBUG = yes

CC = gcc
ifeq ($(DEBUG), yes)
	CFLAGS = -Wall -Wextra -Werror -g3
	LDFLAGS =
else
	CFLAGS = -Wall -Wextra -Werror
	LDFLAGS =
endif
SRC = hello.c main.c
OBJ = $(SRC:.c=.o)
EXEC = hello

all: $(EXEC)
ifeq ($(DEBUG), yes)
	@echo "Génération en mode debug"
else
	@echo "Génération en mode release"
endif

hello: $(OBJ)
	$(CC) -o $@ $^ $(LDFLAGS)

main.o: hello.h

%.o: %.c
	$(CC) -o $@ -c $< $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all

.PHONY: clean fclean re
```

Ainsi, plutôt que de modifier le makefile à chaque fois, il suffit de modifier la variable DEBUG.

</br>

### Les sous-makefile

</br>

Plus les projets deviennent conséquents, plus il est conseillé de les subdiviser en plusieurs parties. Il n'est donc par rare de devoir compiler plusieurs parties d'un même projet pour qu'il fonctionne. Pour se faire et au lieu d'appeler plusieurs makefile manuellement, il est préférable de créer un makefile "maître" qui se chargera d'appeler les autres makefile du projet. Voici un exemple :

```Makefile
# Makefile maître

HELLO_DIR = hello
EXEC = $(HELLO_DIR)/hello

all: $(EXEC)

$(EXEC):
	$(MAKE) -C $(HELLO_DIR)

clean:
	$(MAKE) -C $(HELLO_DIR) $@

fclean: clean
	$(MAKE) -C $(HELLO_DIR) $@

re:
	$(MAKE) -C $(HELLO_DIR) $@

.PHONY: clean fclean re $(EXEC)
```

```Makefile
# Makefile

DEBUG = yes

CC = gcc
ifeq ($(DEBUG), yes)
	CFLAGS = -Wall -Wextra -Werror -g3
	LDFLAGS =
else
	CFLAGS = -Wall -Wextra -Werror
	LDFLAGS =
endif
SRC = hello.c main.c
OBJ = $(SRC:.c=.o)
EXEC = hello

all: $(EXEC)
ifeq ($(DEBUG), yes)
	@echo "Génération en mode debug"
else
	@echo "Génération en mode release"
endif

hello: $(OBJ)
	$(CC) -o $@ $^ $(LDFLAGS)

main.o: hello.h

%.o: %.c
	$(CC) -o $@ -c $< $(CFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all

.PHONY: clean fclean re
```

</br>

### La gestion des dépendances

</br>

Comme précisé précédemment, il est possible de créer des dépendances manuellement entre les fichiers sources et les header files. Mais, en c++ il arrive de travailler avec beaucoup de header files, il devient alors fastidieux d'écrire toutes les dépendances à la main. Heureusement, il existe une solution pour forcer la génération de ces dépendances lors de la compilation. Pour ce faire, il suffit de rajouter le flag `-MD` dans la règle d'inférence qui définit le passage des fichiers .cpp en .o et d'include les fichiers `.d` qui seront générés. Ce sont ces fichiers qui contiendront les informations nécessaires à la résolution des dépendances.

</br>

```Makefile
# Makefile

CC = g++
CXXFLAGS = -Wall -Wextra -Werror -std=c++20
LDFLAGS =

SRC = hello.cpp main.cpp
OBJ = $(SRC:.cpp=.o)

EXEC = hello

all: $(EXEC)

hello: $(OBJ)
	$(CC) -o $@ $^ $(LDFLAGS)

%.o: %.cpp
	$(CC) -o $@ -MD -c $< $(CXXFLAGS)

clean:
	rm -rf *.o

fclean: clean
	rm -rf $(EXEC)

re: fclean all

-include $(OBJ:%.o=%.d)

.PHONY: clean fclean re
```

</br>

### Arborescence de fichier (String Substitution)

</br>

Maintenant que nous avons vu comment générer les fichiers .d, il devient difficile de se repérer dans les fichiers sources qui sont mélangés avec les fichiers générés lors de la compilation. Pour résoudre ce problème, on va pouvoir se servir des règles de `string substitution`. En effet, celles-ci vont nous être utiles pour générer une copy de l'arborescence des fichiers sources pour les fichiers de compilation.

</br>

```Makefile
# Makefile

CC = g++
CXXFLAGS = -Wall -Wextra -Werror -std=c++20 -I ./include
LDFLAGS =

SRC_DIR = ./src
SRC =   $(SRC_DIR)/hello.cpp \
        $(SRC_DIR)/main.cpp

BUILDDIR = ./obj
OBJ = $(patsubst $(SRC_DIR)/%.cpp,$(BUILDDIR)/%.o,$(SRC))

EXEC = hello

$(BUILDDIR)/%.o:	$(SRC_DIR)/%.cpp
	@mkdir -p $(dir $@)
	$(CC) -o $@ -MD -c $< $(CXXFLAGS)

all: $(EXEC)

hello: $(OBJ)
	$(CC) -o $@ $^ $(LDFLAGS)

clean:
	rm -rf $(BUILDDIR)

fclean: clean
	rm -rf $(EXEC)

re: fclean all

-include $(OBJ:%.o=%.d)

.PHONY: clean fclean re
```

</br>

Ici, on utilise la fonction de string substitution `patsubst` afin de créer la copie de l'architecture des fichiers source.

Cette fonction respecte le pattern suivant :

```Makefile
$(patsubst pattern,replacement,text)
```