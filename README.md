import pygame
import math
import random

pygame.init()
screen = pygame.display.set_mode((800, 600))
clock = pygame.time.Clock()

def distance(a, b):
    return math.hypot(a[0] - b[0], a[1] - b[1])

player_pos = [100, 300]
enemy_positions = [[600, 200], [700, 500], [400, 400]]
bullets = []

# Paredes (x, y, largura, altura)
walls = [
    pygame.Rect(300, 100, 30, 400),
    pygame.Rect(500, 0, 30, 300)
]

def get_closest_enemy(player_pos, enemies):
    closest = None
    min_dist = float('inf')
    for e in enemies:
        d = distance(player_pos, e)
        if d < min_dist:
            min_dist = d
            closest = e
    return closest

def line_intersects_wall(start, end, walls):
    """Verifica se a linha entre start e end colide com alguma parede."""
    for wall in walls:
        if wall.clipline(start, end):
            return True
    return False

running = True
while running:
    screen.fill((20, 20, 20))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        # Atira ao clicar
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            target = get_closest_enemy(player_pos, enemy_positions)
            if target:
                dx, dy = target[0] - player_pos[0], target[1] - player_pos[1]
                angle = math.atan2(dy, dx)
                bullet_dx = math.cos(angle) * 15
                bullet_dy = math.sin(angle) * 15
                bullets.append([player_pos[0], player_pos[1], bullet_dx, bullet_dy])

    # Desenha paredes
    for wall in walls:
        pygame.draw.rect(screen, (90, 90, 90), wall)

    # ESP: Desenha inimigos com transparência se estiverem atrás da parede
    for enemy in enemy_positions:
        enemy_rect = pygame.Rect(enemy[0] - 20, enemy[1] - 20, 40, 40)
        
        # Verifica se tem parede entre jogador e inimigo
        blocked = line_intersects_wall(player_pos, enemy, walls)
        
        s = pygame.Surface((40, 40), pygame.SRCALPHA)
        if blocked:
            s.fill((255, 0, 0, 80))  # Vermelho transparente (atrás da parede)
            # Contorno para destacar
            pygame.draw.rect(s, (255, 255, 0, 120), (0, 0, 40, 40), 2)
        else:
            s.fill((255, 0, 0, 200))  # Vermelho opaco (visível)
            pygame.draw.rect(s, (255, 255, 0, 220), (0, 0, 40, 40), 2)
        screen.blit(s, enemy_rect.topleft)

    # Desenha balas e verifica colisão
    for bullet in bullets[:]:
        bullet[0] += bullet[2]
        bullet[1] += bullet[3]
        pygame.draw.circle(screen, (255, 255, 0), (int(bullet[0]), int(bullet[1])), 5)
        # Remove bala se sair da tela
        if not (0 <= bullet[0] <= 800 and 0 <= bullet[1] <= 600):
            bullets.remove(bullet)
        else:
            # Colisão com inimigos
            for enemy in enemy_positions[:]:
                if distance((bullet[0], bullet[1]), enemy) < 20:
                    enemy_positions.remove(enemy)
                    if bullet in bullets:
                        bullets.remove(bullet)

    # Desenha jogador
    pygame.draw.circle(screen, (0, 255, 0), player_pos, 15)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
