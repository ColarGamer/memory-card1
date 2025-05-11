from pygame import *
from random import randint

class GameSprite(sprite.Sprite):
    def __init__(self, player_image, player_x, player_y, player_speed):
        super().__init__()
        self.image = transform.scale(image.load(player_image), (65, 65))
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y

    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def __init__(self, player_image, player_x, player_y, player_speed):
        super().__init__(player_image, player_x, player_y, player_speed)
        self.lives = 3 

    def update(self):
        keys = key.get_pressed()
        if keys[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_RIGHT] and self.rect.x < win_width - 70:
            self.rect.x += self.speed

    def fire(self):
        if len(bullets) < 5: 
            bullet = Bullet("bullet.png", self.rect.centerx - 5, self.rect.top, 10)
            bullets.add(bullet)

class Bullet(GameSprite):
    def __init__(self, player_image, player_x, player_y, player_speed):
        super().__init__(player_image, player_x, player_y, player_speed)

    def update(self):
        self.rect.y -= self.speed
        if self.rect.y < 0:
            self.kill()

class Enemy(GameSprite):
    def update(self):
        self.rect.y += self.speed
        global lost
        if self.rect.y > win_height:
            self.rect.x = randint(0, win_width - 50)
            self.rect.y = 0
            lost += 1

class Asteroid(GameSprite):
    def update(self):
        self.rect.y += self.speed
        global player_lives
        if self.rect.y > win_height:
            self.rect.x = randint(0, win_width - 50)
            self.rect.y = 0
        if sprite.collide_rect(self, player):
            player.lives -= 1  
            self.kill()  

win_width = 700
win_height = 500
window = display.set_mode((win_width, win_height))
display.set_caption("Игра")
background = transform.scale(image.load("galaxy.jpg"), (win_width, win_height))

font.init()
font2 = font.SysFont('Arial', 36)
font_win = font.SysFont('Arial', 80)
win_text = font_win.render("Вы выиграли!", True, (0, 255, 0))
font_lose = font.SysFont('Arial', 80)
lose_text = font_lose.render("Вы проиграли!", True, (255, 0, 0))

score = 0
lost = 0
player_lives = 3 

player = Player('rocket.png', 50, 400, 6)

enemies = sprite.Group()
for i in range(5):
    enemy = Enemy('ufo.png', randint(0, win_width - 50), 0, randint(1, 3))
    enemies.add(enemy)

asteroids = sprite.Group() 
for i in range(3):
    asteroid = Asteroid('asteroid.png', randint(0, win_width - 50), 0, randint(1, 3))
    asteroids.add(asteroid)

bullets = sprite.Group()

mixer.init()
mixer.music.load('space.ogg')
mixer.music.set_volume(0.4)
mixer.music.play(-1) 

clock = time.Clock()
fps = 60
finish = False

while True:
    for e in event.get():
        if e.type == QUIT:
            exit()
        elif e.type == KEYDOWN:
            if e.key == K_SPACE: 
                player.fire()

    if not finish:
        window.blit(background, (0, 0))

        player.update()
        bullets.update()
        enemies.update()
        asteroids.update()

        for bullet in bullets:
            collides_with_enemies = sprite.spritecollide(bullet, enemies, True)
            for enemy in collides_with_enemies:
                score += 1 
                bullet.kill() 

        for asteroid in asteroids:
            if sprite.collide_rect(player, asteroid): 
                asteroid.kill() 
                if player.lives > 1: 
                    player.lives -= 1 
                else: 
                    finish = True 

        player.reset()
        bullets.draw(window)
        enemies.draw(window)
        asteroids.draw(window)
