# Resum C2 AC

## 1. Problemes de cache

### 1. 1. Write-through & write-no-allocate

Si tenim aquesta cache es normal que ens donguin les seguents dades:

+ **tsa**: temps en cas d'encert

+ **m**: *miss rate*

+ temps que tardem a escriure una **paraula** a MP

+ temps que tardem a llegir/escriure un **bloc** a MP

+ percentatge **escriptures/lectures**

Per calcular el **tma** (temps mig d'accés) hem de tenir en compte que quan tenim penalització només quan tenim un fallo de lectura. En les escriptures, escribim una **paraula**, per això en donen dos temps. Llavors sabem que:

`tma = tsa + percentatge_escriptures*temps_MP_paraula + m*percentatge_lectures*temps_MP_bloc`

Es a dir, el **tsa** base d'accedir a la cache, el percentatge d'escritures que tenim mutiplicat per el temps que ens costa fer una escriptura. A més, en cas de **fallo de lectura**, mutipliquem per el temps que ens costa resoldre-la (llegir el bloc de MP).

### 1. 2. Copy back & write allocate

Si tenim aquesta cache es normal que ens donguin les seguents dades:

+ **tsa**: temps en cas d'encert

+ **m**: *miss rate*

+ temps que tardem a llegir/escriure un **bloc** a MP

+ percentatge de **blocs modificats**

Per calcular el **tma** (temps mig d'accés) hem de tenir en compte que en cas de fallo de cache, independentment de escritura o lectura, **llegirem de MP** i, en cas de haber de substituir un bloc modificat, **escriurem a MP**.

`tma = tsa + m * (temps_MP_bloc + percentatge_blocs_modificats * temps_MP_bloc)`

---

## 2. Problemes de DRAM (DDR)

En aquests problemes que ens demanen normalment cronogrames de accessos secuencials a memoria principal (DRAM) solem tenir les seguents dades:

+ Latencia fila (suposem 2 cicles)

+ Latència columna (suposem 3 cicles)

+ Latència de *precharge* (suposem 2 cicle)

Suposem blocs de memoria principals de **32 bytes** i **8 chips de 1 byte**. Per tant, amb **DDR** (Double Data Rate), tardarem **2 cicles** a llegir un bloc de MP.

> Si tenim DDR vol dir que podem enviar dos dades en un mateix temps de cicle, si no en tenim simplement tardarem el doble

### 2. 1. Mateix banc mateixa pàgina

Si accedim a dos blocs de memoria que es troben en el mateix banc i mateixa pàgina hem de saber que això vol dir "baixar" una sola fila del mateix banc. Això significa que no poden ser en paralel, pero només necessitarem fer un *precharge*. Imaginem que accedim a dos blocs consecutius de memoria (D[0] -> D[7]). Per tant, farem:

| 0    | 1   | 2    | 3   | 4   | 5          | 6          | 7   | 8          | 9          | 10  | 11  | 12  |
| ---- | --- | ---- | --- | --- | ---------- | ---------- | --- | ---------- | ---------- | --- | --- | --- |
| ACT  | --  | RD   | --  | --  | RD         | --         | --  |            |            | PRE | --  |     |
| @Fil |     | @Col |     |     | @Col       |            |     |            |            |     |     |     |
|      |     |      |     |     | D[0], D[1] | D[2], D[3] |     | D[4], D[5] | D[6], D[7] |     |     |     |

### 2. 2. Mateix banc diferent pagina

Si accedim a dos blocs de memoria que es troben en el mateix banc i en diferents pàgines hem de saber que això vol dir "baixar" dues files del mateix banc, cosa que suposa no poder-ho fer en paralel, i que entre mig dels dos accessos haurem de fer *precharge* de l primera fila que baixem. Per tant farem

| 0    | 1   | 2    | 3   | 4   | 5          | 6          | 7   | 8   | 9    | 10  | 11   | 12  | 13  | 14         | 15           | 16  | 17  |
| ---- | --- | ---- | --- | --- | ---------- | ---------- | --- | --- | ---- | --- | ---- | --- | --- | ---------- | ------------ | --- | --- |
| ACT  | --  | RD   | --  | --  |            |            | PRE | --  | ACT  | --  | RD   | --  | --  |            |              | PRE | --  |
| @FIL |     | @COL |     |     |            |            |     |     | @FIL |     | @COL |     |     |            |              |     |     |
|      |     |      |     |     | D[0], D[1] | D[2], D[3] |     |     |      |     |      |     |     | D[8], D[9] | D[10], D[11] |     |     |

### 2. 3. Diferent banc (mateixa/diferent pagina)

Si accedim a dos blocs de memoria que es troben en diferents bancs hem de saber que estem "baixant" dues files de diferents bancs, que ho podem fer en paralel i que haurem de fer *precharge* de les dues. La ventaja es que podem aprofitar millor el temps, enviant les senyals perque el temps quadri i sigui el menor possible. Hem de tenir en compte que mentre i hagi latencia no podem enviar cap més senyal, i que el bus de dades es compartit, es a dir, no podem enviar dades de diferents bancs a la vegada.

| 0    | 1   | 2    | 3    | 4   | 5          | 6          | 7   | 8              | 9              | 10  | 11  |
| ---- | --- | ---- | ---- | --- | ---------- | ---------- | --- | -------------- | -------------- | --- | --- |
| ACT  | --  | RD   | ACT  | --  | @RD        | --         | --  | PRE            | --             | PRE | --  |
| @FIL |     | @COL | @FIL |     | @COL       |            |     |                |                |     |     |
|      |     |      |      |     | D[0], D[1] | D[2], D[3] |     | D[128], D[129] | D[130], D[131] |     |     |

---

## 3. TLB

En els problemes de TLB, ens solen posar un bucle (codi màquina o c++) i hem de dir cantitat d'encerts i fallos que ha tingut. Ens interesens diferents dades:

+ **tamany pagina**: amb això sabrem quants elements caben en una pàgina

+ **tamany TLB**: interessa per que a vegades el que sembla un encert ja no ho és perque s'ha sobrescrit la entrada per una altra

+ **algorisme reemplaçament TLB**: sol ser LRU (Least Recently Used), el que porta més temps sense ser accedit el substituim

Normalment resoldrem el problema observant només que fa el codi en en les primeres iteracions del bucle. Hem de buscar factors com:

+ frequència en que cada accés solicita una nova pàgina

+ substitucions en la TLB, que permeten saber el nombre de cops que fallarà la TLB.

Normalment el comportament és **cíclic**, i podem saber el nombre de fallades de la seguent manera.

`sumatori_de_fallades_primers_cicles + (nombre_cicles - primers_cicles) * fallades_per_cicle`

> Hem de tenir en compte que si ens donen el tamany de pàgina en una unitat i la volem convertir a una altra, al estar tractant en temes de memoria, hem de utilitzar la conversió per 1024. (**1KByte = 1024 bytes**, per exemple).
> 
> Normalment no hem de tenir en compte una entrada de la TLB per la pàgina on estan les instruccions, ni que aquestes puguin canviar de pàgina a meitat del bucle. Si no ens diuen res ho podem obviar.

---

## 4. Teoria de cache

### 4. 1. Directa

La cache directa és aquella que cada bloc de MP té associat una única línea de la memoria cache on hi pot anar. Suposem les seguents dades:

+ **tamany cache** = 8 Kbytes

+ **tamany bloc cache** = 32 bytes

+ **direccions de 16 bits**

Aixo vol dir que si tenim 8Kbytes a repartir entre linees de 32 bytes tindrem:

`num_linees = tamany_cache / tamany_bloc_cache`

> Tenim en compte que 8KBytes son 8*1024 bytes.

Llavors sabem que tenim **256 linees**. Per saber a quina linea va cada bloc de MP necessitem `log_2 (256) =  8 bits`. A més, com que cada linea té 32 bytes, necessitarem `log_2(32) = 5 bits`. Per tant ens resten `32 - 8 - 5 = 19 bits TAG`. La distribució del bits de l'adreça quedaria així.

| TAG     | #linea | Byte   |
| ------- | ------ | ------ |
| 19 bits | 8 bits | 5 bits |

Al accedir a la MC, és mira a quina linea hauria de estar el bloc (8 bits de linea), es compara el TAG amb el TAG al que volem accedir, i, si coincideix, seleccionem el byte que volem (5 bits de byte). 

### 4. 2. n-Associativa (2-Associativa)

La cache n-Associativa és com la directa amb una diferència: un cop sabem a quina linea, ara li direm **conjunt**, de la memoria cache va un bloc de MP, tenim més de una **via** on el podem deixar. Suposem les seguents dades:

+ **tamany cache** = 8 KBytes

+ **tamany bloc cache** = 32 Bytes

+ **direccions de 32 bits**

+ **2-Associativa**

Per saber el nombre de **conjunts** que tindrà la MC hem de pensar que els 8 KBytes estaràn repartits en **n** conjunts, i cada conjunt tindrà **dos bloc de 32 Bytes**.

`numero_conjunts = tamany_cache / (32 Bytes/bloc * 2 blocs/conjunt) = 8*1024 / (32 * 2) = 128 conjunts`

> Fixem-nos que si l'associativitat és una altra només hem de canviar: `n blocs/conjunt`

Per a especificar el conjunt necessitem `log_2(128) = 7 bits`. Per seleccionar el byte necessitem `log_2(32) = 5 bits`

| TAG     | #conjunt | Byte   |
| ------- | -------- | ------ |
| 20 bits | 7 bits   | 5 bits |

Quan accedim a la cache, mirem primer a quin conjunt hauriem de trobar l'adreça (7 bytes #conjunut), i després comprovem les 2 TAG de les 2 vies per a veure si tenim el bloc a la cache (o n si tenim una altra associativitat). Després seleccionem el byte (5 bits menor pes).

> Hem de tenir en compte el algorisme de reemlaçament, es a dir, si un bloc va a un conjunt on les dues vies son plenes, a on el posem? L'algorisme LRU (*Least Recently Used*) surt bastant i substitueix aquell bloc al que fa més estona que no s'accedeix.

#### 4. 2. 1. Predictor de via

El predictor de via és un dispositiu (hardware) que preedeix la via a la que accedirem, es a dir, només comprovem una TAG (la de la via que predeix). En cas que no trobem el bloc mirem les altres vies. Això millora el **tma** de la cache. Per a calcular-lo soler tenir les seguents dades:

+ **percentage d'encerts predictor**

+ **cost d'accés a una via**

Si tenim una 2-associativa tenim que el **tma** es veu reduit en `percentatge_encerts_predictor * cost_acces_via`, ja que en cas que el predictor encerti ja no fem accés a l'altre via. 

> En cas que ens parlin de **energia** hem de tenir en compte que també ens estalvia energia, ja que no accedim a l'altre via. Per tant l'energia estalviada és: `percentatge_encerts_predictor * energia_acces_via`

### 4. 3. Completament associativa

La cache completament associativa és aquella en que un bloc de memoria pot anar a qualsevol línea de la cache. Per tant, tenim que el nombre de linees ve donat per: `num_linees = tamany_cache / tamany_bloc`

Només necessitem `log_n(tamany_bloc)` bits per a codificar el byte que volem seleccionar, i els demés bits són de TAG.

---

## 5. RAIDS

Suposem que:

+ n = nombre de discos

+ banda = ample de banda de 1 disc

|            | % mem. útil | fallos permesos | descripció                                                                                                                              | Banda escriptura aleatoria                                | Banda escriptura sequencial |
| ---------- | ----------- | --------------- | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | --------------------------- |
| **RAID 0** | 100 %       | Cap             | Es tracta de separar les dades en discos repartint els bytes                                                                            | `n * banda`                                               | `n * banda`                 |
| **RAID 1** | 50 %        | 1               | Es tracta de tenir dos discos amb la mateixa informació                                                                                 | `(n/2) * banda`, hem de escriure als dos discos el mateix | `n/2 * banda`               |
| **RAID 3** | `(n-1)/n`   | 1               | Es tracta de tenir un disc de paritat. Entrellaçat a nivell de byte. Coll d'ampolla disc de paritat.                                    | `n * banda`, s'ha de escriure a paritat                   | `(n - 1) * banda`           |
| **RAID 4** | `(n-1)/n`   | 1               | Es tracta de tenir un disc de paritat. Entrellaçat a nivell de tira. Coll d'amplolla disc de paritat.                                   | `n * banda`, s'ha de escriure a paritat                   | `(n - 1) * banda`           |
| **RAID 5** | `(n-1)/n`   | 1               | Es tracta de repartir la paritat de cada tira en un disc diferent, així que ja no hi ha coll d'ampolla.                                 | `n * bEsc / 4`                                            | `(n-1) * banda`             |
| **RAID 6** | `(n-2)/n`   | 2               | Es tracta de tenir dos paritats per cada tira, repartides entre els discos (les paritats esta en un disc diferent en funció de la tira) | `n * bEsc / 6`                                            | `(n-2) * banda`             |

#### 5. 1. Multi-RAID

Els sistemes multi-RAID son uns discos lògics que utilitzen més d'un tipus de raid. Tenen la forma **RAID xy**, i es composen de discos organitzats en **RAID x**, que son vistos com discos físics per un **RAID y**. N'hi ha principalment 6 (que entrin als examens):

+ RAID 01 i RAID 10

+ RAID 50 i RAID 05

+ RAID 15 i RAID 51

### 5. 2 Exemple problema de RAIDs

> *Disponemos de 60 discos físicos de 300 Gbytes de capacidad por disco, que ofrecen un ancho de banda efectivo de 100 Mbytes/s por disco. Con estos discos deseamos montar un disco lógico en donde consideramos las siguientes 4 opciones:*
> 
> + *RAID 6*
> 
> + *RAID 10 (mirror doble con 30 grupos de 2 discos)*
> 
> + *RAID 50 (con 6 grupos de 10 discos)*
> 
> + *RAID 51 (mirror doble con 2 grupos de 30 discos)*
> 
> **Calcular la cantidad de información útil que puede almacenar cada uno de los RAIDS considerados i el ancho de banda efectivo si hacemos lecturas sequenciales o aleatorias**

###### RAID 6:

La cantitat de memòria útil d'un RAID 6 ve donada per `(n-2) / n` ja que dels *n* discos que té sabem que dos aniran destinats a paritat (no dos discos sencers, sino el tamany de dos discos repartits en diferents discos en funció de la tira). Per tant tindrà una memòria útil del `58/60 * 100 = 96.7%` i en tamany serà `60 discos * 300 GBytes/disc * 96.7% = 17.406 Gbytes útils`. 

+ lectures sequencials: si son sequencials llegim tires senceres. Per tant: `banda_lectura = n * 100Mbytes/s = 6000MBytes/s`

+ lectures aleatories: L' ample de banda de lectura, serà de `n * bLec`, ja que podem llegir en paral·lel de tots els discos , per tant: `banda_lectura = n * 100MBytes/s = 60 * 100 = 6.000 MBytes/s`. 

+ escriptures sequencials: si escribim sequencial, escribim dades en els *n-2* discos que pertoqui i la paritat de les noves dades als dos altres discos. Per tant: `ample_escriptura = (n-2) * 100Mbytes/s = 5800Mbytes/s`

+ escriptures aleatories: si en canvi son aleatories, per cada dada que escribim, n'hem de llegir la antiga i les dos paritats de la tira, i escriure la nova dada i les dos paritats. Es a dir, fem 3 lectures i 3 escripture, cosa que redueix l'ample de banda en 6. Per tant: `ample_escriptura = n * (100Mbytes/s / 6) = 1000Mbytes/s`

###### RAID 10 (mirror doble con 30 grupos de 2 discos):

El RAID 10 està format per discos logics estructurats en RAID 1, es a dir 30 grups de 2 discos. Com que cada grup d' aquests té un memòria útil del 50%, ja que estan repetides les dades, podem veure que el tamany útil serà: `tamany_util = 60 * 300Gbytes/disc * 50% = 9000Gbytes`.

+ lectures sequencials: quan llegim sequencialment, el RAID 0 buscara cada part de la tira en un grup de RAID 1 diferent, on podrem llegir de qualsevol dels dos, pero no el doble de rapid. Per tant, cada grup tindrà un ample de banda de lectura de `100Mbytes/s`. Pero, hem de tenir en compte que l'enunciat ens diu que sempre tenim peticions pendents, per tant podrem resoltdre en paralel dues lectures sequencials, distribuint-les en els dos discos de cada RAID 1. El total serà de `200Mbytes/grup * 30grups = 6000Mbytes/s`

+ lectures aleatories: si son aleatories, podem llegir dos trosos de informació que estiguin al mateix grup, i llegir un del primer disc i l'altre de la seva copia. Per aixo el ample de banda de lectura de cada grup es de `100Mbytes/s * 2discos = 200Mbytes/s`, i el total serà de `200 Mbytes/s * 30 = 6000Mbytes/s`

+ escriptures sequencials: quan escribim sequencialment, per a cada grup hem de escriure dos cops, per tant l' ample de banda del grup és el de un disc `100Mbytes/s`, iel total de `100Mbytes/s * 30 = 3000Mbytes/s`

+ escriptures aleatories: és igual que les sequencials, ja que sempre haurem de escriure la dada als dos discos d'un mateix grup de RAID 1

###### RAID 50 (con 6 grupos de 10 discos):

Aquest RAID 50, está format per 6 grups de RAIDs 5 amb 10 discos cada un. Això vol dir que per cada tros d'informació que el RAID 0 emmagatzema a un grup, aquest es distribueix en 9 dels discos del grup, i en el altre es guarden les paritats (en diferents discos en funció de la tira). Per tant, la memoria útil serà de `59/60 * 100 = 98.4%`. El tamany útil serà de `60discos * 300Gbytes/disc * 98.4% =  17.700GBytes`

+ lectures sequencials: si les lectures son sequencials, es buscara un tros de la informació a cada grup de RAID 5. Cada grup té una velocitat de lectura de tira de `10*100Mbytes/s = 1000Mbytes/s`. Per tant, el total es de `1000Mbytes/s * 6 grups = 6000Mbytes/s`

+ lectures aleatories: si son aleatories podem llegir dades de tots els discos a la vegada, ja que distribuim les peticions de lectura per a que no intentem llegir dades de la mateixa tira: `ample_banda_lectura = 60 * 100Mbytes/ = 6000Mbytes/s`

+ escriptures sequencials: per a cada grup hem de escriure dades als 9 discos i al de paritat (distribuit en funció de la tira), per tant tenim un ample de banda efectiu de `9 * 100Mbytes/s = 900Mbytes/s`. En total: `900Mbytes/s * 6 grups = 5400Mbytes/s`

+ escriptures aleatories: si son aleatories, per escriure cada dada hem de llegir al partitat de la tira i la dada entiga i encriure la nova dada i la nova paritat, es a dir, 2 lectures i 2 escritures. Per tant l'ample de banda es redueix en 4: `ample banda = 60 * (100Mbytes/s / 4) = 1500Mbytes/s`

###### RAID 51 (mirror doble con 2 grupos de 30 discos):

Aquest RAID 51, està format per 2 RAIDs 5 idèntics, on les dades estàn duplicades. En cada RAID 5 tenim 30 dicos, dels quals 29 estaran dedicats a dades i l'altre a paritat (distribuit entre discos). Per tant la memòria ùtil de cada RAID 5 serà: `29/30 * 100 = 0.97%`. La memòria útil del RAID 1 serà del `50%` ja que les dades estan duplicades. Mutiplicant aquests percentatges trobem la memòria útil total: `97% * 50% = 48%`. Per tant el tamany serà de `60 discos * 300Gbytes/dis * 48% = 8700Gbytes`.

+ lectures sequencials: si les lectures son sequencials, només aprofitem un dels grups, ja que hem de llegir les mateixes dades (mateixa tira), i ens serveix llegir le qualsevol dels dos grups de RAID 5, ja que estan duplicats. Cada grup de RAID te un ample de banda sequencial de `30*100Mbytes/s = 3000Mbytes/s`. Per tant l' ample de banda de lectura es el de un grup: `3000Mbytes/s`. Com que sempre tenim peticions pendents de resoldre, podem resoldre una a el primer de RAID 5, i l'altre a el segon grup. Per tant: `ample_banda = 3000Mbytes/s * 2 = 6000Mbytes`

+ lectures aleatories: si les lectures son aleatories, les podem distribuir entre els dos grups de RAID 5 aquests entre els diferents discos. Per tant: `ample_banda = 60 * 100Mbytes/s = 6000Mbytes/s`.

+ escriptures sequencial: hem de tenir en compte que l'ample de banda serà el de un grup de RAID 5, ja que en haurem de escriure en en els dos. Per escriure en un RAID 5 hem de fer 29 escritures de dades i una de paritat, per tant escribim dades a `29 * 100Mbytes/s = 29000Mbytes/s`

+ escriptures aleatories: si son aleatories, per cada dada que escribim hem de llegir la paritat antiga i la dada antiga i escriure la nova paritat i la nova dada. Es a dir 2 lectures i 2 escriptures. I, a més fer el mateix a l'altre disc. Per tant: `ample_banda_escriptura = 30 * 100Mbytes/s / 4 = 750Mbytes/s`

## . Formules:

`potencia_dinamica = Carga_Capacitiva(F) * Voltatge^2(V) * Frequència(Hz)`

`potencia_estàtica = Voltage(V) * Intesitat_Fuga(A)`
