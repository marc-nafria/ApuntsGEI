# Resum del segon Control de Laboratori

## SAT amb costos

La idea de **SAT** amb costos es definir un predicat que donat un cost, escrigui les **clausules** necessàries per a que es compleixi. Després, iterarem de manera **descendent en cost** per a trobar la solució amb menys cost possible. Així, generem solucions del nostre problema de cost **n-1** en cada iteració fins que *kissat* retorni que és insatisfactible el model proposat.

### Exemple: Min coloring

```prolog
%%%%%%% =======================================================================================
%
% Our LI Prolog template for solving problems using a SAT solver.
%
% It generates the SAT clauses, calls the SAT solver, shows the solution and computes its cost.
% Just specify:
%       1. SAT Variables
%       2. Clause generation
%       3. DisplaySol: show the solution.
%       4. CostOfThisSolution: computes the cost
%
%%%%%%% =======================================================================================

symbolicOutput(0).  % set to 1 for DEBUGGING: to see symbolic output only; 0 otherwise.



%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% Find the minimal number of colors needed to color a graph.
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%


%%%%%%% Begin example input %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

numNodes(15).
adjacency(1, [  2,3,4,5,6,    9,10,11,12,13,14,15]).
adjacency(2, [1,  3,4,5,6,7,  9,   11,12,      15]).
adjacency(3, [1,2,  4,5,6,7,8,9,10,   12,13,14   ]).
adjacency(4, [1,2,3,  5,6,7,  9,10,11,12,13,   15]).
adjacency(5, [1,2,3,4,    7,8,9,      12,13,   15]).
adjacency(6, [1,2,3,4,      8,  10,11,         15]).
adjacency(7, [  2,3,4,5,    8,9,10,11,      14   ]).
adjacency(8, [    3,  5,6,7,  9,10,      13,14,15]).
adjacency(9, [1,2,3,4,5,  7,8,     11,         15]).
adjacency(10,[1,  3,4,  6,7,8,     11,12,   14,15]).
adjacency(11,[1,2,  4,  6,7,  9,10,   12,13,14   ]).
adjacency(12,[1,2,3,4,5,        10,11,   13,14,15]).
adjacency(13,[1,  3,4,5,    8,     11,12,   14,15]).
adjacency(14,[1,  3,      7,8,  10,11,12,13,   15]).
adjacency(15,[1,2,  4,5,6,  8,9,10,   12,13,14   ]).

%%%%%%% End example input %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%


%%%%%%% Some helpful definitions to make the code cleaner: ====================================

node(I):-   numNodes(N), between(1,N,I).
edge(I,J):- adjacency(I,L), member(J,L).
color(C):-  numNodes(N), between(1,N,C).

%%%%%%% End helpful definitions ===============================================================


%%%%%%%  1. Declare SAT variables to be used: =================================================

% x(I,C)    meaning  "node I has color C"
satVariable( x(I,C) ):- node(I), color(C).


%%%%%%%  2. Clause generation for the SAT solver: =============================================

% This predicate writeClauses(MaxCost) generates the clauses that guarantee that
% a solution with cost at most MaxCost is found

writeClauses(infinite):- !, numNodes(N), writeClauses(N),!.
writeClauses(MaxColors):-
    eachNodeExactlyOnecolor(MaxColors),
    noAdjacentNodesWithSameColor(MaxColors),
    true,!.
writeClauses(_):- told, nl, write('writeClauses failed!'), nl,nl, halt.

eachNodeExactlyOnecolor(MaxColors):- 
    node(I), findall(x(I,C), between(1,MaxColors,C), Lits ), exactly(1,Lits), fail.
eachNodeExactlyOnecolor(_).

noAdjacentNodesWithSameColor(MaxColors):- 
    edge(I,J), between(1,MaxColors,C), writeOneClause([ -x(I,C), -x(J,C) ]), fail.
noAdjacentNodesWithSameColor(_).

%%%%%%%  4. This predicate computes the cost of a given solution M: ===========================

% Here the sort predicate is used to remove repeated elements of the list:
costOfThisSolution(M,Cost):- findall(C,member(x(_,C),M),L), sort(L,L1), length(L1,Cost), !.
```

Com podem veure, es tracta del problema de **min coloring**, en el que volem pintar els nodes de un graf sense que n'hi hagi dos adjacents del mateix color. Definim el **cost** de una solució com el nombre de colors necessaris diferents que necessitem per a complir les restriccions del problema. Aquí té molt sentit la existència del cost, ja que si no ens interesés la solució del problema podria ser sempre utilitzar un color diferent per a cada node.

La línea `costOfThisSolution(M,Cost):- findall(C,member(x(_,C),M),L), sort(L,L1), length(L1,Cost), !.` defineix el predicat de cost, que imposa que el cost de la solució es **Cost** sempre que la longitud de la llista sense repeticions (per això el `sort(L, L1`) dels colors utilitzats sigui de longitud **Cost**.

El **main**, que no esta aquí, busca solucions de cost descendent fins que el problema es torna insatisfactible, i retorna la ultima solució factible.

### Exemple: Factory

```prolog
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% We have a factory of concrete products (beams, walls, roofs) that
%% works permanently (168h/week).  Every week we plan our production
%% tasks for the following week.  For example, one task may be to produce
%% a concrete beam of a certain type, which takes 10 hours and requires
%% (always one single unit of) the following resources: platform, crane,
%% truck, mechanic, driver.  But there are only a limited amount of units
%% of each resource available. For example, we may have only 3 trucks.  We
%% have 168 hours (numbered from 1 to 168) for all tasks, but we want to
%% finish all tasks as soon as possible.
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

maxHour(168).

%% task( taskID, Duration, ListOFResourcesUsed ).
task(1,19,[1,2]).
task(2,52,[1,2]).
task(3,16,[1,3]).
task(4,52,[1,3]).
task(5,16,[2,3]).
task(6,20,[2,3]).
task(7,45,[2,3]).

%% resourceUnits( resourceID, NumUnitsAvailable ).
resourceUnits(1,2).
resourceUnits(2,1).
resourceUnits(3,2).

%%%%%%% Some helpful definitions to make the code cleaner: ====================================

task(T):-              task(T,_,_).
duration(T,D):-        task(T,D,_).
usesResource(T,R):-    task(T,_,L), member(R,L).

%%%%%%%  1. Declare SAT variables to be used: =================================================

satVariable( start(T,H) ):- task(T), integer(H).   % "task T starts at hour H"
satVariable( usage(T, H, R) ):- task(T), integer(H), integer(R). % task T uses resource R in hour H

%%%%%%%  2. Clause generation for the SAT solver: =============================================

% This predicate writeClauses(MaxCost) generates the clauses that guarantee that
% a solution with cost at most MaxCost is found

writeClauses(infinite):- !, maxHour(M), writeClauses(M),!.
writeClauses(MaxHours):-
    eachTaskStartsOnce(MaxHours),    % fa la restriccio de cost tambe   
    implicacioResources(MaxHours),
    resourcesMax,
    true,!.
writeClauses(_):- told, nl, write('writeClauses failed!'), nl,nl, halt.


horaIniciValida(T, H, MaxHours):-
        between(1, MaxHours, H),
        duration(T, D),
        End is H + D - 1,
        End =< MaxHours.


eachTaskStartsOnce(infinite):-!.
eachTaskStartsOnce(MaxHours):-
        task(T),
        findall(start(T, H), horaIniciValida(T, H, MaxHours), Lits), exactly(1, Lits),
        fail.
eachTaskStartsOnce(_).


% Implication between a task's start time and its resource usage throughout its duration
implicacioResources(MaxHours) :-
    task(T),
    horaIniciValida(T, Hini, MaxHours),
    duration(T, D),
    Hfi is Hini + D,
    between(Hini, Hfi, H),
    usesResource(T, R),
    writeOneClause([-start(T, Hini), usage(T, H, R)]),
    fail.
implicacioResources(_).

% Enforce constraints on resource usage within available limits per hour
resourcesMax :-
    resourceUnits(R, Units),
    maxHour(mH),
    between(1, maxHour, H),
    findall(usage(T, H, R), task(T), ResourceHourUsages),
    atMost(Units, ResourceHourUsages),
    fail.
resourcesMax.

%%%%%%%  4. This predicate computes the cost of a given solution M: ===========================

costOfThisSolution(M,Cost):- findall(End, ( member(start(T,H),M), duration(T,D), End is H+D-1), L),  max_list(L,Cost),!.
```

En aquest cas tenim un problema d'una fàbrica que fabrica tres tipus de productes diferents, i ho fa en tasques que utilitzen recursos limitats durant el temps en el que transcorren. La idea es definir una hora de inici valida per a cada tasca de tal manera que mai s'utilitzin més recursos dels que es tenen, **minimitzant el temps total**, definit com la hora de acabada de la ultima tasca. Les clausules defineixen en **SAT** les restriccions de consumició de recursos. Per a definir la **restricció de cost** tenim el següent.

```prolog
costOfThisSolution(M,Cost):- findall(End, ( member(start(T,H),M), duration(T,D), End is H+D-1), L),  max_list(L,Cost),!.
```

El que fem es trobar totes del hores de inici de les tasques i guardar aquest valor sumat de la seva duració. Després, el cost es definit per el màxim de tots els elements trobats. De la mateixa manera que en el **min coloring**, el programa *main* iterarà en **cost descendent** fins a trobar model **SAT** insatisfactible, i retornarà la última solució factible.

---

## Prolog avançat

Un dels problemes típics que trobarem serà el de trobar solucions a enunciats que segueixin la següent estructura, tenim:

+ un **Estat Inicial**, 

+ un **Estat Final** al que volem arribar

+ i un seguit de **pasos** que porten d'un estat a un altre, i que tenen un **cost**

L'objectiu es trobar un **camí**, o **conjunt de pasos**, que ens portin del **Estat Inicial** al **Estat Final**, amb el menor **cost** possible.

### Exemple: Problema dels ponts

```prolog
% Trata de averiguar la manera más rápida que tienen cuatro personas (P1, P2, P5 y P8)
% para cruzar de noche un puente que solo aguanta el peso de dos. Tienen una única e
% imprescindible linterna y cada Pi tarda i minutos en cruzar. Dos juntos tardan como
% el más lento de los dos.

main :- EstadoInicial = [[1, 2, 5, 8], [], 0], EstadoFinal = [[], [1, 2, 5, 8], 1],
    between(1, 1000, CosteMax), % Buscamos soluci ́on de coste 0; si no, de 1, etc.
    camino( CosteMax, EstadoInicial, EstadoFinal, [EstadoInicial], Camino ),
    reverse(Camino, Camino1), write(Camino1), write(' con coste '), write(CosteMax), nl, halt.

camino( 0, [[], _, _], _, C,C ).

camino( CosteMax, EstadoActual, EstadoFinal, CaminoHastaAhora, CaminoTotal ) :-
    CosteMax > 0,
    unPaso( CostePaso, EstadoActual, EstadoSiguiente ), % En B.1 y B.2, CostePaso es 1.
    \+ member( EstadoSiguiente, CaminoHastaAhora ),
    CosteMax1 is CosteMax-CostePaso,
    camino(CosteMax1, EstadoSiguiente, EstadoFinal, [EstadoSiguiente|CaminoHastaAhora], CaminoTotal).

maxim(X, Y, R) :-
    X > Y,
    R is X.
maxim(X, Y, R) :-
    X =< Y,
    R is Y.

% creuar el pont de esquerra a dreta, dues persones
unPaso(Tiempo, [E0, D0, 0], [E1, D1, 1]) :-
    select(P1, E0, RestaE0),
    select(P2, RestaE0, E1),    % agafem dos persones de la esquerra i E1 val el que hi queda l'esquerra
    union(D0, [P1,P2], D1),
    maxim(P1, P2, Tiempo).

% creuar el pont de dreta a esquerra
unPaso(Tiempo, [E0, D0, 1], [E1, D1, 0]) :-
    select(P1, D0, D1),   % torna algu [[],[2,5,8,1],1]]amb la llinterna
    union(E0, [P1], E1),
    Tiempo is P1.
```

En aquest problema, 4 persones necessiten creuar un pont amb les següents restriccions:

+ Només poden creuar si van amb la llanterna (només n'hi ha una)

+ Poden creuar com a màxim dos a la vegada

+ Cadascun te un temps de creuada, i si dos creuen junts, el temps es el del mes lent

El problema esta en quina es la manera mes rapida en que poden creuar tots el pont.

Com veiem en el codi, el *main*, que ens ve definit al examen, implementa la lògica per trobar el seguit de pasos amb menys cost, i nosaltres hem de implementar:

+ l' **Estat Inicial**: en aquest cas definim una llista de llistes on s'indica `[ llista_id_persones_esq_pont, llista_id_persones_drt_pont, posicio_lliterna(0->esq, 1->drt) ]`. Al principi, tant les persones com la llanterna estan a la banda esquerra del pont, o sigui que encara no ha creuat: `[ [1, 2, 4, 8], [], 0 ]`.

+ l' **Estat Final**: el nostre estat final serà aquell on tothom hagi creuat, i, per lògica, la llanterna estigui a la banda dreta del pont: `[ [], [1, 2, 4, 8], 1 ]`.

> Hem de veure que per motius de eficiència el `id_persona` es també el temps que tarden a creuar el pont: `1, 2, 4, 8`

Ara, només ens falta definir el predicat `unPaso` que pren tres àtoms:

+ **Tiempo**: expressa el cost d'aquell pas

+ **Estat Anterior**: Estat abans de aplicar el pas

+ **Estat Posterior**: Estat posterior

Bàsicament, el que estem dient es: anar d´aquest estat a aquest altre es un pas i te aquest cost si... i formules les condicions per a que o siguin. En l'exemple proposat, el pas de creuar el pont d'esquerra a dreta es compleix i te cost **Tiempo** si:

+ Han creuat **una** o **dues** persones com a molt.

+ La llanterna estava a l'esquerra i ara a la dreta (**0 => 1**). *Hem de veure que això ve donat en els estats en la definició del pas*.

+ **Tiempo** ha de ser el màxim dels `id_persona` que hagin creuat.

Si això es compleix, el que ha passat entre un estat i l'altre es un **pas**, i pot formar part de la **solució**.

> Per acabar, el pas de creuar de dreta esquerra es el mateix que d'esquerra a dreta, però no contempla la possibilitat de que creuïn dues persones ja que no te gaire sentit i mai serà part de la solució. També canvia que la llanterna ha de estar a la dreta en  el primer estat i a l'esquerra en el segon (**1 => 0**).

---

## CLP

Tenim un conjunt de **restriccions** , **variables** i **un domini per a cada variable**. **L'objectiu** es trobar valors per a les variables, cada un del domini corresponent, tal que es satisfacin les **restriccions**.

> SAT es un *Contraint Problem*, on les variables tenen totes el domini \{0,1\} i les restriccions son les clausules.

**Com s'han de resoldre?**

- :no_entry_sign: Generació i després test

- :white_check_mark: Entrellaçar la **generació** i el **test**

Per a millor la eficiència, podem provocar un efecte de **propagació**, **eliminant elements dels dominis de les variables** que sabem segur que no poden ser part de cap solució.

| Variables      | Constriccions |
| -------------- | ------------- |
| x in \{1..10\} | x < y         |
| y in \{1..8\}  | z = x + y + 1 |
| z in \{1..8\}  |               |

**Propagació inicial:**

- **x**: **1 2 3 4 5** ~~6 7 8 9 10~~

- **y**: ~~1~~ **2 3 4 5 6** ~~7 8~~

- **z**: ~~1 2 3~~ **4 5 6 7 8**

```prolog
:- use_module( library(clpfd) ).


P :- x in 1..10,                    % 1. Definim els dominis de les variables        
    [y, z] ins 1..8,                %

    x #< y,                         % 2. Definim les constriccions
    z #= x + y + 1,                 % 

    label ( [x, y, z]),             % 3. Resolem amb backtracking

    write ([x, y, z)), ln, halt.    % 4. Escriure la solucio
```

### Exemple: Magic

```prolog
:- use_module(library(clpfd)).

% Complete the following program p(N) that writes a (kind of) magic
% square: an NxN matrix with ALL the numbers 1..N^2, such that the
% sum of every row and every column is equal.
% More precisely, this sum is (N + N^3) / 2.
% Note: don't worry if your program is (too) slow when N >= 6.

%% Example:  (this solution is not unique):
%%
%%    1   2  13  24  25
%%    3   8  18  17  19
%%   16  14  15   9  11
%%   22  20   7  10   6
%%   23  21  12   5   4

main:- p(5), nl, halt.

p(N):-

% 1.
    NSquare is N*N,
    length( Vars, NSquare ),

    Vars ins 1..NSquare,

% 2. 
    all_distinct(Vars),
    squareByRows(N,Vars,SquareByRows),
    transpose( SquareByRows, SquareByCols ),  % transpose already exists: no need to implement it

    Sum is (N + N*N*N) // 2, 
    constraintsSum( Sum, SquareByRows),
    constraintsSum( Sum, SquareByCols),

% 3.
    label(Vars),

% 4.
    writeSquare(SquareByRows),nl,!.


squareByRows(_,[],[]):-!.
squareByRows(N,Vars,[Row|SquareByRows]):- append(Row,Vars1,Vars), length(Row,N), squareByRows(N,Vars1,SquareByRows),!.

writeSquare(Square):- member(Row,Square), nl, member(N,Row), write4(N), fail.
writeSquare(_).

write4(N):- N<10,   write('   '), write(N),!.
write4(N):- N<100,  write('  ' ), write(N),!.
write4(N):-         write(' '  ), write(N),!.

exprSuma([X], X).
exprSuma([X|L], X + Expr) :- exprSuma(L, Expr).

constraintsSum( _, []).
constraintsSum( Sum, [CR | CRL]) :- exprSuma(CR, Suma), Suma #= Sum, constraintsSum(Sum, CRL).
```

En aquest problema hem de posar tots el nombres entre **1** i **N^2** en un quadrat de **N x N** complint les següents restriccions:

+ han de estar tots els nombres, i no hi pot haver repetits òbviament,

+ cada fila i columna ha de sumar la mateixa quantitat que és: **(N + N\*N\*N) / 2**.

Per a fer-ho seguim la següent estructura:

1. **Definim el domini de les variables**: Ho fem en la línea: `Vars ins 1..NSquare` 

2. **Definim les constriccions**: La primera restricció es la de `all_distinct(Vars)` que imposa que cap variable pugui tenir el mateix valor, i elimina els valor dels dominis en cas que sigui possible. Després, ens ajudem dels predicats `squareByRows` i `transpose` per obtenim la matriu en llistes de llistes, les quals recorrem amb el predicat `contraintsSum` i imposem que l'expressió que suma tots els elements de cada fila i columna sigui igual (`#=`) a el valor que ens diu l'enunciat. `exprSuma` retorna una expressió que representa la suma dels valors d'una llista.

3. **Fem el `label`**

4. **Imprimim el resultat**

---

## Altres exercicis d'examens passats

**politician.pl**

```prolog
%% Elections are coming!  A politician wants to do a road trip starting and finishing at city 1, driving in
%% total at most MaxKm kilometers, and visiting at least N cities (city 1 and N-1 different other ones),
%% without passing twice through the same city!  Complete the following predicate politician(N,MaxKm,Trip),


% road(A-B,K) means there is a road from A to B (or vice versa) of length K km.
road( 1-3, 3  ).
road( 1-6, 25 ).
road( 2-3, 41 ).
road( 2-4, 31 ).
road( 2-5, 7  ).
road( 2-6, 7  ).
road( 3-4, 32 ).
road( 4-7, 14 ).
road( 4-8, 29 ).
road( 5-6, 36 ).
road( 5-7, 45 ).
road( 5-8, 22 ).
road( 6-8, 11 ).
road( 7-8, 44 ).

road1(A-B,K):- road(A-B,K).
road1(B-A,K):- road(A-B,K).


politician(N,MaxKm,Trip):-     path(N, MaxKm, 1, [], Trip).

% path( NumCitiesRemainingToBeVisited, RemainingKm, CurrentCity, CitiesAlreadyVisited, Path )

path( 0, _, 1, _, [] ):- !.
path( NumCitiesRemainingToBeVisited, RemainingKm, CurrentCity, CitiesAlreadyVisited, [CurrentCity1|Path] ):-
    road1( CurrentCity-CurrentCity1, K ),
    \+ member(CurrentCity1, CitiesAlreadyVisited),
    NewN is NumCitiesRemainingToBeVisited - 1,
    NewRmKm is RemainingKm - K,
    NewRmKm >= 0,
    path( NewN, NewRmKm, CurrentCity1, [CurrentCity1|CitiesAlreadyVisited], Path).

%% Examples: this main writes the six trips below (in some order):

main:- politician(5,100,Trip),   write([1|Trip]), nl, fail.
main:- politician(6,120,Trip),   write([1|Trip]), nl, fail.
main:- halt.

%% [1,3,4,8,6,1]
%% [1,3,4,2,6,1]
%% [1,6,8,4,3,1]
%% [1,6,2,4,3,1]
%% [1,3,2,5,8,6,1]
%% [1,6,8,5,2,3,1]
```

> :warning: **És molt important en *PROLOG* no oblidar mai que no estem programant imperativament, sino que estem definint predicats, i que ha de passar per a que siguin certs**
