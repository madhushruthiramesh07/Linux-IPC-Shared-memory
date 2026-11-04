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
```

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/wait.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written;
    char some_text[TEXT_SZ];
};

int main(void) {
    int shmid;
    void *shared_memory;
    struct shared_use_st *shared_stuff;

    shmid = shmget(IPC_PRIVATE, sizeof(struct shared_use_st),
                   0666 | IPC_CREAT);
    if (shmid == -1) {
        perror("shmget failed");
        return 1;
    }

    printf("Shared memory ID = %d\n", shmid);

    shared_memory = shmat(shmid, NULL, 0);
    if (shared_memory == (void *)-1) {
        perror("shmat failed");
        shmctl(shmid, IPC_RMID, NULL);
        return 1;
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written = 0;

    pid_t pid = fork();

    if (pid < 0) {
        perror("fork failed");
        shmdt(shared_memory);
        shmctl(shmid, IPC_RMID, NULL);
        return 1;
    }

    if (pid == 0) {
        /* Child process: Consumer */
        while (1) {
            while (shared_stuff->written == 0)
                usleep(10000);

            printf("Consumer received: %s",
                   shared_stuff->some_text);
            fflush(stdout);

            if (strncmp(shared_stuff->some_text, "end", 3) == 0)
                break;

            shared_stuff->written = 0;
        }

        shmdt(shared_memory);
        _exit(0);
    } else {
        /* Parent process: Producer */
        char buffer[TEXT_SZ];

        while (1) {
            printf("Enter some text (type end to stop): ");
            fflush(stdout);

            if (fgets(buffer, sizeof(buffer), stdin) == NULL) {
                strcpy(buffer, "end\n");
            }

            snprintf(shared_stuff->some_text, TEXT_SZ, "%s", buffer);
            shared_stuff->written = 1;

            printf("Producer sent: %s", shared_stuff->some_text);

            if (strncmp(buffer, "end", 3) == 0)
                break;

            while (shared_stuff->written == 1)
                usleep(10000);
        }

        waitpid(pid, NULL, 0);

        shmdt(shared_memory);
        shmctl(shmid, IPC_RMID, NULL);

        printf("Shared memory removed.\n");
    }

    return 0;
}



```
## OUTPUT
<img width="492" height="286" alt="image" src="https://github.com/user-attachments/assets/dcf31f1f-c711-4863-bf00-721ab2f00506" />


# RESULT:
The program is executed successfully.
