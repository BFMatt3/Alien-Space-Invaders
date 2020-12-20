''' Alien-Space-Invaders

This is a knock-off of a YouTube video
"Pygame Tutorial for Beginners"
https://medium.com/javarevisited/my-favorite-free-courses-to-learn-game-development-for-beginners-f7615e26f675
with only slight modifications.
I did not add sound effects, my alien spacecraft are different, and there are a few other minor changes. 
In the same directory you need to import png images for ufo1.png, spaceship1.png, and bullet.png.
'''

import pygame
import random
import math
from pygame import mixer
# for sound effects

# Initialize the game
pygame.init ()

screen = pygame.display.set_mode ((800, 600))

# Background
pygame.display.set_caption ("Space Invaders")

'''
# Background Sound if you want it. You need to download a sound file.
mixer.music.load('background.wave')
mixer.music.play(-1)  #play in a loop, so put inside ()
'''

icon = pygame.image.load ('ufo1.png')
pygame.display.set_icon (icon)

# make a player
playerImg = pygame.image.load ('spaceship1.png')

playerX = 370
playerY = 480
playerX_change = 0

# Multiple enemies
enemyImg = []
enemyX = []
enemyY = []
enemyX_change = []
enemyY_change = []
num_of_enemies = 6

for i in range (num_of_enemies):
    enemyImg.append (pygame.image.load ('ufo1.png'))
    enemyX.append (random.randint (0, 735))
    enemyY.append (random.randint (50, 150))
    enemyX_change.append (0.3)
    enemyY_change.append (40)

# Bullet
bulletImg = pygame.image.load ('bullet.png')
bulletX = 0
bulletY = 480
bulletX_change = 0  # Bullet doesn't go in x direction
bulletY_change = 2
bullet_state = "ready"

#score
score_value = 0 # Go to Collision
font = pygame.font.Font('freesansbold.ttf', 32)
textX = 10  #padding from edge
textY = 10

#GAME OVER text
over_font = pygame.font.Font ('freesansbold.ttf', 64)

def show_score(x, y):
    score = font.render("Score: " + str(score_value), True, (255,255,255)) # Type casting
    screen.blit(score, (x, y))  

def game_over_text():
    over_text = over_font.render (("GAME OVER"), True, (255, 255, 255))  # Type casting
    screen.blit (over_text, (200, 250))


def player(x, y):
    screen.blit (playerImg, (x, y))  


def enemy(x, y, i): 
    screen.blit (enemyImg[i], (x, y))


def fire_bullet(x, y):
    global bullet_state  # So we can access bullet_state
    bullet_state = "fire"  # calls "if bullet_state is "fire":" at bottom
    screen.blit (bulletImg, (x + 16, y + 10))  # Appears using x and y
# Collision; Then go to While Loop below if bullet_state is "fire"
def isCollision(enemyX, enemyY, bulletX, bulletY):
    distance = math.sqrt ((math.pow (enemyX - bulletX, 2)) + (math.pow (enemyY - bulletY, 2)))
    if distance < 27:  # determined by trial and error
        return True  # could return False here and not write next two lines
    else:
        return False


# Game loop
running = True
while running:
    # screen.blit(background, (o, 0)) if provided
    screen.fill ((0, 0, 0))
    
    for event in pygame.event.get ():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                playerX_change = -0.4
            if event.key == pygame.K_RIGHT:
                playerX_change = 0.4
            if event.key == pygame.K_SPACE:  
                if bullet_state is "ready":
                    #bullet_sound = mixer.Sound('laser.wav')
                    #bullet_sound.play()
                    # get current x coordinate
                    bulletX = playerX  
                    fire_bullet (bulletX, bulletY)  # function

        if event.type == pygame.KEYUP:
            if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                playerX_change = 0

    playerX += playerX_change
    if playerX <= 0:
        playerX = 0
    elif playerX >= 736:  # 32 pixel space ship or 736 if 64 pixel ship
        playerX = 736
    # enemy movement
    
    for i in range (num_of_enemies):

        # GAME OVER
        if enemyY[i] > 440:
            for j in range (num_of_enemies):  # Collect all enemies to put off screen
                enemyY[j] = 2000 # Puts enemy off screen
            game_over_text()   # calls the game_over_text function
            break


        enemyX[i] += enemyX_change[i]  # i specifies which one you are referring to in list
        if enemyX[i] <= 0:
            enemyX_change[i] = 0.3
            enemyY[i] += enemyY_change[i]
        elif enemyX[i] >= 736:
            enemyX_change[i] = -0.3
            enemyY[i] += enemyY_change[i]

        # Collision
        collision = isCollision (enemyX[i], enemyY[i], bulletX, bulletY)
        if collision:
            #explosion_sound = mixer.Sound('explosion.wav'), if you added sound
            #explosion_sound.play()
            bulletY = 480
            bullet_state = "ready"
            score_value += 1

            enemyX[i] = random.randint (0, 735)
            enemyY[i] = random.randint (50, 150)

            # which enemy is drawn
        enemy (enemyX[i], enemyY[i], i)

    # Bullet Movement

    if bulletY < 0:  # On screen at top of screen
        bulletY = 480
        bullet_state = "ready"  # calls function; once bullet is at top of screen, resets

    if bullet_state is "fire":
        fire_bullet (bulletX, bulletY)  # calls function (within while loop)
        bulletY -= bulletY_change

    player (playerX, playerY)  # Update movement using playerX and Y
    show_score(textX, textY)
    pygame.display.update ()
pygame.quit()
