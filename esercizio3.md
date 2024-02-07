# Esercizio 3

Si consideri il classico problema dei produttori e consumatori, con
il buffer implementato con un array di interi di dimensione 100.
Si assuma, per semplicita’, che gli interi prodotti e consumati
siano >=0. 

Si considerino le seguenti condizioni aggiuntive:
 * Gli interi pari possono occupare solo le posizioni da 0 a 49;
 * Gli interi dispari possono occupare solo le posizioni da 50 a 99;
 * Vi e’ una nuova categoria di processi, i processi ConsPari, che
consumano tutti gli interi pari, se l’array contiene almeno 20
interi pari;

Quanto un processo tenta di effettuare un’operazione al momento non
consentita (per esempio produrre un intero che il buffer non può al
momento ospitare), il processo deve essere messo in attesa.
Programmare il sistema sfruttando i semafori con la semantica
tradizionale.

## Variabili
```c
#define BUFFER_LEN 100

int buffer[BUFFER_LEN]; // inizializzato a -1
semaphore_t sem_even = 50;
sempahore_t sem_odd = 50;
semaphore_t empty = 100;
semaphore_t full = 0;

mutex_t mutex_p_i = 1;
mutex_t mutex_p_j = 1;
mutex_t mutex_c = 1;

int i, j, k = 0;

```

## Consumatore normale
```c
while (1)
{
    wait(full);
    lock(mutex_c);
    // cycle through the buffer to find a valid
    // (not -1) 
    int item = buffer[k];
    while (-1 == item)
    {
        k = k + 1;
        item = buffer[k];
    }

    buffer[k] = -1;
    k = k + 1;
    unlock(mutex_c);

    if (item % 2)
    { 
        signal(sem_even); 
    }
    else 
    {
        signal(sem_odd);
    }
    signal(empty);
}
```

## Consumatore pari
```c
while (1)
{
    // array of integers used from this process
    // somehow, filled in the critical section
    int pippo[50]; // -1 initialized
    // ...
    wait(full);
    lock(mutex_c);

    // count even elements in the first half of
    // the array
    uint8_t even_elements = 0;
    for (int l = 0; l < 50; l++)
    {
        if (-1 != buffer[l]) { even_elements++; }
    }

    // remove all even elements from the buffer
    if (even_elements >= 20)
    {
        for (int l = 0; l < 50; l++)
        {
            pippo[l] = buffer[l];
            buffer[l] = -1;
            signal(sem_even); 
        }
    }
    unlock(mutex_c);
    signal(empty);

}

```

## Produttore
```c
while (1)
{
    wait(empty);
    int item = produce();

    if (item % 2 == 0)
    {
        wait(sem_even);
        lock(mutex_p_i);
        buffer[i] = item;
        // update i in the range [0; 49]
        i = (i + 1) % 50;
        unlock(mutex_p_i);
    }
    else 
    {
        wait(sem_odd);
        lock(mutex_p_j);
        buffer[j] = item;
        // update j in the range [50, 99]
        j = 50 + (j + 1) % 50;
        unlock(mutex_p_j);
    }
    
    signal(full);
}
```
