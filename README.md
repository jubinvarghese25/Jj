# Program 
#include <stdio.h>
#include <limits.h>

struct process {
    int pid, at, bt, rt, pr, ct, wt;
};

int n;
struct process p[10], temp;

/* FCFS */
float fcfs() {
    int time = 0;
    float avg = 0;

    for(int i=0;i<n;i++) {
        if(time < p[i].at)
            time = p[i].at;
        p[i].wt = time - p[i].at;
        time += p[i].bt;
        avg += p[i].wt;
    }
    return avg/n;
}

/* SRTF */
float srtf() {
    int time = 0, completed = 0, min_rt, idx;
    float avg = 0;

    for(int i=0;i<n;i++)
        p[i].rt = p[i].bt;

    while(completed < n) {
        min_rt = INT_MAX;
        idx = -1;

        for(int i=0;i<n;i++) {
            if(p[i].at <= time && p[i].rt > 0 && p[i].rt < min_rt) {
                min_rt = p[i].rt;
                idx = i;
            }
        }

        if(idx == -1) {
            time++;
            continue;
        }

        p[idx].rt--;
        time++;

        if(p[idx].rt == 0) {
            completed++;
            p[idx].ct = time;
            p[idx].wt = p[idx].ct - p[idx].at - p[idx].bt;
            avg += p[idx].wt;
        }
    }
    return avg/n;
}

/* Non-Preemptive Priority */
float priority_np() {
    int time = 0, completed = 0, idx;
    float avg = 0;
    int done[10] = {0};

    while(completed < n) {
        idx = -1;
        int maxp = -1;

        for(int i=0;i<n;i++) {
            if(p[i].at <= time && !done[i] && p[i].pr > maxp) {
                maxp = p[i].pr;
                idx = i;
            }
        }

        if(idx == -1) {
            time++;
            continue;
        }

        p[idx].wt = time - p[idx].at;
        time += p[idx].bt;
        avg += p[idx].wt;
        done[idx] = 1;
        completed++;
    }
    return avg/n;
}

/* Round Robin */
float rr() {
    int tq = 3, time = 0, completed = 0;
    float avg = 0;

    for(int i=0;i<n;i++)
        p[i].rt = p[i].bt;

    while(completed < n) {
        for(int i=0;i<n;i++) {
            if(p[i].rt > 0 && p[i].at <= time) {
                int exec = (p[i].rt > tq) ? tq : p[i].rt;
                p[i].rt -= exec;
                time += exec;

                if(p[i].rt == 0) {
                    p[i].ct = time;
                    p[i].wt = p[i].ct - p[i].at - p[i].bt;
                    avg += p[i].wt;
                    completed++;
                }
            }
        }
    }
    return avg/n;
}

int main() {
    float a1, a2, a3, a4;

    printf("Enter number of processes: ");
    scanf("%d", &n);

    for(int i=0;i<n;i++) {
        p[i].pid = i+1;
        printf("P%d Arrival Burst Priority: ", i+1);
        scanf("%d %d %d", &p[i].at, &p[i].bt, &p[i].pr);
    }

    a1 = fcfs();
    a2 = srtf();
    a3 = priority_np();
    a4 = rr();

    printf("\nAverage Waiting Times:");
    printf("\nFCFS = %.2f", a1);
    printf("\nSRTF = %.2f", a2);
    printf("\nPriority = %.2f", a3);
    printf("\nRound Robin = %.2f", a4);

    float min = a1;
    if(a2 < min) min = a2;
    if(a3 < min) min = a3;
    if(a4 < min) min = a4;

    printf("\n\nMinimum Average Waiting Time = %.2f\n", min);
    return 0;
}
