# 🟡 Pac-Man Game — Python Pygame Project

Welcome to the **Pac-Man Game Project** — a beginner-to-intermediate Python project using the `pygame` library. In this project, you'll create a simplified version of the iconic arcade game Pac-Man.

---

## 🧠 What You'll Learn

* Working with the `pygame` library
* Handling keyboard input and animations
* Using `if`, `elif`, `else` for control flow
* Collision detection and simple enemy AI
* Score tracking and game over screens

---

## 📜 What It Does

* Displays a maze with walls and collectible dots
* Lets you control Pac-Man with the arrow keys
* Adds collectible dots and scoring
* Moves ghost enemies that roam the maze
* Ends the game if a ghost touches Pac-Man

---

## ▶️ How to Run It

Install the required library:

```bash
pip install pygame
```

Then run the file:

```bash
python pacman_game.py
```

Make sure the script file is saved with a `.py` extension (e.g. `pacman_game.py`).

---

## 💾 Project Code

```python
# Import necessary modules
import pygame
import sys
import random

# Initialize all imported pygame modules
pygame.init()

WIDTH, HEIGHT = 600, 460
# Create the game screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pac-Man")

BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)

# Maze layout: 1 = wall, 0 = dot, 2 = path already visited
maze = [
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
    [1,0,0,0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,0,0,1],
    [1,0,1,1,0,1,1,0,1,1,1,0,1,0,1,1,0,1,1,0,1],
    [1,0,1,1,0,1,1,0,0,0,0,0,1,0,1,1,0,1,1,0,1],
    [1,0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0,0,0,1],
    [1,0,1,1,0,1,0,1,1,0,1,1,1,0,1,0,1,1,0,1,1],
    [1,0,0,0,0,1,0,0,0,0,0,0,1,0,1,0,0,0,0,0,1],
    [1,1,1,0,1,1,0,1,1,0,1,1,1,0,1,1,0,1,1,1,1],
    [1,0,0,0,0,0,0,1,0,0,0,0,0,0,1,0,0,0,0,0,1],
    [1,0,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,0,1],
    [1,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,1],
    [1,0,1,1,1,0,1,1,0,1,1,1,0,1,1,0,1,1,1,0,1],
    [1,0,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,0,1],
    [1,1,0,1,1,1,0,1,1,1,1,1,0,1,1,1,0,1,1,1,1],
    [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
]

ROWS = len(maze)
COLS = len(maze[0])
TILE_SIZE = WIDTH // COLS

# Initial player position
player_pos = [1, 1]
player_speed = 1
# Used to animate Pac-Man's mouth opening/closing
mouth_open = True

ghosts = [
    {"pos": [13, 10], "dir": random.choice([(0,1), (1,0), (0,-1), (-1,0)]), "color": RED},
    {"pos": [7, 9], "dir": random.choice([(0,1), (1,0), (0,-1), (-1,0)]), "color": (255, 102, 255)},
]

# Score tracker
score = 0
font = pygame.font.SysFont(None, 30)
# Controls the frame rate of the game
clock = pygame.time.Clock()

# Draw all the walls and dots based on the maze layout
def draw_maze():
    for row in range(ROWS):
        for col in range(COLS):
            tile = maze[row][col]
            rect = pygame.Rect(col * TILE_SIZE, row * TILE_SIZE, TILE_SIZE, TILE_SIZE)
            if tile == 1:
                pygame.draw.rect(screen, BLUE, rect)
            elif tile == 0:
                pygame.draw.circle(screen, WHITE, rect.center, 4)

# Draw the player character (Pac-Man) with mouth animation
def draw_player():
    x = player_pos[1] * TILE_SIZE + TILE_SIZE // 2
    y = player_pos[0] * TILE_SIZE + TILE_SIZE // 2
    if mouth_open:
        pygame.draw.polygon(screen, YELLOW, [
            (x, y),
            (x + TILE_SIZE // 2 - 2, y - TILE_SIZE // 3),
            (x + TILE_SIZE // 2 - 2, y + TILE_SIZE // 3),
        ])
    pygame.draw.circle(screen, YELLOW, (x, y), TILE_SIZE // 2 - 2)

# Draw all ghosts in their current positions
def draw_ghosts():
    for ghost in ghosts:
        x = ghost["pos"][1] * TILE_SIZE + TILE_SIZE // 2
        y = ghost["pos"][0] * TILE_SIZE + TILE_SIZE // 2
        pygame.draw.circle(screen, ghost["color"], (x, y), TILE_SIZE // 2 - 2)

# Render and display the current score
def draw_score():
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))

# Move Pac-Man and update score if a dot is eaten
def move_player(dx, dy):
    global score
    new_row = player_pos[0] + dy
    new_col = player_pos[1] + dx
    if 0 <= new_row < ROWS and 0 <= new_col < COLS:
        if maze[new_row][new_col] != 1:
            player_pos[0] = new_row
            player_pos[1] = new_col
            if maze[new_row][new_col] == 0:
                maze[new_row][new_col] = 2
                score += 10

# Move ghosts in their current direction or choose a new valid one
def move_ghosts():
    for ghost in ghosts:
        r, c = ghost["pos"]
        dy, dx = ghost["dir"]
        new_r = r + dy
        new_c = c + dx
        if 0 <= new_r < ROWS and 0 <= new_c < COLS and maze[new_r][new_c] != 1:
            ghost["pos"] = [new_r, new_c]
        else:
            valid_dirs = []
            for direction in [(0,1), (1,0), (0,-1), (-1,0)]:
                test_r = r + direction[0]
                test_c = c + direction[1]
                if 0 <= test_r < ROWS and 0 <= test_c < COLS and maze[test_r][test_c] != 1:
                    valid_dirs.append(direction)
            if valid_dirs:
                ghost["dir"] = random.choice(valid_dirs)

# Check if any ghost has collided with Pac-Man
def check_collision():
    for ghost in ghosts:
        if ghost["pos"] == player_pos:
            return True
    return False

# Display game over screen and wait for player input
def game_over():
    screen.fill(BLACK)
    over_text = font.render("Game Over! Press Q to Quit or R to Restart", True, WHITE)
    score_text = font.render(f"Final Score: {score}", True, WHITE)
    screen.blit(over_text, (WIDTH // 2 - 170, HEIGHT // 2 - 20))
    screen.blit(score_text, (WIDTH // 2 - 60, HEIGHT // 2 + 20))
    pygame.display.flip()
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
        keys = pygame.key.get_pressed()
        if keys[pygame.K_q]:
            pygame.quit()
            sys.exit()
        elif keys[pygame.K_r]:
            # Start the game
main()

# Main game loop: updates the game state, draws objects, handles input
def main():
    global mouth_open, player_pos, ghosts, score
    player_pos = [1, 1]
    ghosts[0]["pos"] = [13, 10]
    ghosts[1]["pos"] = [7, 9]
    ghosts[0]["dir"] = random.choice([(0,1), (1,0), (0,-1), (-1,0)])
    ghosts[1]["dir"] = random.choice([(0,1), (1,0), (0,-1), (-1,0)])
    score = 0
    tick = 0
    while True:
        screen.fill(BLACK)
        draw_maze()
        draw_player()
        draw_ghosts()
        draw_score()
        pygame.display.flip()
        clock.tick(10)
        tick += 1
        if tick % 5 == 0:
            mouth_open = not mouth_open
        if tick % 2 == 0:
            move_ghosts()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            move_player(-player_speed, 0)
        if keys[pygame.K_RIGHT]:
            move_player(player_speed, 0)
        if keys[pygame.K_UP]:
            move_player(0, -player_speed)
        if keys[pygame.K_DOWN]:
            move_player(0, player_speed)
        if check_collision():
            game_over()

main()
```

---

## 🎯 Try These Challenges

* Try surviving as long as you can
* Try to eat all the dots (add a win condition!)
* Add more ghosts
* Change Pac-Man's starting position
* Speed up the ghost movement or add animations

---

🐍 This is part of the **Pythonly beginner series.**
Learn Python one line at a time. Follow [@Pythonly](https://www.youtube.com/@Pythonly) for more.
