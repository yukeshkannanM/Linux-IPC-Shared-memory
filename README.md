# Linux-IPC-Shared-memory
## Ex06-Linux IPC-Shared-memory

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
```
//shm1.o
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>

#define TEXT_SZ 2048

struct shared_use_st
{
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main()
{
    int running = 1;
    void *shared_memory = NULL;
    struct shared_use_st *shared_stuff;
    int shmid;

    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666);
    if (shmid == -1)
    {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }

    printf("Shared memory id is %d\n", shmid);

    shared_memory = shmat(shmid, NULL, 0);
    if (shared_memory == (void *)-1)
    {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;

    while (running)
    {
        if (shared_stuff->written_by_you == 1)
        {
            printf("You wrote: %s\n", shared_stuff->some_text);

            if (strncmp(shared_stuff->some_text, "end", 3) == 0)
            {
                running = 0;
            }

            shared_stuff->written_by_you = 0;  // ✅ reset flag
        }
        sleep(1);
    }

    if (shmdt(shared_memory) == -1)
    {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}

//shm2.o
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

int main()
{
    int running = 1;
    void *shared_memory = NULL;
    struct shared_use_st *shared_stuff;
    char buffer[BUFSIZ];
    int shmid;

    shmid = shmget((key_t)1234, sizeof(struct shared_use_st),
                   0666 | IPC_CREAT);

    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }

    printf("Shared memory id = %d\n", shmid);

    shared_memory = shmat(shmid, NULL, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written_by_you = 0;   // ✅ IMPORTANT

    while (running) {
        while (shared_stuff->written_by_you == 1) {
            sleep(1);
            printf("Waiting for client...\n");
        }

        printf("Enter some text: ");
        fgets(buffer, BUFSIZ, stdin);

        buffer[strcspn(buffer, "\n")] = '\0'; // remove newline

        strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
        shared_stuff->written_by_you = 1;

        if (strncmp(buffer, "end", 3) == 0) {
            running = 0;
        }
    }

    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    // Remove shared memory
    shmctl(shmid, IPC_RMID, 0);

    exit(EXIT_SUCCESS);
}

```
## Write a C program that illustrates two processes communicating using shared memory.


<img width="358" height="222" alt="image" src="https://github.com/user-attachments/assets/4bf561cc-d361-409b-820f-521f835f3c06" />

<img width="427" height="204" alt="image" src="https://github.com/user-attachments/assets/ae4cca5b-995c-4b32-8eb8-9e515cc25e92" />

## OUTPUT

# RESULT:
The program is executed successfully.
