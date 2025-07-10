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
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/CaCazFBhYKs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your first milestone, describe what your project is and how you plan to build it. You can include:
- An explanation about the different components of your project and how they will all integrate together
- Technical progress you've made so far
- Challenges you're facing and solving in your future milestones
- What your plan is to complete your project

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

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
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")    

# --- Prompt Setup ---
BasePrompt = (
    "You are an AI Game Master running a sword-and-sorcery fantasy adventure.\n"
    "The player is on a quest involving danger, magic, and ancient ruins.\n"
    "At each step: describe the scene briefly, list items, and offer 4 actions.\n"
    "The player cannot die. Keep it short and terse."
)
SessionMessages = [{"role": "system", "content": BasePrompt}]

# --- Pygame Initialization ---
pygame.init()
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Text Adventure (Online)")
font = pygame.font.SysFont("Arial", 24)
label_font = pygame.font.SysFont("Arial", 32, bold=True)

# --- UI Class ---
class WrappedTextDisplay:
    def __init__(self):
        self.Lines = []

    def SetText(self, text):
        self.Lines = text.splitlines()
        self.Refresh()

    def AddText(self, text):
        self.Lines.extend(text.splitlines())
        self.Refresh()

    def Refresh(self):
        screen.fill((10, 10, 30))
        y = 20
        for line in self.Lines[-12:]:
            rendered = font.render(line, True, (255, 255, 255))
            screen.blit(rendered, (SCREEN_WIDTH // 2 - rendered.get_width() // 2, y))
            y += 32
        DrawButtons(ButtonLabels)
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
    del SessionMessages[1:-8]  # keep recent history only

def GetCompletion(messages):
    if not USE_OPENAI:
        return f"FAKE RESPONSE for choice {messages[-1]['content']}"
    try:
        result = requests.post(
            "https://api.openai.com/v1/chat/completions",
            headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
            json={"model": "gpt-3.5-turbo", "messages": messages},
            timeout=10
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
    if ForcedChoice:
        choice = ForcedChoice
    else:
        choice = GetButtonChoice()
    Display.AddText(f"\nPLAYER: {choice}")
    response = GetCompletion(MakePrompt(choice))
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
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

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

To watch the BSE tutorial on how to create a portfolio, click here.
