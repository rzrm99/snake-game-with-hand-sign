import pygame
import time
import random
import cv2
import mediapipe as mp

# Initialize pygame
pygame.init()

# Screen dimensions
WIDTH = 800
HEIGHT = 600
CELL_SIZE = 20

# Colors
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
BLACK = (0, 0, 0)

# Snake speed
SPEED = 15

# Setup the screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Snake Game")

# Clock for controlling game speed
clock = pygame.time.Clock()

# Fonts for score display
font_style = pygame.font.SysFont("bahnschrift", 25)
score_font = pygame.font.SysFont("comicsansms", 35)

# MediaPipe Hands Setup
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(max_num_hands=1, min_detection_confidence=0.7, min_tracking_confidence=0.7)
mp_draw = mp.solutions.drawing_utils

# OpenCV Camera Setup
cap = cv2.VideoCapture(0)


def display_score(score):
    value = score_font.render("Your Score: " + str(score), True, WHITE)
    screen.blit(value, [0, 0])


def draw_snake(snake_list):
    for x in snake_list:
        pygame.draw.rect(screen, GREEN, [x[0], x[1], CELL_SIZE, CELL_SIZE])


def message(msg, color, position):
    mesg = font_style.render(msg, True, color)
    screen.blit(mesg, position)


def get_camera_input():
    ret, frame = cap.read()
    if not ret:
        return None

    frame = cv2.flip(frame, 1)  # Flip the frame horizontally
    rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = hands.process(rgb_frame)

    direction = None

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            x_base = hand_landmarks.landmark[9].x
            y_base = hand_landmarks.landmark[9].y
            x_tip = hand_landmarks.landmark[8].x
            y_tip = hand_landmarks.landmark[8].y

            if abs(x_tip - x_base) > abs(y_tip - y_base):
                if x_tip < x_base:
                    direction = "LEFT"
                else:
                    direction = "RIGHT"
            else:
                if y_tip < y_base:
                    direction = "UP"
                else:
                    direction = "DOWN"

            mp_draw.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)

    cv2.imshow("Camera Input", frame)
    return direction


def game_loop():
    # Snake starting position and attributes
    game_over = False
    game_close = False

    x = WIDTH // 2
    y = HEIGHT // 2

    x_change = 0
    y_change = 0

    snake_list = []
    snake_length = 1

    # Food position
    food_x = round(random.randrange(0, WIDTH - CELL_SIZE) / CELL_SIZE) * CELL_SIZE
    food_y = round(random.randrange(0, HEIGHT - CELL_SIZE) / CELL_SIZE) * CELL_SIZE

    # Main game loop
    while not game_over:
        while game_close:
            screen.fill(BLACK)
            message("Game Over! Press Q-Quit or C-Play Again", RED, [WIDTH // 6, HEIGHT // 3])
            display_score(snake_length - 1)
            pygame.display.update()

            for event in pygame.event.get():
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_q:
                        game_over = True
                        game_close = False
                    if event.key == pygame.K_c:
                        game_loop()

        # Process camera input for movement
        direction = get_camera_input()
        if direction == "LEFT" and x_change == 0:
            x_change = -CELL_SIZE
            y_change = 0
        elif direction == "RIGHT" and x_change == 0:
            x_change = CELL_SIZE
            y_change = 0
        elif direction == "UP" and y_change == 0:
            y_change = -CELL_SIZE
            x_change = 0
        elif direction == "DOWN" and y_change == 0:
            y_change = CELL_SIZE
            x_change = 0

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_over = True

        # Boundary conditions
        if x >= WIDTH or x < 0 or y >= HEIGHT or y < 0:
            game_close = True

        x += x_change
        y += y_change
        screen.fill(BLACK)
        pygame.draw.rect(screen, RED, [food_x, food_y, CELL_SIZE, CELL_SIZE])

        # Update snake position
        snake_head = []
        snake_head.append(x)
        snake_head.append(y)
        snake_list.append(snake_head)
        if len(snake_list) > snake_length:
            del snake_list[0]

        # Check for collision with itself
        for segment in snake_list[:-1]:
            if segment == snake_head:
                game_close = True

        draw_snake(snake_list)
        display_score(snake_length - 1)

        pygame.display.update()

        # Check for food collision
        if x == food_x and y == food_y:
            food_x = round(random.randrange(0, WIDTH - CELL_SIZE) / CELL_SIZE) * CELL_SIZE
            food_y = round(random.randrange(0, HEIGHT - CELL_SIZE) / CELL_SIZE) * CELL_SIZE
            snake_length += 1

        clock.tick(SPEED)

    cap.release()
    cv2.destroyAllWindows()
    pygame.quit()
    quit()


# Start the game
game_loop()
