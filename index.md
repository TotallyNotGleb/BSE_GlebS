# Infinte Text Adventure
Infinite Text Adventure is a touchscreen-based fantasy RPG that generates an endless, interactive story using OpenAI’s GPT model. I built a save/load system, class-based combat mechanics, and a full touchscreen interface in CircuitPython. One of the biggest challenges was getting reliable local file saving and memory persistence, which I overcame by restructuring how the session state is stored and recalled.

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|  
| Gleb S | Harvard Westlake | Aerospace Engineering | Incoming Freshman

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/HNnCgarxydE?si=Vy7CWc7EbL5VF0P_" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my last milestone, I’ve completed the Infinite Text Adventure’s core features. I finished the combat system and added unique spells for each class, with different effects depending on the character type.
I also implemented a save file system using JSON, which required modifying parts of the main code to ensure everything saved and loaded correctly. On the hardware side, I’ve started designing a 3D-printable case in CAD for the Raspberry Pi, though it still needs adjustments like holes for the studs and ports. 
One of my biggest challenges at BSE was getting started at all—on day one, I had serious technical issues including OS installation failures, Pi boot problems, and connectivity issues. I eventually realized the original project was designed for CircuitPython, not Raspberry Pi, so I had to rebuild the entire UI in Pygame.
That part went smoother thanks to my past experience. I didn’t get to integrate the OpenAI API due to time and config issues, but overall, I’m proud I stuck with it and built a working system. 
Throughout BSE, I learned a lot about Python, problem-solving, and figuring out how to build what I imagined without step-by-step instructions. 
In the future, I want to go deeper into mechanical and electrical engineering, since I’ve worked with those in past projects and want to keep building on that experience.



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/zzXAwj_7xGM?si=T3Vw3JWcp5_kf8Xx" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my last milestone, I’ve made several important updates to the Infinite Text Signature project. First, I fixed a major text display issue—now text no longer overflows and scrolls downward properly. 
I also added a class system: the player can choose a class like warlock, paladin, or wizard, and ChatGPT recognizes that class to provide context-specific choices (e.g., “contact your patron” or “cast light”). 
The combat system has been partially implemented. While integrating class-specific combat was challenging, I managed to get a basic system working where damage is calculated and subtracted during turns.
I also added dynamic NPC generation through ChatGPT, including a reoccurring NPC that changes most runs. One surprise was how well ChatGPT handles NPC logic and consistency with minimal input. Looking ahead, I plan to physically build the project into a handheld device using a 3D printer, though power supply and portability will be difficult.
I also want to replace the current placeholder story with a full lore dump from my other D&D campaign involving a vanished evil wizard. Before the final milestone, I’ll need to fully implement the new storyline, polish the combat system, and begin testing the hardware integration.

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/NKLvVwjHIGs?si=0_eYlWlbY35JMApi" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

My project is the "Infinite Text Signature," and it's a touchscreen-based text adventure game built using Python on a Raspberry Pi. The main components include a custom UI made with Pygame and plans for adding JRPG-style combat, which will use random number generation (like Python’s random) for dice rolls.
The current version is a completely new rewrite, keeping only the parts that are necessary for it to work. So far, I’ve successfully built the core interface, and it runs as intended. 
One of the main challenges was switching away from the original CircuitPython setup and figuring out how to make everything work in Pygame, but it now functions on the Pi without issue. 
In the future, I plan to add features like character classes and a combat menu with typical JRPG options like “attack,” “bag,” and “leave.”
To complete the project, I’ll continue building from the current working version, focusing next on adding combat and expanding the gameplay systems step by step.

# Schematics 
<!-- Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. -->

# Code 

```python
import os
import sys
import json
import random
import re
import traceback

import pygame
import requests  

# --------------------------------------------------------------------------------------
# Pygame Setup
# --------------------------------------------------------------------------------------
pygame.init()

# 0,0 + FULLSCREEN => use actual display size; good for Pi kiosk screens.
screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
SCREEN_WIDTH, SCREEN_HEIGHT = screen.get_size()
pygame.display.set_caption("Offline Text Adventure")

# --------------------------------------------------------------------------------------
# Game Configuration
# --------------------------------------------------------------------------------------
BUTTON_RADIUS = 80
MARGIN = 40

# NOTE: flip this to False if you're offline / quota'd-out
USE_OPENAI = True

# I couldent get the settings.toml working.
OPENAI_API_KEY = (
"your api key here")

# --------------------------------------------------------------------------------------
# Fonts
# --------------------------------------------------------------------------------------
font = pygame.font.SysFont("Arial", 24)
label_font = pygame.font.SysFont("Arial", 32, bold=True)

# --------------------------------------------------------------------------------------
# Prompt / Lore Block
# (GM instructions + worldbuilding; sent as system message at session start)
# --------------------------------------------------------------------------------------
BasePrompt = (
    "You are an AI Game Master running a sword-and-sorcery fantasy adventure.\n"
    "The player is on a quest involving danger, magic, political mystery, and ancient threats.\n"
    "At each step: describe the scene in 2-3 short sentences, list visible items or people, and offer 4 numbered actions.\n"
    "If combat begins, say ONLY 'roll initiative!' and describe the enemy. Then STOP.\n"
    "Do NOT resolve combat or actions. The system will handle it.\n"
    "If combat begins, enemy should be introduced like: '*Goblin* appeared'.\n"
    "Keep it concise, gritty, and immersive.\n\n"
    "World background:\n"
    "Twenty years ago, a nobleman named Mike Henry Smith was secretly an evil wizard. While posing as one of the elite mages, he orchestrated secret alliances with nations across Toril, plotting to seize power over the kingdom of Navarre.\n"
    "Then — he vanished. Panic swept the court. His manor was raided by a local detective who uncovered fragments of his plans.\n"
    "The king sent a squad of royal guards to catch him. They failed. In desperation, a legendary adventuring party was dispatched to the foreign realm of Bevaria to end him.\n"
    "They returned claiming success. But they were lying.\n\n"
    "What the people know / what you tell the player:\n"
    "Mike Henry Smith is remembered as a traitor who was slain two decades ago.\n"
    "But lately, rumors have spread of strange happenings in the Bevarian borderlands. Whispers of a cloaked figure matching the old wizard’s description.\n"
    "A hand-written quest note appeared on a tavern wall, asking for an independent investigation.\n"
    "You're broke. No coin, no prospects. So you took the job. That’s how your journey begins.\n\n"
    "What the Game Master secretly knows:\n"
    "The original party never found the wizard — they lied to preserve their reputation.\n"
    "In truth, Mike Henry Smith had been possessed by an ancient being — a dormant deity seeking a physical immortal form.\n"
    "Now, twenty years later, preparations for a kingdom-wide mass sacrifice are nearly complete.\n"
    "The players are walking into the final stages of a cosmic plan far bigger than they realize.\n\n"
    "Start the players in a tavern full of plot hooks: bulletin boards, suspicious patrons, rumors, odd jobs."
)

# Session message history we keep on disk. We'll send only the tail slice to GPT..
    "SpellBook": [],
}

# --------------------------------------------------------------------------------------
# Combat State (persisted)
# --------------------------------------------------------------------------------------
InCombat = False
EvadeNextHit = False       # set True by Teleport / Shield, consumed on enemy swing
AdvantageNextAttack = False  # set True by Hex, consumed on next player melee
CombatState = {
    "EnemyName": "",
    "EnemyHP": 0,
    "EnemyAttack": 0,
}

# Button overlay text (changes when we display spell menus etc.)
CurrentOptions = ["1", "2", "3", "4"]

# --------------------------------------------------------------------------------------
# UI: WrappedTextDisplay
# (centered multi-line text, manual wrapping & crude paging support)
# --------------------------------------------------------------------------------------
class WrappedTextDisplay:
    def __init__(self):
        self.line_offset = 0    # for scroll/paging (unused now, but keep)
        self.lines = []         # already-wrapped lines

    def WrapText(self, text, max_width):
        """Wrap *text* into a list of lines that fit max_width in current font."""
        wrapped_lines = []
        for paragraph in text.splitlines():
            words = paragraph.split()
            current_line = ""
            for word in words:
                test_line = (current_line + " " + word) if current_line else word
                if font.size(test_line)[0] <= max_width:
                    current_line = test_line
                else:
                    wrapped_lines.append(current_line)
                    current_line = word
            if current_line:
                wrapped_lines.append(current_line)
        return wrapped_lines

    def SetText(self, text):
        """Replace content + redraw."""
        self.lines = self.WrapText(text, SCREEN_WIDTH - 40)
        self.line_offset = 0
        self.Refresh()

    def AddText(self, text):
        """Append content + redraw."""
        self.lines.extend(self.WrapText(text, SCREEN_WIDTH - 40))
        self.Refresh()

    def Refresh(self):
        """Redraw the screen: text block centered vertical-ish + 4 corner buttons."""
        screen.fill((10, 10, 30))
        start_y = SCREEN_HEIGHT // 2 - len(self.lines) * 20  # crude center
        y = start_y
        for line in self.lines[self.line_offset : self.line_offset + 15]:
            text_surface = font.render(line, True, (255, 255, 255))
            x = SCREEN_WIDTH // 2 - text_surface.get_width() // 2
            screen.blit(text_surface, (x, y))
            y += 40
        DrawButtons(CurrentOptions)
        pygame.display.flip()

# Instantiate the text display once
Display = WrappedTextDisplay()

# --------------------------------------------------------------------------------------
# Button Layout (4 corners)
# --------------------------------------------------------------------------------------
ButtonPositions = [
    (MARGIN + BUTTON_RADIUS, MARGIN + BUTTON_RADIUS),  # top-left
    (SCREEN_WIDTH - MARGIN - BUTTON_RADIUS, MARGIN + BUTTON_RADIUS),  # top-right
    (MARGIN + BUTTON_RADIUS, SCREEN_HEIGHT - MARGIN - BUTTON_RADIUS),  # bottom-left
    (SCREEN_WIDTH - MARGIN - BUTTON_RADIUS, SCREEN_HEIGHT - MARGIN - BUTTON_RADIUS),  # bottom-right
]

def DrawButtons(labels):
    """Draw 4 circular buttons + label text (string list len 4)."""
    for i, (x, y) in enumerate(ButtonPositions):
        pygame.draw.circle(screen, (100, 0, 0), (x, y), BUTTON_RADIUS)
        pygame.draw.circle(screen, (255, 0, 0), (x, y), BUTTON_RADIUS, 4)
        label = label_font.render(str(labels[i]), True, (255, 255, 255))
        screen.blit(label, (x - label.get_width() // 2, y - label.get_height() // 2))

# --------------------------------------------------------------------------------------
# Spell Data (central; used by combat spell picker)
# values: damage tuple OR heal tuple; mp_cost; message; optional special
# --------------------------------------------------------------------------------------
spell_effects = {
    "Fireball":        {"damage": (2, 20), "mp_cost": 6, "message": "A fiery blast erupts!"},
    "Teleport":        {"damage": 0,      "mp_cost": 3, "message": "You blink away, avoiding the next attack.", "special": "evade"},
    "Magic Missle":    {"damage": (2, 8),  "mp_cost": 3, "message": "You unleash a missile of force."},
    "Magic Shield":    {"damage": 0,      "mp_cost": 3, "message": "A glowing shield appears!", "special": "evade"},
    "Heal":            {"heal":   (2, 8),  "mp_cost": 2, "message": "Holy light mends your wounds."},
    "Smite":           {"damage": (6, 14), "mp_cost": 4, "message": "Divine judgment crashes down!"},
    "Hex":             {"damage": 0,      "mp_cost": 3, "message": "A purple glow curses the enemy.", "special": "advantage"},
    "Serpent Missile": {"damage": (2, 8),  "mp_cost": 3, "message": "A snake-shaped missile flies out."},
    "Eldritch Blast":  {"damage": (6, 14), "mp_cost": 4, "message": "You unleash a chaotic burst of magic."},
}

# --------------------------------------------------------------------------------------
# Melee Class Attack Data
# --------------------------------------------------------------------------------------
ClassAttackData = {
    "Fighter": {"range": (1, 12), "message": "You perform a heavy strike!"},
    "Wizard":  {"range": (1, 4),  "message": "You poke with your staff."},
    "Warlock": {"range": (1, 6),  "message": "You unleash a shadow strike!"},
    "Paladin": {"range": (1, 10), "message": "You smite with divine fury!"},
}

# --------------------------------------------------------------------------------------
# Save / Load
# (Called often; JSON payload stores core game state. Expand later if needed.)
# --------------------------------------------------------------------------------------
def SaveGame(filename="savefile.json"):
    save_data = {
        "PlayerStats": PlayerStats,
        "CombatState": CombatState,
        "InCombat": InCombat,
        "SessionMessages": SessionMessages,
        "EvadeNextHit": EvadeNextHit,
        "AdvantageNextAttack": AdvantageNextAttack,
    }
    try:
        with open(filename, "w") as f:
            json.dump(save_data, f, indent=4)
        # print("[DEBUG] Saved.")  # noisy; comment in if debugging
    except Exception as e:
        print(f"[SAVE ERROR] {e}")

def LoadGame(filename="savefile.json"):
    global PlayerStats, CombatState, InCombat, SessionMessages, EvadeNextHit, AdvantageNextAttack
    if not os.path.exists(filename):
        return  # nothing to load
    try:
        with open(filename, "r") as f:
            save_data = json.load(f)
        PlayerStats.update(save_data.get("PlayerStats", {}))
        CombatState.update(save_data.get("CombatState", {}))
        InCombat = save_data.get("InCombat", False)
        EvadeNextHit = save_data.get("EvadeNextHit", False)
        AdvantageNextAttack = save_data.get("AdvantageNextAttack", False)
        SessionMessages[:] = save_data.get("SessionMessages", SessionMessages)
        # print("[DEBUG] Loaded save.")
    except Exception as e:
        print(f"[LOAD ERROR] {e}")

# Do an initial load before game start
LoadGame()

# --------------------------------------------------------------------------------------
# Input: Wait for one of the 4 circles (or ESC quit)
# Returns 1..4
# --------------------------------------------------------------------------------------
def GetButtonChoice():
    while True:
        for event in pygame.event.get():

            # ESC or window close => save + quit
            if event.type == pygame.QUIT or (
                event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE
            ):
                SaveGame()
                pygame.quit()
                sys.exit()

            # Touch / click => test circles
            elif event.type == pygame.MOUSEBUTTONDOWN:
                mx, my = pygame.mouse.get_pos()
                for i, (bx, by) in enumerate(ButtonPositions):
                    if (mx - bx) ** 2 + (my - by) ** 2 <= BUTTON_RADIUS ** 2:
                        return i + 1
                    
# --------------------------------------------------------------------------------------
# Combat: Player Turn Handler
# --------------------------------------------------------------------------------------
def HandlePlayerCombatTurn(choice):
    global CombatState, InCombat, EvadeNextHit, AdvantageNextAttack
    result = ""

    # -----------------------------------
    # Melee Attack
    # -----------------------------------
    if choice == 1:
        data = ClassAttackData.get(PlayerStats["Class"], {"range": (2, 4), "message": "You attack!"})
        damage = random.randint(*data["range"])
        if AdvantageNextAttack:
            damage += 4
            result += "(Advantage bonus!)\n"
            AdvantageNextAttack = False
        result += data["message"] + f"\nDealt {damage} damage to the {CombatState['EnemyName']}."
        CombatState["EnemyHP"] -= damage

    # -----------------------------------
    # Cast Spell
    # -----------------------------------
    elif choice == 2:
        if not PlayerStats["SpellBook"]:
            return "You have no spells!"
        formatted = "\n".join([f"{i+1}: {s}" for i, s in enumerate(PlayerStats["SpellBook"])])
        Display.SetText("Choose a spell:\n" + formatted)
        spell_index = GetButtonChoice()
        spell_name = PlayerStats["SpellBook"][spell_index - 1]
        spell = spell_effects.get(spell_name)
        if not spell:
            return "Nothing happens."
        if PlayerStats["MP"] < spell["mp_cost"]:
            return "Not enough Mana!"
        PlayerStats["MP"] -= spell["mp_cost"]
        result += spell["message"] + "\n"
        if "damage" in spell:
            dmg = random.randint(*spell["damage"])
            CombatState["EnemyHP"] -= dmg
            result += f"It deals {dmg} damage to the {CombatState['EnemyName']}."
        elif "heal" in spell:
            heal = random.randint(*spell["heal"])
            PlayerStats["HP"] += heal
            result += f"You heal for {heal} HP!"
        elif spell.get("special") == "evade":
            EvadeNextHit = True
            result += "You prepare to dodge the next attack."
        elif spell.get("special") == "advantage":
            AdvantageNextAttack = True
            result += "You sense an opening for a stronger hit."

    # -----------------------------------
    # Use Item
    # -----------------------------------
    elif choice == 3:
        result += "You dig through your inventory, but nothing helps..."

    # -----------------------------------
    # Run
    # -----------------------------------
    elif choice == 4:
        if random.random() < 0.5:
            InCombat = False
            result += "You escape successfully!"
        else:
            result += "You trip and land in front of the enemy!"

    # -----------------------------------
    # Win Check
    # -----------------------------------
    if CombatState["EnemyHP"] <= 0:
        InCombat = False
        result += f"\nThe {CombatState['EnemyName']} collapses. You win!"
        SessionMessages.append({"role": "user", "content": f"COMBAT LOG: Defeated {CombatState['EnemyName']}"})

    return result

# --------------------------------------------------------------------------------------
# Combat Loop
# --------------------------------------------------------------------------------------
def RunCombatLoop():
    global InCombat, EvadeNextHit, AdvantageNextAttack
    while InCombat:
        Display.SetText(
            f"{CombatState['EnemyName']} HP: {CombatState['EnemyHP']}\n"
            f"Your HP: {PlayerStats['HP']} | MP: {PlayerStats['MP']}\n"
            "Choose your action:\n1: Attack\n2: Cast Spell\n3: Use Item\n4: Run"
        )
        choice = GetButtonChoice()
        result = HandlePlayerCombatTurn(choice)

        if InCombat:
            if EvadeNextHit:
                result += f"\nYou dodge the {CombatState['EnemyName']}'s attack!"
                EvadeNextHit = False
            else:
                PlayerStats["HP"] -= CombatState["EnemyAttack"]
                result += f"\nThe {CombatState['EnemyName']} strikes you for {CombatState['EnemyAttack']} damage!"

            if PlayerStats["HP"] <= 0:
                result += "\nYou fall in battle. Game over."
                Display.SetText(result)
                SaveGame()
                pygame.time.wait(3000)
                pygame.quit()
                sys.exit()

        Display.SetText(result)
        GetButtonChoice()  # Wait for player to acknowledge

# --------------------------------------------------------------------------------------
# GPT Helper Functions
# --------------------------------------------------------------------------------------
def MakePrompt(choice):
    # Use only the last 8 messages to avoid token overflow
    recent = SessionMessages[-8:]
    return recent + [{"role": "user", "content": f"PLAYER: {choice}"}]

def RecordGameStep(choice, response):
    SessionMessages.extend([
        {"role": "user", "content": f"PLAYER: {choice}"},
        {"role": "assistant", "content": response.strip()},
    ])
    SaveGame()

def GetCompletion(messages):
    if not USE_OPENAI:
        return f"FAKE RESPONSE for choice {messages[-1]['content']}"
    try:
        result = requests.post(
            "https://api.openai.com/v1/chat/completions",
            headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
            json={"model": "gpt-3.5-turbo", "messages": messages},
            timeout=20,
        )
        if result.status_code != 200:
            return f"Error {result.status_code}: {result.text}"
        return result.json()["choices"][0]["message"]["content"].strip()
    except Exception as e:
        return f"EXCEPTION: {str(e)}"

# --------------------------------------------------------------------------------------
# Class Selection + Game Step
# --------------------------------------------------------------------------------------
def RunGameStep(ForcedChoice=None):
    global PlayerStats, InCombat, CombatState

    # ------------------------
    # Class Selection
    # ------------------------
    if PlayerStats["Class"] is None:
        Display.SetText("Choose your class:\n1: Fighter\n2: Wizard\n3: Warlock\n4: Paladin")
        class_choice = GetButtonChoice()
        class_map = {1: "Fighter", 2: "Wizard", 3: "Warlock", 4: "Paladin"}
        PlayerStats["Class"] = class_map.get(class_choice, "Fighter")

        if PlayerStats["Class"] == "Wizard":
            PlayerStats["HP"] = 8
            PlayerStats["MP"] = 15
            PlayerStats["SpellBook"] = ["Fireball", "Teleport", "Magic Missle"]
            PlayerStats["Inventory"] = ["Staff", "Ingredient Pouch"]
        elif PlayerStats["Class"] == "Warlock":
            PlayerStats["HP"] = 12
            PlayerStats["MP"] = 10
            PlayerStats["SpellBook"] = ["Hex", "Serpent Missile", "Eldritch Blast"]
            PlayerStats["Inventory"] = ["Magic Orb", "Dagger"]
        elif PlayerStats["Class"] == "Paladin":
            PlayerStats["HP"] = 14
            PlayerStats["MP"] = 10
            PlayerStats["SpellBook"] = ["Heal", "Smite", "Magic Shield"]
            PlayerStats["Inventory"] = ["Rapier", "Shield"]
        else:  # Fighter default
            PlayerStats["HP"] = 14
            PlayerStats["MP"] = 2
            PlayerStats["SpellBook"] = []
            PlayerStats["Inventory"] = ["Long Sword", "Light Shield"]

        Display.SetText(
            f"You are a {PlayerStats['Class']}!\nHP: {PlayerStats['HP']}\n"
            f"MP: {PlayerStats['MP']}\nSpells: {PlayerStats['SpellBook']}\nTap to begin..."
        )
        SaveGame()
        GetButtonChoice()
        return

    # ------------------------
    # Normal Step or Forced
    # ------------------------
    choice = ForcedChoice if ForcedChoice else GetButtonChoice()

    if InCombat:
        Display.SetText(HandlePlayerCombatTurn(choice))
        SaveGame()
        return

    Display.AddText(f"\nPLAYER: {choice}")
    status = (
        f"\nClass: {PlayerStats['Class']}, HP: {PlayerStats['HP']}, "
        f"MP: {PlayerStats['MP']}, Items: {', '.join(PlayerStats['Inventory']) or 'None'}"
    )
    prompt = MakePrompt(str(choice) + status)
    response = GetCompletion(prompt)

    if "roll initiative!" in response.lower() or "appeared" in response.lower():
        InCombat = True
        match = re.search(r"\*(\w+)\* appeared", response)
        enemy_name = match.group(1).capitalize() if match else "Unknown"
        initiative_roll = random.randint(1, 20)
        response += f"\n\n[You rolled initiative: {initiative_roll}]"
        CombatState.update({"EnemyName": enemy_name, "EnemyHP": 20, "EnemyAttack": 3})
        response += (
            f"\n[Enemy: {CombatState['EnemyName']} | HP: {CombatState['EnemyHP']} "
            f"| ATK: {CombatState['EnemyAttack']}]"
        )

    Display.SetText(response)
    RecordGameStep(choice, response)
# --------------------------------------------------------------------------------------
# Save & Load
# --------------------------------------------------------------------------------------
def SaveGame(filename="savefile.json"):
    save_data = {
        "PlayerStats": PlayerStats,
        "CombatState": CombatState,
        "InCombat": InCombat,
        "SessionMessages": SessionMessages,
    }
    with open(filename, "w") as f:
        json.dump(save_data, f)

def LoadGame(filename="savefile.json"):
    global PlayerStats, CombatState, InCombat, SessionMessages
    if os.path.exists(filename):
        with open(filename, "r") as f:
            try:
                save_data = json.load(f)
                PlayerStats.update(save_data.get("PlayerStats", {}))
                CombatState.update(save_data.get("CombatState", {}))
                InCombat = save_data.get("InCombat", False)
                SessionMessages[:] = save_data.get(
                    "SessionMessages",
                    [{"role": "system", "content": BasePrompt}]
                )
            except json.JSONDecodeError:
                print("Save file corrupted, starting fresh.")
                SessionMessages[:] = [{"role": "system", "content": BasePrompt}]

# --------------------------------------------------------------------------------------
# Initialization
# --------------------------------------------------------------------------------------
Display = WrappedTextDisplay()
LoadGame()

# --------------------------------------------------------------------------------------
# Main Loop
# --------------------------------------------------------------------------------------
try:
        # Load save
    loaded = LoadGame()
    if not loaded:
        RunGameStep("New game")
    while True:
        if InCombat:
            RunCombatLoop()
        else:
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

