# Soaring-Tunnel-Vision
#Flapping bird
import pygame
from pygame.locals import *
import random

pygame.init()

# Clock and FPS
clock = pygame.time.Clock()
fps = 60

# Screen dimensions
screen_width = 800
screen_height = 900

# Create the screen
screen = pygame.display.set_mode((screen_width, screen_height), pygame.DOUBLEBUF)
pygame.display.set_caption('Flappy Bird')

# Define font
font = pygame.font.SysFont('Time New Roman', 150)

# Define colors
white = (255, 255, 255)

# Game variables
ground_scroll = 0
scroll_speed = 3
flying = False
game_over = False
pipe_gap = 250  # Increased pipe gap
pipe_frequency = 1500  # milliseconds
last_pipe = pygame.time.get_ticks() - pipe_frequency
score = 0
pass_pipe = False

# Power-up variables
invincible = False
gap_boost = False
double_score = False
powerup_timer = pygame.time.get_ticks()

# Load images
bg = pygame.image.load('bgtwo.JPEG')
ground_img = pygame.image.load('ground.png')
button_img = pygame.image.load('restart.png')

# Power-up images
powerup_images = {
    'invincibility': pygame.image.load('invincibility.PNG'),
    'gap_boost': pygame.image.load('gap_boost.PNG'),
    'double_score': pygame.image.load('double_score.PNG')
}

# Draw text on the screen
def draw_text(text, font, text_col, x, y):
    img = font.render(text, True, text_col)
    screen.blit(img, (x, y))

# Bird class
class Bird(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.images = [pygame.image.load(f'bird{n}.PNG') for n in range(1, 4)]
        self.index = 0
        self.counter = 0
        self.image = self.images[self.index]
        self.rect = self.image.get_rect()
        self.rect.center = [x, y]
        self.vel = 0
        self.clicked = False

    def update(self):
        global flying, game_over

        if flying:
            # Gravity
            self.vel += 0.5
            if self.vel > 8:
                self.vel = 8
            if self.rect.bottom < 768:
                self.rect.y += int(self.vel)

        if not game_over:
            # Jump
            if pygame.mouse.get_pressed()[0] == 1 and not self.clicked:
                self.clicked = True
                self.vel = -10
            if pygame.mouse.get_pressed()[0] == 0:
                self.clicked = False

            # Animation
            self.counter += 1
            flap_cooldown = 5
            if self.counter > flap_cooldown:
                self.counter = 0
                self.index = (self.index + 1) % len(self.images)

            # Rotate bird
            self.image = pygame.transform.rotate(self.images[self.index], self.vel * -2)
        else:
            self.image = pygame.transform.rotate(self.images[self.index], -90)

# Pipe class
class Pipe(pygame.sprite.Sprite):
    def __init__(self, x, y, position):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.image.load('pipe.png')
        self.rect = self.image.get_rect()
        if position == 1:  # Top pipe
            self.image = pygame.transform.flip(self.image, False, True)
            self.rect.bottomleft = [x, y - int(pipe_gap / 2)]
        elif position == -1:  # Bottom pipe
            self.rect.topleft = [x, y + int(pipe_gap / 2)]

    def update(self):
        self.rect.x -= scroll_speed
        if self.rect.right < 0:
            self.kill()

# Power-Up class
class PowerUp(pygame.sprite.Sprite):
    def __init__(self, x, y, power_type):
        pygame.sprite.Sprite.__init__(self)
        self.image = powerup_images[power_type]
        self.rect = self.image.get_rect()
        self.rect.center = [x, y]
        self.power_type = power_type

    def update(self):
        self.rect.x -= scroll_speed
        if self.rect.right < 0:
            self.kill()

# Button class
class Button():
    def __init__(self, x, y, image):
        self.image = image
        self.rect = self.image.get_rect()
        self.rect.topleft = (x, y)

    def draw(self):
        action = False
        pos = pygame.mouse.get_pos()
        if self.rect.collidepoint(pos):
            if pygame.mouse.get_pressed()[0] == 1:
                action = True
        screen.blit(self.image, (self.rect.x, self.rect.y))
        return action

# Reset the game
def reset_game():
    global score, invincible, gap_boost, double_score, powerup_timer, pipe_gap
    pipe_group.empty()
    powerup_group.empty()
    bird.rect.x = 100
    bird.rect.y = int(screen_height / 2)
    score = 0
    invincible = False
    gap_boost = False
    double_score = False
    pipe_gap = 250  # Reset to default pipe gap
    powerup_timer = pygame.time.get_ticks()
    return score

# Activate power-ups
def activate_powerup(power_type):
    global invincible, gap_boost, double_score, pipe_gap, powerup_timer
    if power_type == 'invincibility':
        invincible = True
    elif power_type == 'gap_boost':
        gap_boost = True
        pipe_gap = 500  # Increase pipe gap for the 'gap_boost' power-up
    elif power_type == 'double_score':
        double_score = True
    powerup_timer = pygame.time.get_ticks()

# Deactivate power-ups after timer
def deactivate_powerups():
    global invincible, gap_boost, double_score, pipe_gap
    invincible = False
    gap_boost = False
    double_score = False
    pipe_gap = 250

# Update the score
def update_score():
    global score, pass_pipe
    pass_pipe = False  # Reset pass_pipe for this frame
    for pipe in pipe_group:
        # Check if bird passes the center of a pipe
        if pipe.rect.centerx < bird.rect.left and not pipe.rect.right < bird.rect.left:
            pass_pipe = True  # Bird passed a pipe
    if pass_pipe:
        score += 2 if double_score else 1  # Double score if power-up is active

# Groups
bird_group = pygame.sprite.Group()
pipe_group = pygame.sprite.Group()
powerup_group = pygame.sprite.Group()

bird = Bird(100, int(screen_height / 2))
bird_group.add(bird)

# Restart button
button = Button(screen_width // 2 - 50, screen_height // 2 - 100, button_img)

# Game loop
run = True
while run:
    clock.tick(fps)
    screen.blit(bg, (0, 0))

    bird_group.draw(screen)
    bird_group.update()
    pipe_group.draw(screen)
    pipe_group.update()
    powerup_group.draw(screen)
    powerup_group.update()

    # Draw ground
    screen.blit(ground_img, (ground_scroll, 768))

    # Check for power-up collection
    if pygame.sprite.spritecollide(bird, powerup_group, True):
        collected_powerup = random.choice(['invincibility', 'gap_boost', 'double_score'])
        activate_powerup(collected_powerup)

    # Deactivate power-ups after 10 seconds
    if pygame.time.get_ticks() - powerup_timer > 10000:  # 10 seconds
        deactivate_powerups()

    # Check collisions if not invincible
    if not invincible and (pygame.sprite.groupcollide(bird_group, pipe_group, False, False) or bird.rect.top < 0):
        game_over = True

    # Check if bird hit the ground
    if bird.rect.bottom >= 768:
        game_over = True
        flying = False

    if not game_over and flying:
        # Generate pipes
        time_now = pygame.time.get_ticks()
        if time_now - last_pipe > pipe_frequency:
            pipe_height = random.randint(-100, 100)
            btm_pipe = Pipe(screen_width, int(screen_height / 2) + pipe_height, -1)
            top_pipe = Pipe(screen_width, int(screen_height / 2) + pipe_height, 1)
            pipe_group.add(btm_pipe)
            pipe_group.add(top_pipe)
            last_pipe = time_now

        # Generate power-ups occasionally
        if random.randint(1, 300) == 1:  # 1 in 300 chance per frame
            powerup_type = random.choice(['invincibility', 'gap_boost', 'double_score'])
            powerup = PowerUp(screen_width, random.randint(200, 700), powerup_type)
            powerup_group.add(powerup)

        # Scroll ground
        ground_scroll -= scroll_speed
        if abs(ground_scroll) > 35:
            ground_scroll = 0

    # Update score
    update_score()

    # Draw score
    draw_text(str(score), font, white, int(screen_width / 2), 20)

    # Check for game over
    if game_over:
        if button.draw():
            game_over = False
            score = reset_game()

    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            run = False
        if event.type == pygame.MOUSEBUTTONDOWN and not flying and not game_over:
            flying = True

    pygame.display.update()

pygame.quit()
