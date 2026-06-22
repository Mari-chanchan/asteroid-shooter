# asteroid-shooter
Asteroids shooting game.
import pygame
import random

pygame.init()
pygame.mixer.init()

pygame.mixer.music.load("BGM.mp3")
pygame.mixer.music.set_volume(0.5)
pygame.mixer.music.play(-1)

laser_sound = pygame.mixer.Sound("laser_sound.mp3")
explosion_sound = pygame.mixer.Sound("explosion_sound.mp3")

background = pygame.image.load("wp3213868.jpg")
background = pygame.transform.scale(background, (800, 600))

ship_image = pygame.image.load("ship.png.webp")
ship_image = pygame.transform.scale(ship_image, (100, 100))

asteroid_image = pygame.image.load("asteroid_png.png")

boss_image = pygame.image.load("boss.png")
boss_image = pygame.transform.scale(
    boss_image,
    (150, 150)
)


width = 800
height = 600

screen = pygame.display.set_mode(
    (width, height)
)

pygame.display.set_caption(
    "Asteroid Shooter"
)

clock = pygame.time.Clock()

ship_x = width // 2
ship_y = height - 60

ship_speed = 6

bullets = []
asteroids = []

score = 0

boss_active = False
boss_hp = 30

boss_x = 400
boss_y = 100

boss_defeated = False

try:
    with open(
        "highscore.txt",
        "r"
    ) as file:

        highscore = int(
            file.read()
        )

except:
    highscore = 0

font = pygame.font.SysFont(
    None,
    40
)

running = True
game_over = False

shoot_timer = 0

time_limit = 60

start_time = pygame.time.get_ticks()

while running:

    clock.tick(60)

    

    for event in pygame.event.get():

        if event.type == pygame.QUIT:
            running = False

        if game_over:

            if event.type == pygame.KEYDOWN:

                if event.key == pygame.K_r:

                    ship_x = width // 2
                    ship_y = height - 60

                    bullets.clear()
                    asteroids.clear()

                    score = 0

                    boss_active = False
                    boss_hp = 30

                    boss_x = 400
                    boss_y = 100

                    game_over = False

                    shoot_timer = 0

                    start_time = pygame.time.get_ticks()

    keys = pygame.key.get_pressed()

    if not game_over:

        if keys[pygame.K_LEFT]:
            ship_x -= ship_speed

        if keys[pygame.K_RIGHT]:
            ship_x += ship_speed

    ship_x = max(
        50,
        min(width - 50, ship_x)
    )

    shoot_timer += 1

    if not game_over:

        if (
            keys[pygame.K_SPACE]
            and shoot_timer >= 10
        ):

            bullets.append(
                [ship_x, ship_y - 40]
            )

            laser_sound.play()

            shoot_timer = 0

    screen.blit(
        background,
        (0, 0)
    )

    for bullet in bullets[:]:

        bullet[1] -= 10

        if bullet[1] < 0:
            bullets.remove(bullet)

    if not game_over:

        if random.randint(1, 40) == 1:

            asteroid_x = random.randint(
                30,
                width - 30
            )

            asteroid_y = -30

            asteroid_size = random.randint(
                20,
                40
            )

            asteroids.append(
                [
                    asteroid_x,
                    asteroid_y,
                    asteroid_size
                ]
            )

    if score >= 20 and not boss_active and not boss_defeated:
        boss_active = True

        screen.blit(
            boss_image,
            (
                boss_x - 35,
                boss_y - 35
            )
        )
    
    for asteroid in asteroids[:]:

        asteroid[1] += 4

        if asteroid[1] > height:
            asteroids.remove(asteroid)

    ship_radius = 25

    for asteroid in asteroids:

        dx = ship_x - asteroid[0]
        dy = ship_y - asteroid[1]

        distance = (
            dx ** 2 + dy ** 2
        ) ** 0.5

        if distance < asteroid[2] + ship_radius:
            game_over = True

    for bullet in bullets[:]:

        for asteroid in asteroids[:]:

            dx = bullet[0] - asteroid[0]
            dy = bullet[1] - asteroid[1]

            distance = (
                dx ** 2 + dy ** 2
            ) ** 0.5

            if distance < asteroid[2] + 10:

                if bullet in bullets:
                    bullets.remove(bullet)

                if asteroid in asteroids:
                    asteroids.remove(asteroid)

                explosion_sound.play()

                score += 1

                break

    if boss_active:

        boss_x += 3

        if boss_x > 700:
            boss_x = 100

        for bullet in bullets[:]:

            dx = bullet[0] - boss_x
            dy = bullet[1] - boss_y

            distance = (
                dx ** 2 + dy ** 2
            ) ** 0.5

            if distance < 75:

                if bullet in bullets:
                    bullets.remove(bullet)

                boss_hp -= 1

                explosion_sound.play()

        if boss_hp <= 0:

            boss_active = False

            score += 30
            boss_hp = -999

    if boss_hp <= 0 and not boss_defeated:

        boss_active = False

        score += 30

        boss_defeated = True

    screen.blit(
        ship_image,
        (
            ship_x - 50,
            ship_y - 50
        )
    )

    for bullet in bullets:

        pygame.draw.circle(
            screen,
            (255, 255, 0),
            (
                int(bullet[0]),
                int(bullet[1])
            ),
            4
        )

    for asteroid in asteroids:

        size = asteroid[2] * 2

        asteroid_scaled = pygame.transform.scale(
            asteroid_image,
            (size, size)
        )

        screen.blit(
            asteroid_scaled,
            (
                asteroid[0] - size // 2,
                asteroid[1] - size // 2
            )
        )

    if boss_active:

        screen.blit(
            boss_image,
            (
                boss_x - 75,
                boss_y - 75
            )
        )

        hp_text = font.render(
            f"Boss HP: {boss_hp}",
            True,
            (255, 0, 0)
        )

        screen.blit(
            hp_text,
            (520, 10)
        )

    if score > highscore:

        highscore = score

        with open(
            "highscore.txt",
            "w"
        ) as file:

            file.write(
                str(highscore)
            )

    score_text = font.render(
        f"Score: {score}",
        True,
        (255, 255, 255)
    )

    screen.blit(
        score_text,
        (10, 10)
    )

    highscore_text = font.render(
        f"High Score: {highscore}",
        True,
        (255, 255, 255)
    )

    screen.blit(
        highscore_text,
        (10, 90)
    )



    if game_over:

            text = font.render(
                "GAME OVER",
                True,
                (255, 0, 0)
            )

            screen.blit(
            text,
            (230, 220)
        )

            final_score = font.render(
            f"Final Score: {score}",
            True,
            (255, 255, 0)
        )

            screen.blit(
            final_score,
            (220, 170)
        )

            restart_text = font.render(
            "Press R to Restart",
            True,
            (255, 255, 255)
        )

            screen.blit(
            restart_text,
            (180, 300)
        )

    pygame.display.flip()

pygame.quit()
