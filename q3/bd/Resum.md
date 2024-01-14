# Resum BD

## Algebra relacional

#### Unió

`R = T_u_S`

S'uneixen les taules. Els atributs han de coincidir.

#### Reanomenament

`R = S{atr1 -> atr2}`

Es canvia el nom del/s atribut/s de la taula. El contingut no canvia.

#### Intersecció

`R = T_n_S`

El resultat són les entrades que pertanyen tant a una taula com a l'altra. Cal que els atributs coincideixin.

#### Diferència

`R = T - S`

El resultat són les entrades que pertanyen a T i que no pertanyen a S. Cal que els atributs coincideixin.

#### Producte Cartesià

`R = T_x_S`

Els atributs del resultat son tots els de T i els de S. Per a cada entrada de T hi ha |S| entrades al resultat amb valor T + S$_i$. Si algun nom d'atribut de T i S coicideixen no es pot fer el producte cartesia.

#### Seleció

`R = T(atr1 = 'exemple' and atr2 > 2)`

El resultat conté les entrades que compleixen les condicions.

#### Projecció

`R = A[atr1, atr3]`

El resultat és una relació amb només alguns atributs de A. Les entrades són les mateixes.

#### Join

`R = A[atr_a1 = atr_b1]B`

El resultat és una relació amb els atributs de A  i de B on les entrades queden relacionades per la condició o condicions.

També es pot fer `R = A[atr_a1*atr_b1]B`, i l'esquema del resultat només tindrà un dels atributs de la condició, ja que a efectes practics tindran el mateix valor.

També es pot fer `R = A*B` i es compararan els atributs amb mateix nom de les dues relacions

## Assercions

Les assercions són restriccions de integritat que afecten a **més de una taula.** A diferència de les restriccions de columna, que només es comproven en consultes *insert* i *update*, aquestes es comproven constantment.

```plsql
CREATE ASSERTION nom CHECK (condició)
```

###### Exemple

```plsql
empleat(nemp, ciutat_e, ndept)
dept(ndept, ciutat_d)

CREATE ASSERTION ciutat_emp_dept CHECK
    (NOT EXISTS (SELECT *
                 FROM empleat e, dept d,
                 WHERE e.ndept = d.ndept and
                 e.ciutat_e <> d.ciutat_d)
```

## Vistes

Una vista és una relació derivada d'altres relacions. És com mirar una taula desde un punt de vista que ens interesa.

#### Exemple

```sql
empleat (dni, nom, sou)

CREATE VIEW empleats_rics AS SELECT dni, nom 
    FROM persona WHERE sou > 100
```

Aquí creem una vista de la relació empleat, que tindra dues de les columnes de la relació, i a més aplicarem un WHERE per a seleccionar els empleats que ens interesen.

#### Consultes a una vista

Les vistes són, igual que les relacions, consultables (SELECT, INSERT, UPDATE, DELETE). El SELECT funciona de manera normal, pero les consultes que modifiquen la vista tenen certes restriccions que s'han de complir. Una vista és **actualitzable** si ha sigut definida per:

+ SELECT sobre una **única** relació (o vista actualitzable), sense agregats ni **DISTINCT**. (si tenim una entrada a la vista pero que refereix a dues o més en la relació a la que fa referència no podem distingir quin registre s'ha de actualitzar)

+ Els atributs del SELECT han d'incloure tots els atributs **NOT NULL** que no tinguin valor per defecte de la relació original. (sino podriem fer un insert valid per a la vista, i a l'hora de modificar la relació no posariem valor a atributs amb restricció NOT NULL)

+ Sense **GROUP BY**. (si tenim una entrada a la vista pero que refereix a un conjunt de entrades en la relació a la que fa referència no podem distingir quin registre s'ha de actualitzar)

En resum no s'admeten les consultes que provoquin **ambigüitat** a l'hora de modificar la relació a la que fa referencia la vista.

> Les consultes a una vista és fan de la mateixa manera que a una relació. Hi poden haver vistes de altres vistes.

## Privilegis

Quan una sessió es comença amb una connexió tenim la oportunitat de indicar l'usuari:

```sql
CONNECT TO nom_servidor AS nom_conexió USER nom_usuari
```

Les consultes SQL que fem durant la sessió només es duran a terme si tenim els permisos per a realitzar-les. **Els privilegis són una manera de definir quines accions pot dur a terme un usuari sobre la base de dades**. Hi ha 9 tipus diferents de privilegis, els importants per a la assignatura són quatre:

+ SELECT

+ INSERT

+ UPDATE

+ DELETE

#### Com donar privilegis: GRANT i REVOKE

Quan creem una BD ens convertim en el DBA (**Data Base Administrator**) i tenim tots el privilegis sobre una base de dades. Per a donar privilegis a un altre usuari utilitzem la sentència **GRANT**.

```sql
GRANT privilegis ON objectes TO usuaris [WITH GRANT OPTION]
```

Els privilegis poden ser:

+ select, select (columnes específiques)

+ update, update (columnes específiques)

+ insert

+ delete

Els objectes de moment són taules i vistes.

Els usuaris són es que volguem, separants per comes si n'hi ha més d'un.

Si s'utilitza la opció **WITH GRANT OPTION** aquell usuari que rep els permisos també es capaç de donarlos a altres usuaris (només els que especifica la sentència)

Per a treure privilegis a un usuari utilitzem la sentència **REVOKE**.

```sql
REVOKE [GRANT OPTION FOR] privilegis ON objectes FROM usuaris {CASCADE|RESTRICT}
```

Els privilegis, objectes i usuaris funciones igual que en el GRANT. Si utilitzem la opció **GRANT OPTION FOR** no treiem el privilegi sino la **capacitat de donar-lo a altres usuaris.**

+ **CASCADE:** Es treu el privilegi a l'usuari i a aquells usuaris als que hagi autoritzat aquest (si tenen el privilegi per una altra via no).

+ **RESTRICT:** No es realitza el REVOKE si això implica retira el privilegi a un altre usuari (al que el usuari al que li volem treure hagues autoritzat). 

#### Rols

A vegades es molt costos especificar els privilegis per a cada usuari. Per això tenim els rols. Un rol és una agrupació de privilegis que podem aplicar a un usuari, és a dir, el usuari té un rol. En canviar els privilegis del rol es canvien automàticament els privilegis de tots els usuaris amb aquell rol.

```sql
-- El DBA crea dos rols, el de lector i el de escriptor
CREATE ROLE lector
GRANT select on llibres to lector
CREATE ROLE escriptor
GRANT insert on llibres to escriptor 

-- Ara assignem al usuari MARC els privilegis del rol lector
GRANT lector to MARC

-- Ara assignem al usuari JOAN els privilegis del rol lector i escriptor
GRANT lector to JOAN
GRANT escriptor to JOAN

-- Si el DBA decideix que els escriptors també podran modificar (update) 
-- la taula *llibres*, només ha de canviar els privilegis del rol escriptor
GRANT update ON llibres TO escriptor


-- Ara el DBA decideix que un lector també ha de poder donar els permisos
-- de lector a un altre usuari només cal que faci
GRANT select ON llibres to lector WITH GRANT OPTION

-- EL DBA també pot decidir treure privilegis a un rol
REVOKE insert ON llibres FROM escriptor
```

Per a que un usuari utilitzi el rol al que té permis ha de realitzar la seguent consulta dins d'una sessió:

```sql
SET ROLE lector/escriptor -- depen de quin permis tingui


-- A més, si el rol se li assigna amb WITH GRANT OPTION el pot donar
-- a un altre usuari
GRANT lector TO EDUARD -- Ara l'usuari EDUARD tambés té el rol de lector
```
