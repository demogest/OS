//  Progress scheduler, based on HRRN algorithm  (Highest Response Ratio Next)
//  1. The process with the highest response ratio is selected to run next.
//  2. The response ratio of a process is the ratio of its waiting time to its service time.
//  3. The waiting time of a process is the time that has elapsed since the process arrived and the process has not yet completed.
//  4. The service time of a process is the time required for the CPU to complete the process.
//  5. The response ratio of a process is calculated as: (waiting time + burst time) / burst time.
//  author: xuzhekai
//  time: 2022-12-9

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define MAX 100

// define the process structure
typedef struct process
{
    int pid;             // process id
    int priority;        // priority, 0 is the highest priority
    int arrival_time;    // arrival time
    int burst_time;      // burst time
    int response_ratio;  // response ratio
    int waiting_time;    // waiting time
    int turnaround_time; // turnaround time
    int remaining_time;  // remaining time
    int resoure_id;      // resource id
    int completed;       // whether the process is completed
} Process;

// define the process node structure
typedef struct process_node
{
    Process *process;          // process
    struct process_node *next; // next node
} ProcessNode;

// define the process queue structure
typedef struct process_queue
{
    ProcessNode *front; // front pointer
    ProcessNode *rear;  // rear pointer
    int size;           // queue size
} ProcessQueue;

// Queue operations
ProcessQueue *create_queue()
{
    ProcessQueue *queue = (ProcessQueue *)malloc(sizeof(ProcessQueue));
    queue->front = NULL;
    queue->rear = NULL;
    queue->size = 0;
    return queue;
}

// add a process to the queue
void enqueue(ProcessQueue *queue, Process *process)
{
    ProcessNode *node = (ProcessNode *)malloc(sizeof(ProcessNode));
    node->process = process;
    node->next = NULL;
    if (queue->rear == NULL)
    {
        queue->front = node;
        queue->rear = node;
    }
    else
    {
        queue->rear->next = node;
        queue->rear = node;
    }
    queue->size++;
}

// remove a process from the queue
Process *dequeue(ProcessQueue *queue, int pid)
{
    if (queue->front == NULL)
    {
        return NULL;
    }
    ProcessNode *node = queue->front;
    Process *process = node->process;
    if (node->process->pid == pid)
    {
        queue->front = node->next;
        if (queue->front == NULL)
        {
            queue->rear = NULL;
        }
        queue->size--;
        free(node);
        return process;
    }
    while (node->next != NULL)
    {
        if (node->next->process->pid == pid)
        {
            ProcessNode *temp = node->next;
            process = temp->process;
            node->next = temp->next;
            if (node->next == NULL)
            {
                queue->rear = node;
            }
            queue->size--;
            free(temp);
            return process;
        }
        node = node->next;
    }
    return NULL;
}

// get the front process from the queue
Process *front(ProcessQueue *queue)
{
    if (queue->front == NULL)
    {
        return NULL;
    }
    return queue->front->process;
}

// get the rear process from the queue
Process *rear(ProcessQueue *queue)
{
    if (queue->rear == NULL)
    {
        return NULL;
    }
    return queue->rear->process;
}

// get the size of the queue
int size(ProcessQueue *queue)
{
    return queue->size;
}

// get the sieze of the running queue
int running_size(Process **cores, int numCores)
{
    int size = 0;
    for (int i = 0; i < numCores; i++)
    {
        size += cores[i] == NULL ? 0 : 1;
    }
    return size;
}

// check whether the queue is empty
int is_empty(ProcessQueue *queue)
{
    return queue->size == 0;
}

// quick sort the process array by arrival time
void quick_sort(Process **processes, int left, int right)
{
    if (left >= right)
    {
        return;
    }
    int i = left;
    int j = right;
    Process *temp = processes[left];
    while (i < j)
    {
        while (i < j && processes[j]->arrival_time >= temp->arrival_time)
        {
            j--;
        }
        processes[i] = processes[j];
        while (i < j && processes[i]->arrival_time <= temp->arrival_time)
        {
            i++;
        }
        processes[j] = processes[i];
    }
    processes[i] = temp;
    quick_sort(processes, left, i - 1);
    quick_sort(processes, i + 1, right);
}

// check if all cores are busy
int is_all_busy(Process **cores, int numCores)
{
    for (int i = 0; i < numCores; i++)
    {
        if (cores[i] == NULL)
        {
            return 0;
        }
    }
    return 1;
}

// Sort the waiting queue by response ratio and priority
void sort_by_priority(ProcessQueue *queue)
{
    ProcessNode *node = queue->front;
    while (node != NULL)
    {
        ProcessNode *temp = node->next;
        while (temp != NULL)
        {
            if (node->process->response_ratio < temp->process->response_ratio)
            {
                Process *process = node->process;
                node->process = temp->process;
                temp->process = process;
            }
            else if (node->process->response_ratio == temp->process->response_ratio)
            {
                if (node->process->priority > temp->process->priority)
                {
                    Process *process = node->process;
                    node->process = temp->process;
                    temp->process = process;
                }
            }
            temp = temp->next;
        }
        node = node->next;
    }
}

int main()
{
    int numProcesses; // number of processes
    int numCores;     // number of cores
    int numResources; // number of resources
    int nowtime = 0;  // current time
    // input the number of processes, cores and resources
    printf("Please input the number of processes, cores and resources: ");
    scanf("%d %d %d", &numProcesses, &numCores, &numResources);
    // input the processes information (priority, arrival time, burst time, resource id)
    Process **processes = (Process **)malloc(sizeof(Process *) * numProcesses);
    for (int i = 0; i < numProcesses; i++)
    {
        processes[i] = (Process *)malloc(sizeof(Process));
        processes[i]->pid = i;
        printf("Please input the priority, arrival time, burst time and resource id of process %d: ", i);
        scanf("%d %d %d %d", &processes[i]->priority, &processes[i]->arrival_time, &processes[i]->burst_time, &processes[i]->resoure_id);
        processes[i]->remaining_time = processes[i]->burst_time;
        processes[i]->completed = 0;
    }
    // sort the processes by arrival time
    quick_sort(processes, 0, numProcesses - 1);
    // Initialize the resource array
    int *resources = (int *)malloc(sizeof(int) * numResources);
    for (int i = 0; i < numResources; i++)
    {
        resources[i] = 0;
    }
    // Initialize the waiting queue
    ProcessQueue *waiting_queue = create_queue();
    // Initialize the completed queue
    ProcessQueue *completed_queue = create_queue();
    // Initialize the running process array
    Process **running_processes = (Process **)malloc(sizeof(Process *) * numCores);
    for (int i = 0; i < numCores; i++)
    {
        running_processes[i] = NULL;
    }
    // repeat until all processes are completed
    int arrived = 0; // number of arrived processes
    while (size(completed_queue) < numProcesses)
    {
        printf("Current time: %d\n", nowtime);
        // update the remaining time of each process on running cores
        for (int i = 0; i < numCores; i++)
        {
            if (running_processes[i] != NULL)
            {
                running_processes[i]->remaining_time--;
                // if the process is completed
                if (running_processes[i]->remaining_time == 0)
                {
                    // set the resource to available
                    resources[running_processes[i]->resoure_id] = 0;
                    // set the process to completed
                    running_processes[i]->completed = 1;
                    // update waiting time and turnaround time
                    running_processes[i]->waiting_time = nowtime - running_processes[i]->arrival_time - running_processes[i]->burst_time;
                    running_processes[i]->turnaround_time = nowtime - running_processes[i]->arrival_time;
                    // add the process to the completed queue
                    enqueue(completed_queue, running_processes[i]);
                    // print the completed information
                    printf("Process %d is completed at time %d\n", running_processes[i]->pid, nowtime);
                    // remove the process from the running process array
                    running_processes[i] = NULL;
                }
            }
        }
        // add the processes that have arrived to the waiting queue
        for (int i = 0; i < numProcesses - arrived; i++)
        {
            if (processes[i]->arrival_time <= nowtime && processes[i]->completed == 0)
            {
                enqueue(waiting_queue, processes[i]);
                // Remove the process from the processes array
                for (int j = i; j < numProcesses - 1; j++)
                {
                    processes[j] = processes[j + 1];
                }
                arrived++;
                i--;
            }
        }
        // select the process with the highest priority
        // if there are multiple processes with the same priority, select the process with the highest response ratio
        Process *selected_process = NULL; // the selected process
        // sort the waiting queue with the highest priority
        sort_by_priority(waiting_queue);
        // select a process which requires an available resource
        ProcessNode *node = waiting_queue->front;
        while (node != NULL)
        {
            if (resources[node->process->resoure_id] == 0)
            {
                selected_process = node->process;
                break;
            }
            node = node->next;
        }
        // if there is a selected process and exists an available core and an available resource
        if (selected_process != NULL && !is_all_busy(running_processes, numCores))
        {
            dequeue(waiting_queue, selected_process->pid);
            // select the first available core
            int core_id = 0;
            while (running_processes[core_id] != NULL)
                core_id++;
            // run the process on the core
            running_processes[core_id] = selected_process;
            // set the resource to busy
            resources[selected_process->resoure_id] = 1;
            // print the running information
            printf("Process %d is running on core %d at time %d\n", selected_process->pid, core_id, nowtime);
        }
        else if (running_size(running_processes, numCores) > 0)
        {
            printf("Process:");
            for (int i = 0; i < numCores; i++)
            {
                if (running_processes[i] != NULL)
                {
                    printf(" %d", running_processes[i]->pid);
                }
            }
            printf(" are running at time %d\n", nowtime);
        }
        else
        {
            printf("No process is running at time %d\n", nowtime);
        }

        nowtime++;
    }
    // print the waiting time and turnaround time of each process
    printf("PID\tArrival Time\tBurst Time\tWaiting Time\tTurnaround Time\tPriority\tResource ID\n");
    ProcessNode *node = completed_queue->front;
    double total_waiting_time = 0;
    double total_turnaround_time = 0;
    while (node != NULL)
    {
        Process *process = node->process;
        printf("%d\t%d\t\t%d\t\t%d\t\t%d\t\t%d\t\t%d\n", process->pid, process->arrival_time, process->burst_time, process->waiting_time, process->turnaround_time, process->priority, process->resoure_id);
        total_turnaround_time += process->turnaround_time;
        total_waiting_time += process->waiting_time;
        node = node->next;
    }
    // print the average waiting time and turnaround time
    printf("Average waiting time: %.2f\nAverage turnaround time: %.2f\n", total_waiting_time / numProcesses, total_turnaround_time / numProcesses);
    return 0;
}
