# Linux-IPC--Pipes
Linux-IPC-Pipes


# Ex03-Linux IPC - Pipes

# AIM:
To write a C program that illustrate communication between two process using unnamed and named pipes

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - pipe(), fifo()

### Step 3:

Testing the C Program for the desired output. 

# PROGRAM:

## C Program that illustrate communication between two process using unnamed pipes using Linux API system calls
~~~
// C program to illustrate communication between two processes using unnamed pipes
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/wait.h>

void server(int rfd, int wfd);
void client(int wfd, int rfd);

int main()
{
    int p1[2], p2[2];
    pid_t pid;

    if (pipe(p1) == -1 || pipe(p2) == -1) {
        perror("pipe");
        return 1;
    }

    pid = fork();

    if (pid < 0) {
        perror("fork");
        return 1;
    }

    if (pid == 0) {
        close(p1[1]);
        close(p2[0]);
        server(p1[0], p2[1]);
        close(p1[0]);
        close(p2[1]);
        return 0;
    }

    close(p1[0]);
    close(p2[1]);
    client(p1[1], p2[0]);
    close(p1[1]);
    close(p2[0]);

    wait(NULL);
    return 0;
}

void client(int wfd, int rfd)
{
    int n;
    char fname[2000];
    char buff[2000];

    printf("ENTER THE FILE NAME: ");
    scanf("%1999s", fname);

    printf("CLIENT SENDING THE REQUEST...\nPLEASE WAIT\n");
    sleep(2);

    write(wfd, fname, strlen(fname) + 1);

    n = read(rfd, buff, sizeof(buff) - 1);
    if (n > 0) {
        buff[n] = '\0';
        printf("THE RESULTS OF CLIENT ARE:\n");
        write(1, buff, n);
    } else {
        printf("No data received from server.\n");
    }
}

void server(int rfd, int wfd)
{
    int n;
    char fname[2000];
    char buff[2000];

    n = read(rfd, fname, sizeof(fname) - 1);
    if (n <= 0) {
        write(wfd, "Invalid request", 15);
        return;
    }

    fname[n] = '\0';

    int fd = open(fname, O_RDONLY);
    sleep(2);

    if (fd < 0) {
        write(wfd, "can't open file", 15);
    } else {
        n = read(fd, buff, sizeof(buff) - 1);
        if (n > 0) {
            write(wfd, buff, n);
        } else {
            write(wfd, "file is empty or read error", 27);
        }
        close(fd);
    }
}
~~~




## OUTPUT
<img width="1020" height="765" alt="OS-3" src="https://github.com/user-attachments/assets/7f20cfab-3fb7-476d-92ef-bd56dbe54d6a" />



## C Program that illustrate communication between two process using named pipes using Linux API system calls
~~~
//C Program that illustrate communication between two process using named pipes
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <sys/wait.h>
#include <errno.h>

#define FIFO_FILE "/tmp/my_fifo"
#define FILE_NAME "hello.txt"

void server(void);
void client(void);

int main(void) {
    pid_t pid;

    if (mkfifo(FIFO_FILE, 0666) == -1) {
        if (errno != EEXIST) {
            perror("mkfifo");
            exit(EXIT_FAILURE);
        }
    }

    pid = fork();

    if (pid < 0) {
        perror("fork failed");
        unlink(FIFO_FILE);
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {
        client();
        exit(EXIT_SUCCESS);
    } else {
        sleep(1);
        server();
        wait(NULL);
        unlink(FIFO_FILE);
    }

    return 0;
}

void server(void) {
    int fifo_fd, file_fd;
    char buffer[1024];
    ssize_t bytes_read, bytes_written;

    file_fd = open(FILE_NAME, O_RDONLY);
    if (file_fd == -1) {
        perror("Error opening hello.txt");
        exit(EXIT_FAILURE);
    }

    fifo_fd = open(FIFO_FILE, O_WRONLY);
    if (fifo_fd == -1) {
        perror("Error opening FIFO for writing");
        close(file_fd);
        exit(EXIT_FAILURE);
    }

    while ((bytes_read = read(file_fd, buffer, sizeof(buffer))) > 0) {
        bytes_written = write(fifo_fd, buffer, bytes_read);
        if (bytes_written == -1) {
            perror("Error writing to FIFO");
            close(file_fd);
            close(fifo_fd);
            exit(EXIT_FAILURE);
        }
    }

    if (bytes_read == -1) {
        perror("Error reading file");
    }

    close(file_fd);
    close(fifo_fd);
}

void client(void) {
    int fifo_fd;
    char buffer[1024];
    ssize_t bytes_read;

    fifo_fd = open(FIFO_FILE, O_RDONLY);
    if (fifo_fd == -1) {
        perror("Error opening FIFO for reading");
        exit(EXIT_FAILURE);
    }

    while ((bytes_read = read(fifo_fd, buffer, sizeof(buffer))) > 0) {
        if (write(STDOUT_FILENO, buffer, bytes_read) == -1) {
            perror("Error writing to stdout");
            close(fifo_fd);
            exit(EXIT_FAILURE);
        }
    }

    if (bytes_read == -1) {
        perror("Error reading from FIFO");
    }

    close(fifo_fd);
}
~~~
## OUTPUT
<img width="1020" height="765" alt="OS-3 1" src="https://github.com/user-attachments/assets/5a814fb8-35e9-4142-b6f2-0c1ab7f94414" />



# RESULT:
The program is executed successfully.
