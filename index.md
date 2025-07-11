# Project Name Here
Replace this text with a brief description (2-3 sentences) of your project. This description should draw the reader in and make them interested in what you've built. You can include what the biggest challenges, takeaways, and triumphs from completing the project were. As you complete your portfolio, remember your audience is less familiar than you are with all that your project entails!

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Gleb S | Harvard Westlake | Aerospace Engineering | Incoming Freshman

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BS



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/NKLvVwjHIGs?si=0_eYlWlbY35JMApi" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# Schematics 
<!-- Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. -->

# Code 

```python
import os
import sys
import pygame
import requests
import traceback

# --- Pygame Setup ---
pygame.init()
screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
SCREEN_WIDTH, SCREEN_HEIGHT = screen.get_size()
pygame.display.set_caption("Offline Text Adventure")

# --- Game Configuration ---
BUTTON_RADIUS = 80
MARGIN = 40
USE_OPENAI = True
OPENAI_API_KEY = "sk..." #I hardcoded it because it worked better you can use a safer way by making a settings.toml file
#and adding GPTKEY = "sk...."

# --- Prompt Setup ---
BasePrompt = (
    "You are an AI Game Master running a sword-and-sorcery fantasy adventure.\\n"
    "The player is on a quest involving danger, magic, and ancient ruins.\\n"
    "At each step: describe the scene in 2-3 sentences, list items, and offer 4 actions.\\n"
    "Sometimes, describe enemies. If combat begins, say 'roll initiative!' and describe the foe. Enemy has to make sense based on current setting."
    "The player can only die in combat. Keep it short and terse."
)
SessionMessages = [{"role": "system", "content": BasePrompt}]

# --- Fonts ---
font = pygame.font.SysFont("Arial", 24)
label_font = pygame.font.SysFont("Arial", 32, bold=True)

# --- Player Stats ---
PlayerStats = {
    "Class": None,
    "HP": 10,
    "MP": 5,
    "Inventory": [],
    "SpellBook": []
}

InCombat = False
CombatState = {
    "EnemyName": "",
    "EnemyHP": 0,
    "EnemyAttack": 0
}

CurrentOptions = ["1", "2", "3", "4"]

# --- UI Class ---
class WrappedTextDisplay:
    def __init__(self):
        self.line_offset = 0
        self.lines = []

    def WrapText(self, text, max_width):
        wrapped_lines = []
        for paragraph in text.splitlines():
            words = paragraph.split()
            current_line = ""
            for word in words:
                test_line = current_line + " " + word if current_line else word
                if font.size(test_line)[0] <= max_width:
                    current_line = test_line
                else:
                    wrapped_lines.append(current_line)
                    current_line = word
            if current_line:
                wrapped_lines.append(current_line)
        return wrapped_lines

    def AddText(self, text):
        self.lines.extend(self.WrapText(text, SCREEN_WIDTH - 40))
        self.Refresh()

    def SetText(self, text):
        self.lines = self.WrapText(text, SCREEN_WIDTH - 40)
        self.line_offset = 0
        self.Refresh()

    def Refresh(self):
        screen.fill((10, 10, 30))
        y = SCREEN_HEIGHT // 2 - len(self.lines) * 20
        for line in self.lines[self.line_offset:self.line_offset + 15]:
            text_surface = font.render(line, True, (255, 255, 255))
            x = SCREEN_WIDTH // 2 - text_surface.get_width() // 2
            screen.blit(text_surface, (x, y))
            y += 40
        DrawButtons(CurrentOptions)
        pygame.display.flip()

# --- Buttons ---
ButtonPositions = [
    (MARGIN + BUTTON_RADIUS, MARGIN + BUTTON_RADIUS),
    (SCREEN_WIDTH - MARGIN - BUTTON_RADIUS, MARGIN + BUTTON_RADIUS),
    (MARGIN + BUTTON_RADIUS, SCREEN_HEIGHT - MARGIN - BUTTON_RADIUS),
    (SCREEN_WIDTH - MARGIN - BUTTON_RADIUS, SCREEN_HEIGHT - MARGIN - BUTTON_RADIUS),
]
ButtonLabels = ["1", "2", "3", "4"]

def DrawButtons(labels):
    for i, (x, y) in enumerate(ButtonPositions):
        pygame.draw.circle(screen, (100, 0, 0), (x, y), BUTTON_RADIUS)
        pygame.draw.circle(screen, (255, 0, 0), (x, y), BUTTON_RADIUS, 4)
        label = label_font.render(str(labels[i]), True, (255, 255, 255))
        screen.blit(label, (x - label.get_width() // 2, y - label.get_height() // 2))

# --- GPT Functions ---
def MakePrompt(choice):
    return SessionMessages + [{"role": "user", "content": f"PLAYER: {choice}"}]

def RecordGameStep(choice, response):
    SessionMessages.extend([
        {"role": "user", "content": f"PLAYER: {choice}"},
        {"role": "assistant", "content": response.strip()},
    ])
    del SessionMessages[1:-8]

def GetCompletion(messages):
    if not USE_OPENAI:
        return f"FAKE RESPONSE for choice {messages[-1]['content']}"
    try:
        result = requests.post(
            "https://api.openai.com/v1/chat/completions",
            headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
            json={"model": "gpt-3.5-turbo", "messages": messages},
            timeout=20
        )
        if result.status_code != 200:
            return f"Error {result.status_code}: {result.text}"
        return result.json()["choices"][0]["message"]["content"].strip()
    except Exception as e:
        return f"EXCEPTION: {str(e)}"

# --- Input ---
def GetButtonChoice():
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                mx, my = pygame.mouse.get_pos()
                for i, (bx, by) in enumerate(ButtonPositions):
                    if (mx - bx) ** 2 + (my - by) ** 2 <= BUTTON_RADIUS ** 2:
                        return i + 1

# --- Game Logic ---
Display = WrappedTextDisplay()

def RunGameStep(ForcedChoice=None):
    global PlayerStats, InCombat, CombatState

    if PlayerStats["Class"] is None:
        Display.SetText("Choose your class:\\n1: Fighter\\n2: Wizard\\n3: Warlock\\n4: Paladin")
        class_choice = GetButtonChoice()
        class_map = {1: "Fighter", 2: "Wizard", 3: "Warlock", 4: "Paladin"}
        PlayerStats["Class"] = class_map.get(class_choice, "Fighter")

        if PlayerStats["Class"] == "Wizard":
            PlayerStats["HP"] = 6
            PlayerStats["MP"] = 10
            PlayerStats["SpellBook"] = ["Fireball", "Teleport", "Magic Shield"]
        elif PlayerStats["Class"] == "Warlock":
            PlayerStats["HP"] = 8
            PlayerStats["MP"] = 6
            PlayerStats["SpellBook"] = ["Hex", "Serpent Missile", "Eldritch Blast"]
        elif PlayerStats["Class"] == "Paladin":
            PlayerStats["HP"] = 10
            PlayerStats["MP"] = 8
            PlayerStats["SpellBook"] = ["Heal", "Smite", "Holy Shield"]
        else:  # Fighter
            PlayerStats["HP"] = 12
            PlayerStats["MP"] = 2
            PlayerStats["SpellBook"] = []

        Display.SetText(
            f"You are a {PlayerStats['Class']}!\\n"
            f"HP: {PlayerStats['HP']}\\n"
            f"MP: {PlayerStats['MP']}\\n"
            f"Tap to begin..."
        )
        GetButtonChoice()
        return

    if ForcedChoice:
        choice = ForcedChoice
    else:
        choice = GetButtonChoice()

    Display.AddText(f"\\nPLAYER: {choice}")
    status = f"\\nClass: {PlayerStats['Class']}, HP: {PlayerStats['HP']}, MP: {PlayerStats['MP']}, Items: {', '.join(PlayerStats['Inventory']) or 'None'}"
    prompt = MakePrompt(str(choice) + status)
    response = GetCompletion(prompt)

    if "roll initiative" in response.lower():
        InCombat = True
        match = re.search(r"A ([a-zA-Z]+) appears", response, re.IGNORECASE)
        enemy_name = match.group(1).capitalize() if match else "Unknown Foe"
        CombatState.update({
            "EnemyName": enemy_name,
            "EnemyHP": 10,
            "EnemyAttack": 3
        })
        response += f"\\n\\n[Enemy: {CombatState['EnemyName']} | HP: {CombatState['EnemyHP']} | ATK: {CombatState['EnemyAttack']}]"

    Display.SetText(response)
    RecordGameStep(choice, response)

# --- Main Loop ---
try:
    RunGameStep("New game")
    while True:
        RunGameStep()
except Exception as e:
    traceback.print_exception(e)
    Display.SetText("Error occurred. Tap to reload.")
    GetButtonChoice()
    os.execl(sys.executable, sys.executable, *sys.argv)
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Raspberry Pi 4 | main "brain" of the project | $103 | <a href="https://www.amazon.com/RasTech-Raspberry-Starter-Heatsink-Screwdriver/dp/B0C8LV6VNZ/ref=sr_1_4?crid=3506HY00MCGVM&dib=eyJ2IjoiMSJ9._zkM62vSQ8p7tNr88715LdMv_qHh72Je-tkF9PXEa3chDE53QT4aZu4AGAb4ihE61QY4ZD55nKF6Fp2Kfs8t7AbafM_JrlJFfHo9OB4eAVGqa0EB-7aoBQHPmhKHZ2MW8ny-Kd44bMVlVxPlTWVk5YHIN5P3uKVqrE5Dcal0rKkHny-O6Xyb5ux2AOU6OwVbkag_bqBX66RQNRrgBuz-0pS43mcx93IZTQA9R8NaJJypYU2HAycp-XicTFmyU60a01Nfm9iuyo6B9yA8ppN3OQQyJ-NQ9xyNPxfTLwkqtng.yAYpU6outhQcZmOZhN9Wb6yTw7A85CNUbXZguGInZNg&dib_tag=se&keywords=raspberry%2Bpi%2Bkit&qid=1718848547&s=electronics&sprefix=rasbperry%2Bpi%2Bkit%2Celectronics%2C83&sr=1-4&th=1"> Link </a> |
| 7 Inch IPS LCD Touch Screen Display Panel | Display/Monitor | $45.99 | <a href="https://www.amazon.com/Hosyond-Display-1024×600-Capacitive-Raspberry/dp/B09XKC53NH/ref=sr_1_3?crid=1KKB9WC62OIAD&keywords=raspberry%2Bpi%2Bips&qid=1685911698&s=electronics&sprefix=raspberry%2Bpi%2Bips%2B%2Celectronics%2C87&sr=1-3&th=1#customerReviews)](https://www.amazon.com/dp/B081VHSB2V?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)"> Link </a> 
| uni SD Card Reader | Lets the pc write Raspberry Pi OS on the sd card for the Raspberry Pi | $9.99 | <a href="https://www.amazon.com/dp/B081VHSB2V?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1"> Link </a> |
| Wireless Keyboard and Mouse | Easy Control of the Raspberry Pi for setup | $21.99 | <a href="https://www.amazon.com/gp/product/B07XDWCLYF/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)](https://www.amazon.com/gp/product/B07XDWCLYF/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

