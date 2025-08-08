import pygame
import sys
import math
import random
from datetime import datetime

# Initialize pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 1000, 700
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Birthday Surprise for Akzad & Umaira!")

# Colors
BACKGROUND = (10, 5, 30)
PINK = (255, 105, 180)
GOLD = (255, 215, 0)
PURPLE = (180, 70, 230)
CYAN = (0, 255, 255)
WHITE = (255, 255, 255)
RED = (255, 50, 50)
BLUE = (50, 150, 255)
GREEN = (50, 255, 100)

# Birthday information
BIRTHDAY_MONTH = 10
BIRTHDAY_DAY = 16

# Fonts
title_font = pygame.font.SysFont("arial", 50, bold=True)
name_font = pygame.font.SysFont("arial", 36, bold=True)
text_font = pygame.font.SysFont("arial", 28)
countdown_font = pygame.font.SysFont("arial", 32, bold=True)
message_font = pygame.font.SysFont("arial", 24)

class Particle:
    """Particle for fireworks effect"""
    def __init__(self, x, y, color):
        self.x = x
        self.y = y
        self.color = color
        self.size = random.randint(2, 5)
        self.speed_x = random.uniform(-2, 2)
        self.speed_y = random.uniform(-3, -1)
        self.life = random.randint(20, 40)
    
    def update(self):
        self.x += self.speed_x
        self.y += self.speed_y
        self.speed_y += 0.1
        self.life -= 1
        
    def draw(self, surface):
        pygame.draw.circle(surface, self.color, (int(self.x), int(self.y)), self.size)
        
    def is_dead(self):
        return self.life <= 0

class Balloon:
    """Floating balloon with swinging motion"""
    def __init__(self):
        self.reset()
        self.color = random.choice([RED, PINK, GOLD, PURPLE, BLUE, CYAN, GREEN])
        
    def reset(self):
        self.x = random.randint(50, WIDTH-50)
        self.y = HEIGHT + 50
        self.speed = random.uniform(1.5, 3.5)
        self.radius = random.randint(20, 40)
        self.swing = random.uniform(0.02, 0.08)
        self.angle = random.uniform(0, 2 * math.pi)
        
    def update(self):
        self.y -= self.speed
        self.angle += self.swing
        self.x += math.sin(self.angle) * 0.8
        
        if self.y < -100:
            self.reset()
            
    def draw(self, surface):
        # Balloon body
        pygame.draw.circle(surface, self.color, (int(self.x), int(self.y)), self.radius)
        pygame.draw.circle(surface, WHITE, (int(self.x), int(self.y)), self.radius, 2)
        
        # Balloon tip
        pygame.draw.polygon(surface, self.color, [
            (self.x, self.y + self.radius),
            (self.x - 5, self.y + self.radius + 10),
            (self.x + 5, self.y + self.radius + 10)
        ])
        
        # String
        pygame.draw.line(surface, WHITE, (self.x, self.y + self.radius + 10), 
                         (self.x + math.sin(self.angle)*15, self.y + self.radius + 50), 2)

def draw_cake(surface, x, y):
    """Draw birthday cake with layers, icing, decorations, and candles"""
    # Cake layers
    pygame.draw.rect(surface, (220, 150, 130), (x-100, y-50, 200, 40))  # Top layer
    pygame.draw.rect(surface, (200, 120, 100), (x-120, y-10, 240, 50))   # Middle layer
    pygame.draw.rect(surface, (180, 90, 70), (x-140, y+40, 280, 60))     # Bottom layer
    
    # Icing
    pygame.draw.rect(surface, PINK, (x-100, y-55, 200, 10))
    pygame.draw.rect(surface, PURPLE, (x-120, y-15, 240, 10))
    pygame.draw.rect(surface, (255, 105, 180), (x-140, y+35, 280, 10))
    
    # Decorations
    for i in range(8):
        pygame.draw.circle(surface, GOLD, (x - 80 + i*25, y-60), 8)
        pygame.draw.circle(surface, CYAN, (x - 90 + i*25, y-15), 10)
        pygame.draw.circle(surface, BLUE, (x - 100 + i*30, y+35), 12)
    
    # Candles
    candle_colors = [RED, PINK, GOLD, PURPLE, BLUE, CYAN]
    for i in range(6):
        cx = x - 75 + i*30
        pygame.draw.rect(surface, candle_colors[i], (cx-3, y-110, 6, 60))
        
        # Flame
        flame_color = (255, random.randint(150, 255), 50)
        pygame.draw.circle(surface, flame_color, (cx, y-115), 8)
        pygame.draw.circle(surface, WHITE, (cx, y-115), 4)

def draw_birthday_text(surface):
    """Draw the birthday greeting texts"""
    title = title_font.render("Happy Birthday Akzad & Umaira!", True, GOLD)
    surface.blit(title, (WIDTH//2 - title.get_width()//2, 30))
    
    subtitle = text_font.render("October 16 - Birthday Twins", True, CYAN)
    surface.blit(subtitle, (WIDTH//2 - subtitle.get_width()//2, 90))
    
    akzad_text = name_font.render("Akzad Imaad", True, BLUE)
    umaira_text = name_font.render("Umaira", True, PINK)
    heart = name_font.render("❤️", True, RED)
    
    surface.blit(akzad_text, (WIDTH//2 - akzad_text.get_width() - 30, 150))
    surface.blit(heart, (WIDTH//2 - heart.get_width()//2, 150))
    surface.blit(umaira_text, (WIDTH//2 + 30, 150))

def calculate_countdown():
    """Calculate time remaining to the birthday"""
    today = datetime.now()
    current_year = today.year
    
    if (today.month, today.day) > (BIRTHDAY_MONTH, BIRTHDAY_DAY):
        current_year += 1
    
    birthday = datetime(current_year, BIRTHDAY_MONTH, BIRTHDAY_DAY)
    delta = birthday - today
    
    days = delta.days
    hours, remainder = divmod(delta.seconds, 3600)
    minutes, seconds = divmod(remainder, 60)
    
    return days, hours, minutes, seconds

def draw_countdown(surface):
    """Draw the countdown timer"""
    days, hours, minutes, seconds = calculate_countdown()
    
    countdown_text = countdown_font.render(
        f"Birthday Countdown: {days} days, {hours} hours, {minutes} minutes", True, WHITE
    )
    surface.blit(countdown_text, (WIDTH//2 - countdown_text.get_width()//2, HEIGHT - 150))
    
    if days == 0:
        celebration = countdown_font.render("TODAY IS OUR BIRTHDAY! 🎉🎂🎈", True, GOLD)
        surface.blit(celebration, (WIDTH//2 - celebration.get_width()//2, HEIGHT - 100))

def draw_messages(surface):
    """Draw birthday messages on screen"""
    messages = [
        "Sharing a birthday with you makes it extra special!",
        "From late-night talks to endless laughter,",
        "you're not just my bestie, you're family.",
        "Here's to more shared birthdays,",
        "more memories, and more adventures!",
        "Wishing you the most amazing day,",
        "just as amazing as you are!",
        "With love, your birthday twin Akzad"
    ]
    
    for i, message in enumerate(messages):
        color = random.choice([PINK, GOLD, CYAN, BLUE, PURPLE, GREEN])
        text = message_font.render(message, True, color)
        surface.blit(text, (WIDTH//2 - text.get_width()//2, 220 + i*30))

def main():
    clock = pygame.time.Clock()
    particles = []
    balloons = [Balloon() for _ in range(15)]
    
    running = True
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    running = False
                elif event.key == pygame.K_SPACE:
                    # Create fireworks particles when space pressed
                    for _ in range(50):
                        particles.append(Particle(
                            random.randint(0, WIDTH),
                            HEIGHT - 100,
                            random.choice([PINK, GOLD, PURPLE, CYAN, GREEN])
                        ))
        
        # Update balloons
        for balloon in balloons:
            balloon.update()
        
        # Update particles and remove dead ones
        for particle in particles[:]:
            particle.update()
            if particle.is_dead():
                particles.remove(particle)
        
        # Add occasional particles for sparkle effect
        if random.random() < 0.2:
            particles.append(Particle(
                random.randint(0, WIDTH),
                HEIGHT - 100,
                random.choice([PINK, GOLD, PURPLE, CYAN, GREEN])
            ))
        
        # Draw everything
        screen.fill(BACKGROUND)
        
        # Draw stars in background
        for i in range(100):
            x = (i * 17) % WIDTH
            y = (i * 23) % HEIGHT
            size = (i % 3) + 1
            brightness = 150 + (i % 105)
            pygame.draw.circle(screen, (brightness, brightness, brightness), (x, y), size)
        
        # Draw balloons
        for balloon in balloons:
            balloon.draw(screen)
        
        # Draw cake
        draw_cake(screen, WIDTH//2, HEIGHT - 200)
        
        # Draw texts and countdown
        draw_birthday_text(screen)
        draw_countdown(screen)
        draw_messages(screen)
        
        # Draw particles
        for particle in particles:
            particle.draw(screen)
        
        pygame.display.flip()
        clock.tick(60)
    
    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
    
