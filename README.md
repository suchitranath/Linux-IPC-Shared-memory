name: Suhcitra Nath
reg no: 212223220112
# Linux-IPC-Shared-memory
Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that illustrates two processes communicating using shared memory.

//shmry1.c

```
#include <unistd.h>      // For process management functions
#include <stdlib.h>      // For general utilities
#include <stdio.h>       // For standard I/O operations
#include <string.h>      // For string manipulation functions
#include <sys/shm.h>     // For shared memory operations

#define TEXT_SZ 2048     // Size of the shared text buffer

// Define a structure for shared memory
struct shared_use_st {
    int written_by_you;     // Flag indicating whether data is written
    char some_text[TEXT_SZ];  // Buffer to hold the shared text
};

int main() {
    int running = 1;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;
    int shmid;

    srand((unsigned int)getpid()); // Generate random seed based on process ID

    // Creating or accessing shared memory segment
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }

    // Attaching shared memory segment
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    // Main loop
    while (running) {
        // Check if data has been written
        shared_stuff = (struct shared_use_st *)shared_memory;
        if (shared_stuff->written_by_you) {
            // Process the shared text
            printf("You Wrote: %s", shared_stuff->some_text);
            sleep(rand() % 4);
            shared_stuff->written_by_you = 0;
            if (strncmp(shared_stuff->some_text, "end", 3) == 0) {
                running = 0;
            }
        }
    }

    // Detaching shared memory
    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    // Deleting shared memory segment
    if (shmctl(shmid, IPC_RMID, 0) == -1) {
        fprintf(stderr, "failed to delete\n");
        exit(EXIT_FAILURE);
    }

    // Exiting the program
    exit(EXIT_SUCCESS);
}

```
//shmry2.c
```
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main() {
    int running = 1;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;
    char buffer[BUFSIZ];
    int shmid;

    // Create or access shared memory segment
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    printf("Shared memory id = %d\n", shmid);
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }

    // Attach shared memory segment
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    printf("Memory Attached at %p\n", shared_memory);

    // Point to the shared memory region
    shared_stuff = (struct shared_use_st *)shared_memory;

    // Main loop
    while (running) {
        // Wait for the other process to read the data
        while (shared_stuff->written_by_you == 1) {
            sleep(1);
            printf("waiting for client.\n");
        }

        // Read input from the user
        printf("Enter Some Text: ");
        fgets(buffer, BUFSIZ, stdin);

        // Copy input into the shared memory
        strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
        shared_stuff->written_by_you = 1;

        // Check for termination command
        if (strncmp(buffer, "end", 3) == 0) {
            running = 0;
        }
    }

    // Detach shared memory
    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    // Exit the program
    exit(EXIT_SUCCESS);
}

```




## OUTPUT
![Screenshot from 2024-05-13 21-28-18](https://github.com/suchitranath/Linux-IPC-Shared-memory/assets/145742631/d32bba7c-d3f8-4f18-93e4-9a21615aebce)
![Screenshot from 2024-05-13 21-29-22](https://github.com/suchitranath/Linux-IPC-Shared-memory/assets/145742631/f14c7a17-4e3c-4761-ba05-92fb202821ea)
![Screenshot from 2024-05-13 21-33-38](https://github.com/suchitranath/Linux-IPC-Shared-memory/assets/145742631/e5704557-0514-4cce-9e85-d5e4adcd514b)



# RESULT:
The program is executed successfully.
