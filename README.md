# Starve Free Readers-Writers Problem
This problem deals with multiple processes reading from and writing to a shared resource synchronously with neither the readers or writers getting starved indefinitely. 

### Solution
All pseudocode is in C++-like syntax. 

We design a semaphore that handles processes in a First-In-First-Out order. The processes are given access to the semaphore in the order they called wait. Each process gets blocked and pushed to the queue when it calls wait and is then woken up when it reaches the head of the queue. The following are the pseudo code for the FIFO queue and the semaphore that utilizes the queue for resource allocation. 

```cpp

//This represents a process in the FIFO queue. 
Struct Proc {
    Proc* next;
    int process_id;
}

//Implementation of a FIFO queue of Proc nodes. 
struct Queue {
    Proc* head, back;
    
   	void push(int pid) {
        Proc* proc = new Proc();
        proc->process_id = pid;
        if(back == NULL) {
            head = proc;
            back = proc; 
        } else {
            back->next = proc;
            back = proc;
        }
    }
    
    int pop() {
        if(head == NULL) {
            return -1; // underflow 
        } else {
            int pid = head->process_id;
            head = head->next;
            if(head == NULL) {
                back = NULL;
            }
            
            return pid;
        }
    }
}

struct Semaphore {
    int value;
    Queue* queue = new Queue();
    
    void wait(int pid) {
        value--;
        if(value < 0) {
            queue->push(pid);
            block(pid); //block the process until wake is called. 
        }
    }
    
    void signal() {
        value++;
        if(value <= 0) {
            int pid = queue->pop();
            wake(pid); //wake the process at the head of the queue. 
        }
    }
}
```

We declare the following global variables

```cpp 
Semaphore* in, out, write; 
in->value = 1; 
out->value = 1; 
write->value = 0; 

int in_ctr = 0; // #readers started reading
int out_ctr = 0; // #readers completed reading
bool wait_write = false; // true if a writer is waiting 
```

Then, the reader and writer processes are as follows  
Reader Process
```cpp 
// Assume the process_id is the process id
in->wait(process_id); // Call wait 
in_ctr++;             // increment in_ctr as the process starts reading
in->signal();         

//Read the data(Critical section)

out->wait(process_id); // wait on the out semaphore 
out_ctr++;             // increment out_ctr after reading complete
if(in_ctr == out_ctr && wait_write) {   // if writer is waiting and 
    write->signal();                    // this is last reader then 
}                                       // signal write semaphore
out->signal(); 
```

Writer Process
```cpp 
//Assume process_id is the process id
in->wait(process_id); 
out->wait(process_id); 
if(in_ctr == out_ctr) {   // if no processes are currently reading
    out->signal(); 
} else {                  // wait for readers to finish
    wait_write = true; 
    out->signal(); 
    writer->wait(); 
    wait_write = false; 
}

//Write the data(Critical section)

in->signal(); 
```

Multiple readers can simultaneously read the resource. Once a writer has entered the waiting list, no new process can begin reading before the writer gets access to the resource.  
A writer lets its presence known by setting `wait_write = true`. It ensures no new process can begin reading by using the `in` semaphore. Hence writers do not starve in this scheme.  
Suppose another writer arrives before the first one has completed. It will get queued up in the `in` semaphore. Since the semaphore is FIFO, any readers that arrived after the first writer and before the second one, will be executed before the second writer. This is ensured in the FIFO nature of the queues. Hence readers also do not starve in this scheme.  
Hence this is a valid solution to the starve-free-readers-writers problem. 

### References

- Operating Systems Concepts (9th Edition), Abraham Silberchatz, Peter B Galvin and Greg Gagne
- [arXiv:1309.4507](https://arxiv.org/abs/1309.4507)
