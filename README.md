exp 4 
#include<pthread.h>
#include<semaphore.h>
#include<unistd.h>
#include<stdio.h>
void *calcPrime(void* input){
int *n=(int *)input;
int flag=0;
for(int i=2;i<=(*n/2);i++){
if (*n%i==0){
flag++;
break;
}
}
if(flag==0){
printf("%d is prime\n",*n);
}
else{
printf("%d isn't prime\n",*n);
}
}
void *armstrong(void* input){
int *n=(int *)input;
int temp=*n;
int digits=0;
for (temp=*n;temp!=0;temp/=10){
digits++;
}
int res=0;
for(int temp=*n;temp!=0;temp/=10){
int rem=temp%10;
int prod=1;
for(int i=0;i<digits;i++){
prod*=rem;
}
res+=prod;
temp/=10;
}
if(res==*n){
printf("%d is an armstrong number.\n",*n);
}
else{
printf("%d isn't an armstrong number.\n",*n);
}
}
void *fact(void* input){
int *n=(int *)input;
int prod=1;
for (int i=1;i<=*n;i++){
prod*=i;
}
printf("the factorial of the number is %d",prod);
}
int main(){
pthread_t threads[3];
int n;
printf("enter the number : ");
scanf("%d",&n);
int *ptr=&n;
pthread_create(&threads[0],NULL,calcPrime,(void *)&n);
pthread_create(&t hreads[1],NULL,armstrong,(void *)&n);
pthread_create(&threads[2],NULL,fact,(void *)&n);
pthread_join(threads[0],NULL);
pthread_join(threads[1],NULL);
pthread_join(threads[2],NULL);
return 0;
}


exp-5(peterson for n):
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdbool.h>

#define N 3 // Define the number of processes

int turn[N - 1];       // Turn array for hierarchy of decisions
bool flag[N];          // Flag array to indicate if process wants to enter CS

void* process(void* arg) {
    int i = *(int*)arg;
    
    for (int level = 0; level < N - 1; ++level) {
        flag[i] = true;              // Indicate process wants to enter CS
        turn[level] = i;             // Set turn at this level to process i
        
        // Wait until no other process at this level or higher wants to enter CS
        while (turn[level] == i && flag[(i + 1) % N]) {
            // Busy wait
        }
    }

    // Critical Section
    printf("Process %d is in the critical section.\n", i);
    sleep(2); // Simulate work in the critical section

    // Exit section
    flag[i] = false;                 // Indicate process is leaving the critical section

    // Remainder section
    printf("Process %d is in the remainder section.\n", i);

    return NULL;
}

int main() {
    pthread_t threads[N];
    int ids[N];

    // Initialize the ids for each process
    for (int i = 0; i < N; ++i) {
        ids[i] = i;
    }

    // Create threads for each process
    for (int i = 0; i < N; ++i) {
        pthread_create(&threads[i], NULL, process, &ids[i]);
    }

    // Wait for all threads to finish
    for (int i = 0; i < N; ++i) {
        pthread_join(threads[i], NULL);
    }

    return 0;
}

exp-5(peterson for one):
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <stdbool.h>

bool flag[2] = {false, false}; // Flags to indicate if a process wants to enter CS
int turn = 0;                  // Variable to indicate whose turn it is

void* process_0() {
    flag[0] = true;            // P0 wants to enter the critical section
    turn = 1;                  // Give turn to P1
    while (flag[1] && turn == 1) {
        // Wait until P1 finishes its critical section
    }

    // Critical Section of P0
    printf("Process 0 is in the critical section.\n");
    sleep(2); // Simulating work in CS

    // Exit section
    flag[0] = false;           // P0 is leaving the critical section

    // Remainder section
    printf("Process 0 is in the remainder section.\n");
    return NULL;
}

void* process_1() {
    flag[1] = true;            // P1 wants to enter the critical section
    turn = 0;                  // Give turn to P0
    while (flag[0] && turn == 0) {
        // Wait until P0 finishes its critical section
    }

    // Critical Section of P1
    printf("Process 1 is in the critical section.\n");
    sleep(2); // Simulating work in CS

    // Exit section
    flag[1] = false;           // P1 is leaving the critical section

    // Remainder section
    printf("Process 1 is in the remainder section.\n");
    return NULL;
}

int main() {
    pthread_t t1, t2;

    // Create threads for both processes
    pthread_create(&t1, NULL, process_0, NULL);
    pthread_create(&t2, NULL, process_1, NULL);

    // Wait for both threads to finish
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    return 0;
}

exp-6(A)(bounded buffer):
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define BUFFER_SIZE 5
#define NUM_PRODUCERS 2
#define NUM_CONSUMERS 2

int buffer[BUFFER_SIZE];
int in = 0;
int out = 0;

sem_t empty;
sem_t full;
pthread_mutex_t mutex;

void* producer(void* arg) {
    int item;
    for (int i = 0; i < 10; i++) {
        item = rand() % 100; // produce an item
        sem_wait(&empty);     // decrease the count of empty slots
        pthread_mutex_lock(&mutex); // enter critical section
        
        buffer[in] = item;    // insert item into buffer
        printf("Producer %ld produced %d\n", (long)arg, item);
        in = (in + 1) % BUFFER_SIZE; // circular buffer
        
        pthread_mutex_unlock(&mutex); // exit critical section
        sem_post(&full);      // increase the count of full slots
        
        sleep(rand() % 2); // simulate time taken to produce
    }
    return NULL;
}

void* consumer(void* arg) {
    int item;
    for (int i = 0; i < 10; i++) {
        sem_wait(&full);      // decrease the count of full slots
        pthread_mutex_lock(&mutex); // enter critical section
        
        item = buffer[out];   // remove item from buffer
        printf("Consumer %ld consumed %d\n", (long)arg, item);
        out = (out + 1) % BUFFER_SIZE; // circular buffer
        
        pthread_mutex_unlock(&mutex); // exit critical section
        sem_post(&empty);     // increase the count of empty slots
        
        sleep(rand() % 2); // simulate time taken to consume
    }
    return NULL;
}

int main() {
    pthread_t producers[NUM_PRODUCERS], consumers[NUM_CONSUMERS];

    sem_init(&empty, 0, BUFFER_SIZE); // initially all slots are empty
    sem_init(&full, 0, 0);            // initially no slots are full
    pthread_mutex_init(&mutex, NULL);

    // Create producer threads
    for (long i = 0; i < NUM_PRODUCERS; i++) {
        pthread_create(&producers[i], NULL, producer, (void*)i);
    }

    // Create consumer threads
    for (long i = 0; i < NUM_CONSUMERS; i++) {
        pthread_create(&consumers[i], NULL, consumer, (void*)i);
    }

    // Wait for all producers to finish
    for (int i = 0; i < NUM_PRODUCERS; i++) {
        pthread_join(producers[i], NULL);
    }

    // Wait for all consumers to finish
    for (int i = 0; i < NUM_CONSUMERS; i++) {
        pthread_join(consumers[i], NULL);
    }

    sem_destroy(&empty);
    sem_destroy(&full);
    pthread_mutex_destroy(&mutex);

    return 0;
}

exp-6(A)(unbounded buffer):
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

typedef struct Node {
    int data;
    struct Node* next;
} Node;

Node* head = NULL;
pthread_mutex_t mutex;
sem_t empty;

void* producer(void* arg) {
    int item;
    for (int i = 0; i < 10; i++) {
        item = rand() % 100; // produce an item
        
        pthread_mutex_lock(&mutex); // enter critical section
        
        // Add item to linked list (buffer)
        Node* newNode = (Node*)malloc(sizeof(Node));
        newNode->data = item;
        newNode->next = head;
        head = newNode;
        
        printf("Producer %ld produced %d\n", (long)arg, item);
        
        pthread_mutex_unlock(&mutex); // exit critical section
        sem_post(&empty); // signal that there is a new item
        
        sleep(rand() % 2); // simulate time taken to produce
    }
    return NULL;
}

void* consumer(void* arg) {
    int item;
    for (int i = 0; i < 10; i++) {
        sem_wait(&empty); // wait for an item
        
        pthread_mutex_lock(&mutex); // enter critical section
        
        // Remove item from linked list (buffer)
        if (head != NULL) {
            Node* temp = head;
            item = head->data;
            head = head->next;
            free(temp);
            printf("Consumer %ld consumed %d\n", (long)arg, item);
        }
        
        pthread_mutex_unlock(&mutex); // exit critical section
        
        sleep(rand() % 2); // simulate time taken to consume
    }
    return NULL;
}

int main() {
    pthread_t producers[2], consumers[2];

    sem_init(&empty, 0, 0);
    pthread_mutex_init(&mutex, NULL);

    // Create producer threads
    for (long i = 0; i < 2; i++) {
        pthread_create(&producers[i], NULL, producer, (void*)i);
    }

    // Create consumer threads
    for (long i = 0; i < 2; i++) {
        pthread_create(&consumers[i], NULL, consumer, (void*)i);
    }

    // Wait for all producers to finish
    for (int i = 0; i < 2; i++) {
        pthread_join(producers[i], NULL);
    }

    // Wait for all consumers to finish
    for (int i = 0; i < 2; i++) {
        pthread_join(consumers[i], NULL);
    }

    // Clean up
    while (head != NULL) {
        Node* temp = head;
        head = head->next;
        free(temp);
    }
    
    sem_destroy(&empty);
    pthread_mutex_destroy(&mutex);

    return 0;
}

exp-6(B)(reader):
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define NUM_READERS 5
#define NUM_WRITERS 3
#define MAX_READS 3
#define MAX_WRITES 2

sem_t read_mutex;     // Mutex for read count
sem_t resource;       // Semaphore for resource access
int read_count = 0;   // Number of readers currently reading

typedef struct {
    int id;
    int read_count;  // Number of reads completed
} Reader;

typedef struct {
    int id;
    int write_count; // Number of writes completed
} Writer;

void* reader(void* arg) {
    Reader* reader_data = (Reader*)arg;

    while (reader_data->read_count < MAX_READS) {
        // Start reading
        sem_wait(&read_mutex); // Enter critical section for read_count
        read_count++;
        if (read_count == 1) {
            sem_wait(&resource); // First reader locks the resource
        }
        sem_post(&read_mutex); // Exit critical section

        // Reading section
        printf("Reader %d is reading.\n", reader_data->id);
        sleep(rand() % 2); // Simulate reading time

        // Done reading
        sem_wait(&read_mutex); // Enter critical section for read_count
        read_count--;
        if (read_count == 0) {
            sem_post(&resource); // Last reader unlocks the resource
        }
        sem_post(&read_mutex); // Exit critical section

        reader_data->read_count++; // Increment reads completed
        sleep(rand() % 2); // Simulate thinking time
    }
    printf("Reader %d has finished reading.\n", reader_data->id);
    free(reader_data);
    return NULL;
}

void* writer(void* arg) {
    Writer* writer_data = (Writer*)arg;

    while (writer_data->write_count < MAX_WRITES) {
        // Writing section
        sem_wait(&resource); // Lock the resource for writing
        printf("Writer %d is writing.\n", writer_data->id);
        sleep(rand() % 2); // Simulate writing time
        sem_post(&resource); // Unlock the resource

        writer_data->write_count++; // Increment writes completed
        sleep(rand() % 2); // Simulate thinking time
    }
    printf("Writer %d has finished writing.\n", writer_data->id);
    free(writer_data);
    return NULL;
}

int main() {
    pthread_t readers[NUM_READERS];
    pthread_t writers[NUM_WRITERS];

    // Initialize semaphores
    sem_init(&read_mutex, 0, 1); // Mutex for reader count
    sem_init(&resource, 0, 1);    // Semaphore for resource access

    // Create reader threads
    for (int i = 0; i < NUM_READERS; i++) {
        Reader* r = malloc(sizeof(Reader));
        r->id = i;
        r->read_count = 0; // Initialize read count
        pthread_create(&readers[i], NULL, reader, r);
    }

    // Create writer threads
    for (int i = 0; i < NUM_WRITERS; i++) {
        Writer* w = malloc(sizeof(Writer));
        w->id = i;
        w->write_count = 0; // Initialize write count
        pthread_create(&writers[i], NULL, writer, w);
    }

    // Join reader threads
    for (int i = 0; i < NUM_READERS; i++) {
        pthread_join(readers[i], NULL);
    }

    // Join writer threads
    for (int i = 0; i < NUM_WRITERS; i++) {
        pthread_join(writers[i], NULL);
    }

    // Destroy semaphores
    sem_destroy(&read_mutex);
    sem_destroy(&resource);

    return 0;
}

exp-6(B)(writer):
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define NUM_READERS 5
#define NUM_WRITERS 3

sem_t mutex;          // For mutual exclusion
sem_t wrt;            // For writers
int read_count = 0;   // Number of readers currently reading
int write_count = 0;  // Number of writers currently waiting

void* reader(void* arg) {
    int id = (int)(long)arg;
    for (int i = 0; i < 5; i++) {
        sem_wait(&mutex);          // Start of critical section
        if (write_count > 0) {
            sem_post(&mutex);      // If writers are waiting, release mutex
            continue;              // Skip reading
        }
        read_count++;
        if (read_count == 1) {
            sem_wait(&wrt);        // First reader locks the writer
        }
        sem_post(&mutex);          // End of critical section
        
        // Reading section
        printf("Reader %d is reading\n", id);
        sleep(1);
        
        sem_wait(&mutex);          // Start of critical section
        read_count--;
        if (read_count == 0) {
            sem_post(&wrt);        // Last reader unlocks the writer
        }
        sem_post(&mutex);          // End of critical section
    }
    return NULL;
}

void* writer(void* arg) {
    int id = (int)(long)arg;
    for (int i = 0; i < 5; i++) {
        sem_wait(&mutex);          // Start of critical section
        write_count++;             // Increase the count of writers waiting
        sem_post(&mutex);          // End of critical section

        sem_wait(&wrt);            // Writers wait for access

        // Writing section
        printf("Writer %d is writing\n", id);
        sleep(1);
        
        sem_post(&wrt);            // Release the lock

        sem_wait(&mutex);          // Start of critical section
        write_count--;             // Decrease the count of writers waiting
        sem_post(&mutex);          // End of critical section
    }
    return NULL;
}

int main() {
    pthread_t readers[NUM_READERS], writers[NUM_WRITERS];
    
    sem_init(&mutex, 0, 1);       // Initialize mutex
    sem_init(&wrt, 0, 1);         // Initialize writer semaphore

    // Create reader threads
    for (long i = 0; i < NUM_READERS; i++) {
        pthread_create(&readers[i], NULL, reader, (void*)i);
    }

    // Create writer threads
    for (long i = 0; i < NUM_WRITERS; i++) {
        pthread_create(&writers[i], NULL, writer, (void*)i);
    }

    // Wait for all readers to finish
    for (int i = 0; i < NUM_READERS; i++) {
        pthread_join(readers[i], NULL);
    }

    // Wait for all writers to finish
    for (int i = 0; i < NUM_WRITERS; i++) {
        pthread_join(writers[i], NULL);
    }

    sem_destroy(&mutex);
    sem_destroy(&wrt);
    
    return 0;
}

exp-7(Bankers):
#include <stdio.h>
#include <stdbool.h>

#define P 5 // Number of processes
#define R 3 // Number of resources

// Allocation matrix
int allocation[P][R] = {
    {0, 1, 0},
    {2, 0, 0},
    {3, 0, 2},
    {2, 1, 1},
    {0, 0, 2}
};

// Maximum demand matrix
int max[P][R] = {
    {7, 5, 3},
    {3, 2, 2},
    {9, 0, 2},
    {2, 2, 2},
    {4, 3, 3}
};

// Available resources
int available[R] = {3, 3, 2};

// Function to check if a process can finish
bool canFinish(int process, int work[], bool finish[]) {
    for (int j = 0; j < R; j++) {
        if (max[process][j] - allocation[process][j] > work[j]) {
            return false; // Process cannot finish if it needs more than available resources
        }
    }
    return true;
}

// Function to check if the system is in a safe state
bool isSafe() {
    int work[R];
    bool finish[P] = {false};

    // Copy available resources to work
    for (int i = 0; i < R; i++) {
        work[i] = available[i];
    }

    int count = 0;
    while (count < P) {
        bool found = false;
        for (int i = 0; i < P; i++) {
            if (!finish[i] && canFinish(i, work, finish)) {
                for (int j = 0; j < R; j++) {
                    work[j] += allocation[i][j]; // Release resources
                }
                finish[i] = true; // Mark process as finished
                found = true;
                count++;
                printf("Process P%d can finish. Current work state: {", i);
                for (int j = 0; j < R; j++) {
                    printf("%d", work[j]);
                    if (j != R - 1) printf(", ");
                }
                printf("}\n");
            }
        }
        if (!found) {
            return false; // Not in a safe state
        }
    }
    return true; // Safe state
}

// Function to request resources
bool requestResources(int processNum, int request[]) {
    // Check if request exceeds maximum claim
    for (int j = 0; j < R; j++) {
        if (request[j] > max[processNum][j] - allocation[processNum][j]) {
            printf("Error: Process P%d exceeded its maximum claim.\n", processNum);
            return false; // Request exceeds maximum claim
        }
    }

    // Check if request can be satisfied
    for (int j = 0; j < R; j++) {
        if (request[j] > available[j]) {
            printf("Process P%d must wait; resources not available.\n", processNum);
            return false; // Not enough resources available
        }
    }

    // Pretend to allocate the requested resources
    printf("Allocating resources to Process P%d. Request: {", processNum);
    for (int j = 0; j < R; j++) {
        printf("%d", request[j]);
        if (j != R - 1) printf(", ");
    }
    printf("}\n");

    for (int j = 0; j < R; j++) {
        available[j] -= request[j];
        allocation[processNum][j] += request[j];
    }

    // Check if the system is in a safe state after allocation
    if (isSafe()) {
        printf("Resources allocated to Process P%d safely.\n", processNum);
        return true; // Request can be granted
    } else {
        // Rollback the allocation if not safe
        for (int j = 0; j < R; j++) {
            available[j] += request[j];
            allocation[processNum][j] -= request[j];
        }
        printf("Resources cannot be allocated to Process P%d; system would be unsafe.\n", processNum);
        return false; // Request cannot be granted
    }
}

int main() {
    // Example requests
    int request1[R] = {1, 0, 2}; // Request from process P1
    requestResources(1, request1); // Attempt to request resources for process P1

    int request4[R] = {3, 3, 0}; // Request from process P4
    requestResources(4, request4); // Attempt to request resources for process P4

    int request2[R] = {0, 0, 0}; // Modified request from process P2 (valid)
    requestResources(2, request2); // Attempt to request resources for process P2

    return 0;
}

exp-8(deadlock detection):
#include <stdio.h>
#include <stdbool.h>

#define P 5 // Number of processes
#define R 3 // Number of resources

// Allocation matrix
int allocation[P][R] = {
    {0, 1, 0},
    {2, 0, 0},
    {3, 0, 2},
    {2, 1, 1},
    {0, 0, 2}
};

// Request matrix
int max[P][R] = {
    {7, 5, 3},
    {3, 2, 2},
    {9, 0, 2},
    {2, 2, 2},
    {4, 3, 3}
};

// Available resources
int available[R] = {3, 3, 2};

// Function to print the current finish state
void printFinishState(bool finish[]) {
    printf("Finish state: ");
    for (int i = 0; i < P; i++) {
        printf("P%d: %s ", i, finish[i] ? "true" : "false");
    }
    printf("\n");
}

// Function to check if the system is in a deadlock state
bool detectDeadlock() {
    int work[R];
    bool finish[P] = {false};

    // Initialize work with available resources
    for (int i = 0; i < R; i++) {
        work[i] = available[i];
    }

    int count = 0; // To track finished processes

    while (count < P) {
        bool found = false; // Flag to check if any proce
