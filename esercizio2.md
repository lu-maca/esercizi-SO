# Esercizio 2

Variabili
---

```c
uint8_t buffer[100];    // initialize all elements to 255
semaphore_t sem_even = 50;
semaphore_t sem_odd_div3 = 17;
semaphore_t sem_odd_nondiv3 = 33;
semaphore_t mutex_even = 1;
semaphore_t mutex_odd3 = 1;
semaphore_t mutex_odd = 1;
semaphore_t empty = 100;
semaphore_t full = 0;
int i, j, k, h = 0;
```

Consumatore
---

```c
while (1) 
{
    // wait if there are no process in queue
    wait(full);

    // take all semaphores, here we want to 
    // access to all indexes of buffer
    wait(mutex_even);
    wait(mutex_odd3);
    wait(mutex_odd);
    
    while (buffer[h] == 255)
    {
        h++;
    }
    h = h % 100;
    uint8_t item = buffer[h];
    buffer[h] = 255;
    if (item % 2 == 0) { signal(sem_even); }
    else if ((item % 2 == 1) && (item % 3 == 0)) { signal(sem_odd_div3); }
    else { signal(sem_odd_noddiv3); }

    signal(mutex_even);
    signal(mutex_odd3);
    signal(mutex_odd);
}
```

Produttore
---
```c
while (1) 
{
    wait(empty);
    uint8_t item = produce();
    
    // if item is even, wait until an even index 
    // is available
    if (item % 2 == 0) 
    {
        wait(sem_even);
        wait(mutex_even);
        buffer[i] = item;
        i = (i + 2) % 100;
        signal(mutex_even);
    }
    // if item is odd and divisible by 3, wait
    // until an odd index (divisible by 3) is available 
    else if ((item % 2 == 1) && (item % 3 == 0))
    {
        wait(sem_odd_div3);
        wait(mutex_odd3);
        buffer[j] = item;
        j == 99? j = 3 : j += 6;
        signal(mutex_odd3);
    }
    // otherwise, wait until an odd inex not divisible
    // by 3 is available
    else 
    {
        wait(sem_odd_nondiv3);
        wait(mutex_odd);
        buffer[k] = item;
        do
        {
            k += 2;
        }
        while (k % 3 == 0)
        k %= 100;
        signal(mutex_odd)
    }  
    signal(full);
}

```
