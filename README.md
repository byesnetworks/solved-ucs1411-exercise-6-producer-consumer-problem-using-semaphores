Download Link: https://assignmentchef.com/product/solved-ucs1411-exercise-6-producer-consumer-problem-using-semaphores
<br>
<u>Study the following system calls</u>

Semaphores – sem_init, sem_wait, sem_post, sem_destroy – POSIX, pthread                        semget , semctl, semop ( BSD – for your understanding)

Shared memory – shmget, shmat, shmdt, shmctl

<u>Assignment 1:</u>

<u>Aim:</u>

To write a C program to create parent/child processes to implement the producer/consumer problem using semaphores in pthread library.

<u>Procedure:</u>

<ol>

 <li>Create a Shared memory for buffer and semaphores – empty, full, mutex</li>

 <li>Create a parent and a child process one acting as a producer and the other consumer.</li>

 <li>In the producer process, produce an item, place it in the buffer. Increment full and decrement empty using wait and signal operations appropriately.</li>

 <li>In the consumer process, consume an item from the buffer and display it on the terminal. Increment empty and decrement full using wait and signal operations appropriately.</li>

 <li>Compile the sample program with pthread library   cc prg.c – lpthread</li>

</ol>

<u>Assignment 2:</u>

Modify the program as separate client / server process programs to generate ‘N’ random numbers in producer and write them into shared memory. Consumer process should read them from shared memory and display them in terminal




<u>Sample Program:</u>

#include &lt;stdio.h&gt;

#include &lt;stdlib.h&gt;

#include &lt;string.h&gt;

#include &lt;semaphore.h&gt;

#include &lt;pthread.h&gt; // for semaphore operations sem_init,sem_wait,sem_post

#include &lt;sys/ipc.h&gt;

#include &lt;sys/shm.h&gt;

#include &lt;sys/sem.h&gt;

#include &lt;sys/wait.h&gt;

#include &lt;sys/errno.h&gt; #include &lt;sys/types.h&gt; extern int errno;

#define SIZE 10 /* size of the shared buffer*/

#define VARSIZE 1 /* size of shared variable=1byte*/

#define INPUTSIZE 20

#define SHMPERM 0666 /* shared memory permissions */

int segid; /* id for shared memory bufer */ int empty_id; int full_id; int mutex_id; char * buff; char * input_string; sem_t *empty; sem_t *full; sem_t *mutex;

int p=0,c=0;

//

// Producer function

//

void produce()

{

int i=0;             while (1)

{

if(i&gt;=strlen(input_string))

{

printf(“
 Producer %d exited 
”,getpid());

wait(NULL);

exit(1);

}

printf(“
Producer %d trying to aquire Semaphore Empty 
”,getpid());

sem_wait(empty);

printf(“
Producer %d successfully aquired Semaphore Empty 
”,getpid());

printf(“
Producer %d trying to aquire Semaphore Mutex 
”,getpid());

sem_wait(mutex);

printf(“
Producer %d successfully aquired Semaphore Mutex 
”,getpid());                           buff[p]=input_string[i];

printf(“
Producer %d Produced Item [ %c ] 
”,getpid(),input_string[i]);

i++;

p++;

printf(“
Items in Buffer %d 
”,p);                 sem_post(mutex);

printf(“
Producer %d released Semaphore Mutex 
”,getpid());

sem_post(full);

printf(“
Producer %d released Semaphore Full 
”,getpid());                         sleep(2/random());

} //while

} //producer fn

//

// Consumer function

//

void consume()

{

int i=0;             while (1)

{

if(i&gt;=strlen(input_string))

{

printf(“
 Consumer %d exited 
”,getpid());

exit(1);

}

printf(“
Consumer %d trying to aquire Semaphore Full 
”,getpid());

sem_wait(full);

printf(“
Consumer %d successfully aquired Semaphore Full 
”,getpid());

printf(“
Consumer %d trying to aquire Semaphore Mutex 
”,getpid());

sem_wait(mutex);

printf(“
Consumer %d successfully aquired Semaphore Mutex
”,getpid());                          printf(“
Consumer %d Consumed Item [ %c ] 
”,getpid(),buff[c]);

buff[c]=’ ‘;

c++;

printf(“
Items in Buffer %d 
”,strlen(input_string)c);

i++;

sem_post(mutex);

printf(“
Consumer %d released Semaphore Mutex 
”,getpid());

sem_post(empty);

printf(“
Consumer %d released Semaphore Empty 
”,getpid());                   sleep(1);

} //while

} //consumer fn

//————————————————————–

Main function

//————————————————————— int main()

{

int i=0;             pid_t temp_pid;              segid = shmget (IPC_PRIVATE, SIZE, IPC_CREAT | IPC_EXCL | SHMPERM );

empty_id=shmget(IPC_PRIVATE,sizeof(sem_t),IPC_CREAT|IPC_EXCL|

SHMPERM);

full_id=shmget(IPC_PRIVATE,sizeof(sem_t),IPC_CREAT|IPC_EXCL|

SHMPERM);

mutex_id=shmget(IPC_PRIVATE,sizeof(sem_t),IPC_CREAT|IPC_EXCL|

SHMPERM);

buff = shmat( segid, (char *)0, 0 );      empty = shmat(empty_id,(char *)0,0);              full = shmat(full_id,(char *)0,0);         mutex = shmat(mutex_id,(char *)0,0);  // Initializing Semaphores Empty , Full &amp; Mutex

sem_init(empty,1,SIZE);          sem_init(full,1,0);        sem_init(mutex,1,1);              printf(“
 Main Process Started 
”);    printf(“
 Enter the input string (20 characters MAX) : “);

input_string=(char *)malloc(20);         scanf(“%s”,input_string);           printf(“Entered string : %s”,input_string);              temp_pid=fork();

if(temp_pid&gt;0) //parent

{

produce();

}

else //child

{ consume();

}

shmdt(buff);    shmdt(empty);             shmdt(full);              shmdt(mutex);             shmctl(segid, IPC_RMID, NULL);              semctl( empty_id, 0, IPC_RMID, NULL);       semctl( full_id, 0, IPC_RMID, NULL);            semctl( mutex_id, 0, IPC_RMID, NULL);

sem_destroy(empty);  sem_destroy(full);

sem_destroy(mutex);

printf(“
 Main process exited 

”);  return(0);

} //main