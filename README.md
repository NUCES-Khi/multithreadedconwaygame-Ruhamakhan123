#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

int rows, cols;
int** grid;
int** new_grid;
pthread_mutex_t mutex;


void init_grid() {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            new_grid[i][j] = 0;
        }
    }
}


void print_grid(int** grid) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%d ", grid[i][j]);
        }
        printf("\n");
    }
    printf("\n");
}


int count_neighbors(int row, int col) {
    int count = 0;
    for (int i = row - 1; i <= row + 1; i++) {
        for (int j = col - 1; j <= col + 1; j++) {
            if (i >= 0 && i < rows && j >= 0 && j < cols && !(i == row && j == col)) {
                count += grid[i][j];
            }
        }
    }
    return count;
}


void* update_row(void* arg) {
    
    int row = *((int*) arg);
    pthread_mutex_lock(&mutex);
    for (int col = 0; col < cols; col++) {
        int neighbors = count_neighbors(row, col);
        if (grid[row][col] == 1 && (neighbors == 2 || neighbors == 3)) {
            new_grid[row][col] = 1;
        } else if (grid[row][col] == 0 && neighbors == 3) {
            new_grid[row][col] = 1;
        }
    }
    pthread_mutex_unlock(&mutex);
    pthread_exit(NULL);
}


int main() {
    int generations;
    printf("Enter the number of rows: ");
    scanf("%d", &rows);
    printf("Enter the number of columns: ");
    scanf("%d", &cols);
    grid = (int**) malloc(rows * sizeof(int*));
    new_grid = (int**) malloc(rows * sizeof(int*));
    for (int i = 0; i < rows; i++) {
        grid[i] = (int*) malloc(cols * sizeof(int));
        new_grid[i] = (int*) malloc(cols * sizeof(int));
    }
    int tid[rows];
    for (int i = 0; i < rows; i++) {
        tid[i] = i;
    }
    printf("Enter the initial grid:\n");
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            scanf("%d", &grid[i][j]);
        }
    }
    printf("Enter the number of generations: ");
    scanf("%d", &generations);
    pthread_t threads[rows];
    pthread_mutex_init(&mutex, 0);
    printf("Initial grid:\n");
    print_grid(grid);
    for (int i = 0; i < generations; i++) {
        init_grid();
        for (int j = 0; j < rows; j++) {
pthread_create(&threads[j], NULL, update_row, &tid[j]);
}
    
    for (int j = 0; j < rows; j++) {
        pthread_join(threads[j], NULL);
    }
    
    
    for (int j = 0; j < rows; j++) {
        for (int k = 0; k < cols; k++) {
            grid[j][k] = new_grid[j][k];
        }
    }
    
    
    printf("Generation %d:\n", i + 1);
    print_grid(grid);
}


pthread_mutex_destroy(&mutex);
free(grid);
free(new_grid);
return 0;
}

![image](https://user-images.githubusercontent.com/105592893/236697417-8cb5e0b3-9609-4516-8d01-332d13936849.png)


![image](https://user-images.githubusercontent.com/105592893/236698824-f8657440-4573-416a-bdfb-cf31d1b20efa.png)
![image](https://user-images.githubusercontent.com/105592893/236699187-87489e11-b6ce-41c2-b3b4-64e65b4385a8.png)
![image](https://user-images.githubusercontent.com/105592893/236699477-901c7555-456f-438d-9d0c-b1290fd7aaca.png)


Implementation and Design Decisions:
The program simulates Conway's Game of Life, where each cell in a grid is either alive (represented by 1) or dead (represented by 0). The program uses multi-threading with pthreads to update each row of the grid in parallel, and a mutex is used to synchronize access to the grid. The implementation uses integer arrays to represent the grid data structure, and a separate function is created to calculate the number of live neighbors of a cell.

Regarding the design decisions, the choice of using integer arrays allows for efficient memory usage and straightforward implementation of the game's rules. Additionally, creating a separate function to count the number of neighbors makes the code more modular and easier to read.

In terms of the speedup measurements, the program's performance can be evaluated using metrics such as execution time or throughput. These metrics can be measured by running the program with different thread counts and comparing the results. By doing so, the optimal thread count can be determined to achieve the best performance for the specific hardware being used.
