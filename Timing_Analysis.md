# Thread_Mutex
//# Threads_Mutex_timimg_analysis
#include <stdio.h>
#include <pthread.h>
#include <sys/time.h>
#include <unistd.h>

int shared_value = 0;
pthread_mutex_t mutex;

void *increment_thread(void *arg) {
    struct timeval tv;
    for (int i = 1; i <= 20; i++) {
        pthread_mutex_lock(&mutex);

        shared_value = i;
        gettimeofday(&tv, NULL);
        printf("T1: Value updated to %d at %ld.%06ld seconds\n",
               shared_value, tv.tv_sec, tv.tv_usec);

        pthread_mutex_unlock(&mutex);

        usleep(5000); // Sleep for 500ms to simulate delay
    }
    pthread_exit(NULL);
}

void *print_thread(void *arg) {
    int last_value = 0;
    struct timeval tv;

    while (1) {
        pthread_mutex_lock(&mutex);

        if (shared_value > last_value) {
            gettimeofday(&tv, NULL);
            printf("________T2: Value read as %d at %ld.%06ld seconds\n",
                   shared_value, tv.tv_sec, tv.tv_usec);
            last_value = shared_value;
        }

        pthread_mutex_unlock(&mutex);
 if (shared_value >= 20) break; // Exit loop when max value is reached
    }
    pthread_exit(NULL);
}

int main() {
    pthread_t thread1, thread2;

    pthread_mutex_init(&mutex, NULL);
    pthread_create(&thread1, NULL, increment_thread, NULL);
    pthread_create(&thread2, NULL, print_thread, NULL);
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);
    pthread_mutex_destroy(&mutex);

    return 0;
}
