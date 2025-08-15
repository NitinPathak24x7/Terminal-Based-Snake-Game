#include <ncurses.h>
#include <stdlib.h>
#include <unistd.h>
#include <time.h>

#define WIDTH  40
#define HEIGHT 20
#define DELAY 100000 // microseconds

typedef struct SnakeSegment {
    int x, y;
    struct SnakeSegment *next;
} SnakeSegment;

SnakeSegment *snake = NULL;
int food_x, food_y;
int dir_x = 1, dir_y = 0; // start moving right
int score = 0;

void init_game() {  
    snake = malloc(sizeof(SnakeSegment));
    snake->x = WIDTH / 2;
    snake->y = HEIGHT / 2;
    snake->next = NULL;
    srand(time(NULL));
    food_x = rand() % WIDTH;
    food_y = rand() % HEIGHT;
}

void end_game() {
    SnakeSegment *current = snake;
    while (current) {
        SnakeSegment *tmp = current;
        current = current->next;
        free(tmp);
    }
    endwin();
printf("Game Over! Final Score: %d\n", score);
    exit(0);
}

void draw_game() {
    clear();
    // draw border
    for (int i = 0; i < WIDTH + 2; i++) {
        mvprintw(0, i, "#");
        mvprintw(HEIGHT + 1, i, "#");
    }
    for (int i = 1; i <= HEIGHT; i++) {
        mvprintw(i, 0, "#");
        mvprintw(i, WIDTH + 1, "#");
    }

    // draw snake
    SnakeSegment *current = snake;
    while (current) {
        mvprintw(current->y + 1, current->x + 1, "O");
        current = current->next;
    }

    // draw food
    mvprintw(food_y + 1, food_x + 1, "*");

    // draw score
    mvprintw(HEIGHT + 3, 0, "Score: %d", score);
    refresh();
}

void move_snake() {
    int new_x = snake->x + dir_x;
    int new_y = snake->y + dir_y;

    // collision with wall
    if (new_x < 0 || new_x >= WIDTH || new_y < 0 || new_y >= HEIGHT)
        end_game();

    // collision with self
    SnakeSegment *curr = snake;
    while (curr) {
        if (curr->x == new_x && curr->y == new_y)
            end_game();
        curr = curr->next;
    }

    // create new head
    SnakeSegment *new_head = malloc(sizeof(SnakeSegment));
    new_head->x = new_x;
    new_head->y = new_y;
    new_head->next = snake;
    snake = new_head;

    // check food collision
    if (new_x == food_x && new_y == food_y) {
        score++;
        food_x = rand() % WIDTH;
        food_y = rand() % HEIGHT;
    } else {
        // remove tail
        SnakeSegment *prev = NULL;
        SnakeSegment *tail = snake;
        while (tail->next) {
            prev = tail;
            tail = tail->next;
        }
        free(tail);
        if (prev)
            prev->next = NULL;
    }
}

void handle_input() {
    int ch = getch();
    switch (ch) {
        case KEY_UP:    if (dir_y != 1) { dir_x = 0; dir_y = -1; } break;
        case KEY_DOWN:  if (dir_y != -1){ dir_x = 0; dir_y = 1; }  break;
        case KEY_LEFT:  if (dir_x != 1) { dir_x = -1; dir_y = 0; } break;
        case KEY_RIGHT: if (dir_x != -1){ dir_x = 1; dir_y = 0; }  break;
        case 'q': end_game(); break;
    }
}

int main() {
    initscr();
    noecho();
    curs_set(FALSE);
    keypad(stdscr, TRUE);
    nodelay(stdscr, TRUE); // non-blocking input

    init_game();

    while (1) {
        handle_input();
        move_snake();
        draw_game();
        usleep(DELAY);
    }

    endwin();
    return 0;
}
