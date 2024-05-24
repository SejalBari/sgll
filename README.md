// 4. Implement the solutions for following synchronization problems using semaphore: 
// i. Producer Consumer  
// ii. Reader Writer 

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>

#define BUFFER_SIZE 10

int buffer[BUFFER_SIZE];
int in = 0, out = 0;
sem_t mutex, full, empty;

void *producer(void *arg) {
    int item = 1; 
    while (1) {
        sem_wait(&empty);       
        sem_wait(&mutex);

        
        buffer[in] = item++;
        printf("Producer produces item %d\n", buffer[in]);
        in = (in + 1) % BUFFER_SIZE;

        sem_post(&mutex);
        sem_post(&full);

       
        sleep(1);
    }
}

void *consumer(void *arg) {
    while (1) {
        sem_wait(&full);
        sem_wait(&mutex);

        int consumed_item = buffer[out];
        printf("Consumer consumes item %d\n", consumed_item);
        out = (out + 1) % BUFFER_SIZE;

        sem_post(&mutex);
        sem_post(&empty);

        sleep(2);
    }
}

int main() {
    pthread_t producer_thread, consumer_thread;
    sem_init(&mutex, 0, 1);
    sem_init(&full, 0, 0);
    sem_init(&empty, 0, BUFFER_SIZE);

    pthread_create(&producer_thread, NULL, producer, NULL);
    pthread_create(&consumer_thread, NULL, consumer, NULL);

    pthread_join(producer_thread, NULL);
    pthread_join(consumer_thread, NULL);
 
    sem_destroy(&mutex);
    sem_destroy(&full);
    sem_destroy(&empty);

    return 0;
}






-------------------



#include <semaphore.h>
#include <stdio.h>
#include <pthread.h>

sem_t x, y;
pthread_t writerthreads[100], readerthreads[100];
int readercount = 0;

void* reader(void *param) {
    sem_wait(&x);
    readercount++;
    if (readercount == 1)
        sem_wait(&y);
    sem_post(&x);
    printf("\n%d reader is inside", readercount);
    sem_wait(&x);
    readercount--;
    if (readercount == 0) {
        sem_post(&y);
    }
    sem_post(&x);
    printf("\n%d Reader is leaving", readercount + 1);
    return NULL;
}

void* writer(void *param) {
    printf("\nWriter is trying to enter");
    sem_wait(&y);
    printf("\nWriter has entered");
    sem_post(&y);
    printf("\nWriter is leaving");
    return NULL;
}

int main() {
    int n2, i;
    printf("Enter the number of readers:");
    scanf("%d", &n2);
    sem_init(&x, 0, 1);
    sem_init(&y, 0, 1);
    for (i = 0; i < n2; i++) {
        pthread_create(&readerthreads[i], NULL, reader, NULL);
    }
    for (i = 0; i < n2; i++) {
        pthread_create(&writerthreads[i], NULL, writer, NULL);
    }
    for (i = 0; i < n2; i++) {
        pthread_join(readerthreads[i], NULL);
        pthread_join(writerthreads[i], NULL);
    }
    return 0;
}
