# Esercizio 7

Un distributore automatico e' fornito di confezioni di biscotti e
cracker. Dispone di 50 cestelli, ognuno dei quali può ospitare una
sola confezione.

Esistono 4 categorie di processi:
- fornitori di biscotti: inseriscono una confezione di biscotti nel
distributore
- consumatori di biscotti: acquistano una confezione di biscotti dal
distributore
- fornitori di cracker: inseriscono una confezione di cracker nel
distributore
- consumatori di cracker: acquistano una confezione di cracker dal
distributore

Quando un fornitore di cracker vuole inserire una confezione e ci
sono cestelli liberi, non può farlo se sono valide entrambe le
seguenti condizioni:
- il numero di confezioni di cracker già presenti e' >= del numero
di confezioni di biscotti già presenti (A)
- il numero di confezioni di cracker già presenti e' >= 3 (B).
I fornitori che tentano di rifornire prodotti ma non possono farlo
devono essere messi in attesa.

I consumatori che desiderano acquistare prodotti non presenti,
rinunciano (non devono essere messi in attesa).
Programmare il sistema usando i semafori con la semantica
tradizionale.

## Variabili
```c
#define FULL_CAPACITY 50

int waiting_cookie = 0;
int waiting_cracker = 0;

int tot_cookie = 0;
int tot_cracker = 0;

semaphore_t mutex = 1;
semaphore_t sem_cookie = 0;
semaphore_t sem_cracker = 0;
```

## Produttore biscotti

```c
while (1)
{
    lock(mutex);
    if ( FULL_CAPACITY == (tot_cookie + tot_cracker) )
    {
        // block the process: machine is full
        waiting_cookie++;
        // release mutex
        unlock(mutex);
        wait(sem_cookie);
        // acquire the mutex again
        lock(mutex);
    }

    waiting_cookie--;
    tot_cookie++;

    // signal to cracker producer to run if
    // the only condition involving cookies
    // to add cracker is now not respected
    if ( tot_cracker < tot_cookie 
        && waiting_cracker > 0)
    {
        signal(sem_cracker);
    }
    // release mutex
    unlock(mutex);
}
```

## Produttore cracker

```c
while (1)
{
    lock(mutex);
    if ( FULL_CAPACITY == (tot_cookie + tot_cracker) 
        || ( tot_cracker >= tot_cookie 
        && tot_cracker >= 3 ))
    {
        // block the process: machine is full
        // or conditions A and B are respected
        waiting_cracker++;
        // release mutex
        unlock(mutex);
        wait(sem_cracker);
        // acquire the mutex again
        lock(mutex);
    }

    waiting_cracker--;
    tot_cracker++;

    // release mutex
    unlock(mutex);
}
```

## Consumatore biscotti

```c
while (1)
{
    lock(mutex);
    // check if number of cookie is greater
    // than zero; if so, the consumer can 
    // take a cookie, it can't otherwise
    if ( 0 < tot_cookie )
    {
        // take a cookie
        tot_cookie--; 
        { /* take it */ }
        if (waiting_cookie > 0 )
        {
            // if a cookie producer is waiting
            // unblock it
            signal(sem_cookie);
        }
        else if ( waiting_cracker > 0 
            && (tot_cracker < tot_cookie
            || tot_cracker < 3)) 
        {
            signal(sem_cracker);
        }
    }
    else
    {
        printf("Cookie consumer %d goes away\n", getpid())
    }
    unlock(mutex);
}
```

## Consumatore cracker

```c
while (1)
{
    lock(mutex);
    // check if number of cracker is greater
    // than zero; if so, the consumer can 
    // take a cracker, it can't otherwise
    if ( 0 < tot_cracker )
    {
        // take a cracker
        tot_cracker--; 
        { /* take it */ }
        if (waiting_cracker > 0 
            && (tot_cracker < tot_cookie
            || tot_cracker < 3) )
        {
            // if a cracker producer is waiting
            // unblock it
            signal(sem_cracker);
        }
        else if ( waiting_cookie > 0 ) 
        {
            signal(sem_cookie);
        }
    }
    else
    {
        printf("Cracker consumer %d goes away\n", getpid())
    }
    unlock(mutex);
}
```