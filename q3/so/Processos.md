# Processos

## Per a que volem processos?

Els sistemes operatius que hi ha actualment permeten l'execucio de molts programes i funcionalitats a la vegada. Però, el codi que defineix que fa cada un es diferent, i es troba en llocs diferents de memoria. Es per aixo, i per altres complicacions més, que es convenient diferenciar cada tasca que faci l'ordinador. Un proces és una esctructura de diferents dades que simbolitza una tasca a l'ordinador: 

+ PID: identificador únic del procés (unicitat temporal)

+ Regions de memoria que utilitza

+ Usuari que l'executa (permisos)

+ Imatge del codi que executa

+ Instant en el que ha començat

+ etc...

---

## Com gestionem els processos?

Un programa d'ordinador no és més que un seguit instruccions emmagatzemades a memoria que l'ordinador sap executar. A més, en un computador, la cantitat d'instruccions que es poden realitzar a la vegada queda limitada per el nombre de CPU que tingui aquest., per tant és el sistema operatiu qui decideix quin proces utilitza la CPU i quin no en cada moment. Hi ha certs problemes que el sistema operatiu ha de gestionar per a que diferents programes utilitzin una mateixa CPU:

+ en canviar el proces el **PC** ha canvia (el codi es troba a un lloc diferent de memoria)

+ en canviar el proces les dades de la CPU poden ser modificades per un altre proces 

Per resoldre aquest problema definim una estructura de dades que contindra tota la informació de un proces necessaria per a que quan torni a utilitzar la CPU poguem recuperar les dades que tenia abans de deixar de utilitzarla: **PCB (Process Control Block)**. Les seves dades son de dos tipus:

+ **Espai d'adreçament:** conté les descripcions de els espai de memòria que utilitza el process (codi, dades, pila)

+ **Context d'execució**: dades referents a l'estat de la CPU en el moment en el que s'executa el process (utilitza CPU)

---

## Syscalls

| Syscall         | Description                                     | Header files          | Get more info   |
| --------------- | ----------------------------------------------- | --------------------- | --------------- |
| `fork`          | Crea un process fill                            | `unistd.h`            | `man 2 fork`    |
| `exec (execlp)` | Canvia el codi d'execució                       | `unistd.h`            | `man execlp`    |
| `exit`          | Finallitza un process                           | `unistd.h`/`stdlib.h` | `man 2 exit`    |
| `wait/waitpid`  | Espera a que finalitzi un process fill          | `sys/wait.h`          | `man 2 waitpid` |
| `getpid`        | Retorna el PID (Process Identifier) del process | `unistd.h`            | `man 2 getpid`  |
| `getppid`       | Retorna el PID del process pare                 | `unistd.h`            | `man 2 getppid` |

### Fork

```c
int fork();
```

Crea un process fill identic al pare amb certes característiques

+ el fill té el seu propi **PID**

+ si el fill executa `getppid()`  se li retorna el **PID** del pare

+ el temps d'execució del fill es 0 després del `fork()`

+ el fill no té senyals pendents després del el `fork()`

+ el fill no hereda els *timers* del pare (`alarm`)

#### Return value

El valor que retorna la **syscall** es diferent per a cada proces. Es a dir, despres del `fork()`, el pare rep un valor de retorn i el fill un altre.

#### Exemple

```c
// fins aqui només executa el PARE
int pid = fork();

// a partir d'aqui tant el PARE com el FILL executen aquest codi
// RETURN VALUES:
// per al PARE pid valdrà el PID del FILL o -1 en cas de que no s'hagi creat el process amb èxit
// per al FILL pid valdrà 0

if (pid == 0) {
    // ho executa el FILL només, pare no entre al if
}
else if (pid == -1) {
    // ho executa el PARE en cas d'error, fill no entra al if
}
else {
    // ho executa el PARE, pid = PID(fill)
}
```

### Exit

```c
void exit (int status);
```

Finalitza el proces voluntariament (pot finalitzar per altres raons involuntaries, *senyals*). Si el proces tenia algun fill, aquest passen a tenir com a pare el process 1 (**init process**). El proces PARE rep una senyal **SIGCHLD**. La variable `status` representa la rao per la qual ha finalitzat el proces i pot ser recuperada per el pare mitjançat les syscall `wait()`. El proces continua en estat **ZOMBIE** fins que el pare reculli el seu `exit status`.

#### Return Value

No hi ha valor de retorn, ja que es una funció `void`. A més no té sentit que n'hi hagi perque el proces finalitza.

#### Exemple

```c
// exemple de creacició de n processos i acabar-los
int main () {
    int n = 10;
    for (int i = 0; i < n; ++i) {
        int pid = fork();
        if (pid == 0) {
            exit(0);    // acaba el proces FILL
        }
        else if (pid == -1) {
            // tractem error
        }
        else {
            // el PARE no fa res
        }
    }
    // en aquest punt tenim nomes el process PARE
    // i 10 procesos FILL en estat ZOMBIE!
}
```

### Waitpid

```c
pid_t waitpid (pid_t pid, int *status, int options);
```

Suspen la execució del proces i espera a que un fill acabi. Parametres de la crida:

+ El paràmetre `pid` especifica el **PID** del fill al que s'ha de esperar. Si volem esperar a qualsevol fill: `pid = -1`

+ El paràmetre `status` es pasa per referencia, i és on la funció deixarà el estat del process fill. Aquí trobarem per exemple el seu `exit status`

+ El parametre `options` s'utilitza per definir el comportament del `waitpid`. Algunes de les opcions més importants son:
  
  + `options = 0`: el process que crida la funció es **bloqueja** fins que arribi un **SIGCHLD**
  
  + `options = WNOHANG`: la funcio retorna instantaneament si no ha acabat cap proces encara 

#### Return value

El valor de retorn del `waitpid` pot ser:

+ `return = -1`: Error. Pot ser per diferents raons, pero la que ens insteresa és l'error que es dona quan el proces no té cap FILL.

+ `return = 0`: Cap proces FILL ha acabat

+ `return > 0`: Especificant quin era el **PID** del proces que ha acabat

> Donem-nos compte que si no està activada la opció **WNOHANG** només tenen sentit els valors de retorn `-1` i `> 0`, ja que si no ha acabat cap fill seguirem esperant en bloqueig.

#### Exemple

```c
int main () {
    int n = 0;
    for (int i = 0; i < n; ++i) {
        int pid = fork();
        if (pid == 0) {
            for (int j = 0; j < 10000; ++j);  // creem una mica de delay
            exit(0);                          // els FILLS acaben aquí
        }
        else if (pid == -1) {
            // tractem error
        }
        else {
            // PARE no fa res
        }
    }

    // per aquí només pasa el proces PARE i com que hem creat delay, quan arribi
    // els fills encara no hauran mort
    int status;
    while (waitpid(-1, status, 0) > 0);   
    // aqui esperem a tots els FILLS ja que options=0, es a dir boquejem
    // fins que algun acabi, i ho fem sempre que el resultat sigui un PID (> 0).
    // Quan el resultat sigui -1 (no queden més FILLS) sortirem del bucle
}
```

### Execlp

```c
int execlp (const char *exec_file, const char *arg0, ..., (char *) NULL);
```

Canvia la imatge (es a dir el codi) del proces per a una altre. El proces segueix sent exactament el mateix:

+ els recursos utilitzats segueixen sent els mateixos

+ els tractaments de les **senyals** es reseteja.

#### Getpid/Getppid

```c
pid_t getpid();
pid_t getppid();
```

#### Return value

`getpid` retorna el PID del proces, i `getppid` retorna el **PID** del proces **PARE**.

#### Exemple

```c
int main () {
    char buffer[40];  // per escriure
    int pid = fork();

    if (pid == 0) {
        // FILL
        sprintf("FILL: PID=%d\n", getpid());
        write(1, buffer, strlen(buffer));
        exit(0);  // el FILL acaba
    }
    else if (pid == -1) {
        // PARE (en cas d'error)
        sprintf("Error en el fork!\n");
        write(1, buffer, strlen(buffer));
    }
    else {
        // PARE
        sprintf("PARE: PID=%d\n");
        write(1, buffer, strlen(buffer));
        waitpid(-1, NULL, 0); // esperem a que el FILL acabi
    }
}
```

---

## Comunicació entre processos
