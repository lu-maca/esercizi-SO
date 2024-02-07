# Esercizio 8

Un parcheggio con una disponibilita’ totale di 50 posti viene usato
da vetture bianche e da vetture di altro colore.

Quando una vettura bianca vuole entrare, il numero di vetture
bianche non puo’ diventare maggiore del numero delle vetture di
altro colore.
Il parcheggio ha un unico gate, usato dalle vetture per entrare ed
uscire.

Le vetture che tentano di entrare devono essere messe in coda di
attesa non possono entrare per la mancanza di posti disponibili,
oppure per la violazione della condizione sui colori sopra
descritta.

Non deve capitare che ci siano vetture in attesa senza ragione.
Programmare l'ingresso e l'uscita dal parcheggio delle vetture,
usando i semafori con la semantica tradizionale.

## Variabili
```c
int totB = 0;
int totN = 0;
int waitB = 0;
int waitN = 0;
semaphore_t semB = 0;
semaphore_t semN = 0;
semaphore_t mutex = 1;
```

## Soluzione
### Macchine bianche
```c
while (1) 
{
    enterB();
    park();
    exitB();
}
```

### Macchine colorate
```c
while (1) 
{
    enterN();
    park();
    exitN();
}
```

### Implementazioni delle funzioni
```c
void enterB()
{
    lock(mutex);
    if (totB + totN == 50 || totB >= totN)
    {
        // Enter only if there are free slots
        // and given condition is respected.
        // Otherwise, wait until a condition
        // is respected

        // increase the white waiting car number
        waitB++;
        unlock(mutex);
        wait(semB);
        // after wait, we need to lock the mutex
        // again to use shared variables
        lock(mutex);
        // decrease the white waiting car number
        waitB--;
    }

    // enter the white car
    carB++;
    // release the mutex
    unlock(mutex);
}
```

```c
void enterN()
{
    lock(mutex);
    if (totB + totN == 50)
    {
        // Enter only if there are free slots.
        // Otherwise, wait until a condition
        // is respected

        // increase the colored waiting car number
        waitN++;
        unlock(mutex);
        wait(semN);
        // after wait, we need to lock the mutex
        // again to use shared variables
        lock(mutex);
        // decrease the colored waiting car number
        waitN--;
    }
    
    // enter a colored car
    carN++;

    // With a new colored car, given condition 
    // can be respected again; signal the semB 
    // in that case (if someone is waiting)
    if (totB + totN < 50 
        && totB < totN 
        && waitB > 0)
    {
        signal(semB);
    }
    
    // release the mutex
    unlock(mutex);
}
```

```c
void exitB()
{
    lock(mutex);
    
    // exit
    totB--;
    
    if (waitB > 0 && totB < totN)
    {
        // this implementation checks if a
        // white car is waiting and if the 
        // condition still holds; in that
        // case it gives the white car the 
        // permission to enter
        signal(semB);
    }
    else if (waitN > 0)
    {   
        // Otherwise it checks if a colored
        // car is waiting and in that case
        // it gives it the permission to enter
        signal(semN);
    }
    unlock(mutex);
}
```

```c
void exitN()
{
    lock(mutex);
    
    totN--;

    if (waitN > 0) 
    {
        // here the first check is on the 
        // colored waiting cars
        signal(semN);
    }
    else if (waitB > 0 && totB < totN)
    {
        // the second check is important 
        // because, since a colored car 
        // goes out, we want to be sure the
        // second condition is satisfied 
        // before giving the permission to
        // a white waiting car
        signal(semB);
    }

    unlock(mutex);
}
```