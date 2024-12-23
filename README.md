import os
import time
import json

# Helper function to clear the screen
def clear_screen():
    os.system('cls' if os.name == 'nt' else 'clear')

# User class for authentication and handling user data
class User:
    def __init__(self, username, password):
        self.username = username
        self.password = password
        self.tasks = []

    def add_task(self, task):
        self.tasks.append(task)

    def remove_task(self, task_name):
        self.tasks = [task for task in self.tasks if task.name != task_name]

    def get_task_by_name(self, task_name):
        for task in self.tasks:
            if task.name == task_name:
                return task
        return None

# Task class to represent a task with status and progress tracking
class Task:
    def __init__(self, name, description):
        self.name = name
        self.description = description
        self.status = 'Not Started'  # Possible values: 'Not Started', 'In Progress', 'Completed'
        self.progress = 0  # Value between 0 and 100

    def update_status(self, status):
        self.status = status

    def update_progress(self, progress):
        if 0 <= progress <= 100:
            self.progress = progress
        else:
            print("Progress must be between 0 and 100.")

    def display(self):
        print(f"Task: {self.name}")
        print(f"Description: {self.description}")
        print(f"Status: {self.status}")
        print(f"Progress: {self.progress}%")
        print("-" * 40)

# TaskManager to manage all tasks and user data
class TaskManager:
    def __init__(self):
        self.users = {}
        self.logged_in_user = None

    def load_users(self):
        if os.path.exists("users.json"):
            with open("users.json", "r") as file:
                data = json.load(file)
                for username, user_data in data.items():
                    user = User(username, user_data["password"])
                    for task_data in user_data["tasks"]:
                        task = Task(task_data["name"], task_data["description"])
                        task.update_status(task_data["status"])
                        task.update_progress(task_data["progress"])
                        user.add_task(task)
                    self.users[username] = user

    def save_users(self):
        with open("users.json", "w") as file:
            data = {}
            for username, user in self.users.items():
                user_data = {
                    "password": user.password,
                    "tasks": [{
                        "name": task.name,
                        "description": task.description,
                        "status": task.status,
                        "progress": task.progress
                    } for task in user.tasks]
                }
                data[username] = user_data
            json.dump(data, file)

    def register_user(self):
        clear_screen()
        print("Register a new account")
        username = input("Username: ")
        if username in self.users:
            print("Username already exists. Please choose another.")
            return
        password = input("Password: ")
        self.users[username] = User(username, password)
        print("Registration successful!")
        time.sleep(2)

    def login_user(self):
        clear_screen()
        print("Login to your account")
        username = input("Username: ")
        if username not in self.users:
            print("No such user found.")
            return False
        password = input("Password: ")
        if self.users[username].password == password:
            self.logged_in_user = self.users[username]
            print(f"Welcome back, {username}!")
            time.sleep(1)
            return True
        else:
            print("Incorrect password.")
            return False

    def view_tasks(self):
        if self.logged_in_user:
            if not self.logged_in_user.tasks:
                print("No tasks available.")
            else:
                for task in self.logged_in_user.tasks:
                    task.display()
        else:
            print("You must be logged in to view tasks.")
        time.sleep(2)

    def create_task(self):
        if self.logged_in_user:
            clear_screen()
            print("Create a new task")
            name = input("Task name: ")
            description = input("Task description: ")
            task = Task(name, description)
            self.logged_in_user.add_task(task)
            print(f"Task '{name}' created successfully.")
        else:
            print("You must be logged in to create tasks.")
        time.sleep(2)

    def update_task(self):
        if self.logged_in_user:
            clear_screen()
            print("Update an existing task")
            task_name = input("Enter task name to update: ")
            task = self.logged_in_user.get_task_by_name(task_name)
            if task:
                print("1. Update status")
                print("2. Update progress")
                choice = input("Choose option: ")
                if choice == '1':
                    status = input("Enter new status (Not Started, In Progress, Completed): ")
                    task.update_status(status)
                    print(f"Task '{task_name}' status updated to {status}.")
                elif choice == '2':
                    progress = int(input("Enter new progress (0-100): "))
                    task.update_progress(progress)
                    print(f"Task '{task_name}' progress updated to {progress}%.")
                else:
                    print("Invalid option.")
            else:
                print("Task not found.")
        else:
            print("You must be logged in to update tasks.")
        time.sleep(2)

    def delete_task(self):
        if self.logged_in_user:
            clear_screen()
            print("Delete a task")
            task_name = input("Enter task name to delete: ")
            self.logged_in_user.remove_task(task_name)
            print(f"Task '{task_name}' deleted.")
        else:
            print("You must be logged in to delete tasks.")
        time.sleep(2)

    def logout(self):
        self.logged_in_user = None
        print("Logged out successfully.")
        time.sleep(2)

    def menu(self):
        while True:
            clear_screen()
            print("Task Manager Menu")
            if self.logged_in_user:
                print(f"Logged in as: {self.logged_in_user.username}")
                print("1. View Tasks")
                print("2. Create Task")
                print("3. Update Task")
                print("4. Delete Task")
                print("5. Logout")
            else:
                print("1. Login")
                print("2. Register")
            print("0. Exit")
            choice = input("Choose an option: ")
            
            if choice == '0':
                break
            elif choice == '1' and self.logged_in_user:
                self.view_tasks()
            elif choice == '2' and self.logged_in_user:
                self.create_task()
            elif choice == '3' and self.logged_in_user:
                self.update_task()
            elif choice == '4' and self.logged_in_user:
                self.delete_task()
            elif choice == '5' and self.logged_in_user:
                self.logout()
            elif choice == '1' and not self.logged_in_user:
                if self.login_user():
                    continue
            elif choice == '2' and not self.logged_in_user:
                self.register_user()
            else:
                print("Invalid option, please try again.")
                time.sleep(1)

if __name__ == "__main__":
    task_manager = TaskManager()
    task_manager.load_users()
    task_manager.menu()
    task_manager.save_users()




import random
import time

class Item:
    def __init__(self, name, description):
        self.name = name
        self.description = description

    def __str__(self):
        return f"{self.name}: {self.description}"

class Room:
    def __init__(self, name, description):
        self.name = name
        self.description = description
        self.items = []
        self.exits = {}
        self.npc = None

    def add_item(self, item):
        self.items.append(item)

    def add_exit(self, direction, room):
        self.exits[direction] = room

    def set_npc(self, npc):
        self.npc = npc

    def get_item_by_name(self, item_name):
        for item in self.items:
            if item.name.lower() == item_name.lower():
                return item
        return None

    def __str__(self):
        return f"{self.name}: {self.description}"

class NPC:
    def __init__(self, name, greeting, dialogue):
        self.name = name
        self.greeting = greeting
        self.dialogue = dialogue

    def interact(self):
        print(self.greeting)
        time.sleep(1)
        print(self.dialogue)

class Player:
    def __init__(self, name):
        self.name = name
        self.inventory = []
        self.current_room = None

    def move(self, direction):
        if direction in self.current_room.exits:
            self.current_room = self.current_room.exits[direction]
            print(f"\nYou have moved to the {self.current_room.name}.")
            print(self.current_room.description)
        else:
            print("You can't go that way.")

    def take_item(self, item_name):
        item = self.current_room.get_item_by_name(item_name)
        if item:
            self.inventory.append(item)
            self.current_room.items.remove(item)
            print(f"\nYou took the {item.name}.")
        else:
            print("Item not found in this room.")

    def drop_item(self, item_name):
        item = next((i for i in self.inventory if i.name.lower() == item_name.lower()), None)
        if item:
            self.inventory.remove(item)
            self.current_room.add_item(item)
            print(f"\nYou dropped the {item.name}.")
        else:
            print("You don't have that item.")

    def show_inventory(self):
        if not self.inventory:
            print("\nYour inventory is empty.")
        else:
            print("\nYour inventory:")
            for item in self.inventory:
                print(f"- {item.name}")

    def interact_with_npc(self):
        if self.current_room.npc:
            self.current_room.npc.interact()
        else:
            print("\nThere's no one to talk to here.")

    def get_current_room(self):
        return self.current_room

def create_rooms():
    # Room 1: Entrance Hall
    hall = Room("Entrance Hall", "A grand entrance hall with marble floors. A chandelier hangs from the ceiling.")
    sword = Item("Sword", "A sharp, gleaming sword.")
    hall.add_item(sword)

    # Room 2: Library
    library = Room("Library", "A room filled with bookshelves. The smell of old paper fills the air.")
    book = Item("Old Book", "An ancient book that looks valuable.")
    library.add_item(book)

    # Room 3: Kitchen
    kitchen = Room("Kitchen", "A warm, cozy kitchen. A fire crackles in the hearth.")
    apple = Item("Apple", "A shiny red apple, perfect for eating.")
    kitchen.add_item(apple)

    # Room 4: Throne Room
    throne_room = Room("Throne Room", "A majestic throne room with golden pillars.")
    throne_npc = NPC("King", "Welcome, brave adventurer!", "I am the King of this realm. How may I assist you?")
    throne_room.set_npc(throne_npc)

    # Exits between rooms
    hall.add_exit("north", library)
    hall.add_exit("east", kitchen)
    library.add_exit("south", hall)
    kitchen.add_exit("west", hall)
    kitchen.add_exit("north", throne_room)
    throne_room.add_exit("south", kitchen)

    return hall  # Starting room

def print_menu():
    print("\nWhat would you like to do?")
    print("1. Move")
    print("2. Take item")
    print("3. Drop item")
    print("4. Show inventory")
    print("5. Talk to NPC")
    print("6. Quit")

def main():
    print("Welcome to the Adventure Game!")
    name = input("Enter your character's name: ")
    player = Player(name)
    starting_room = create_rooms()
    player.current_room = starting_room

    print(f"\nYou are standing in the {player.get_current_room().name}.")
    print(player.get_current_room().description)

    while True:
        print_menu()
        choice = input("\nEnter your choice: ")

        if choice == '1':  # Move
            direction = input("Which direction would you like to go? (north, south, east, west): ").lower()
            player.move(direction)

        elif choice == '2':  # Take item
            item_name = input("Enter the name of the item you want to take: ").strip()
            player.take_item(item_name)

        elif choice == '3':  # Drop item
            item_name = input("Enter the name of the item you want to drop: ").strip()
            player.drop_item(item_name)

        elif choice == '4':  # Show inventory
            player.show_inventory()

        elif choice == '5':  # Talk to NPC
            player.interact_with_npc()

        elif choice == '6':  # Quit
            print("Thanks for playing the Adventure Game!")
            break

        else:
            print("Invalid choice. Please try again.")

        time.sleep(1)

if __name__ == "__main__":
    main()













import random
import time
import sys

# --- Helper Functions ---

def clear_screen():
    sys.stdout.write("\033[H\033[J")
    sys.stdout.flush()

def wait_for_input():
    input("\nPress Enter to continue...")

# --- Classes for RPG System ---

class Item:
    def __init__(self, name, description, item_type, power=0):
        self.name = name
        self.description = description
        self.item_type = item_type
        self.power = power

    def use(self, character):
        if self.item_type == "healing":
            character.health += self.power
            print(f"{character.name} used {self.name} and healed {self.power} health!")
        elif self.item_type == "weapon":
            print(f"{character.name} equips {self.name}. Attack power is increased by {self.power}.")
            character.attack_power += self.power

    def __str__(self):
        return f"{self.name}: {self.description}"

class Character:
    def __init__(self, name, health, attack_power, defense):
        self.name = name
        self.health = health
        self.attack_power = attack_power
        self.defense = defense
        self.inventory = []
        self.is_alive = True

    def attack(self, enemy):
        damage = max(0, self.attack_power - enemy.defense)
        print(f"{self.name} attacks {enemy.name} for {damage} damage!")
        enemy.health -= damage
        if enemy.health <= 0:
            enemy.is_alive = False
            print(f"{enemy.name} has been defeated!")

    def take_damage(self, damage):
        damage_taken = max(0, damage - self.defense)
        self.health -= damage_taken
        print(f"{self.name} takes {damage_taken} damage. Health: {self.health}")
        if self.health <= 0:
            self.is_alive = False
            print(f"{self.name} has been defeated!")

    def heal(self, amount):
        self.health += amount
        print(f"{self.name} heals {amount} health. Health: {self.health}")

    def add_item(self, item):
        self.inventory.append(item)

    def show_inventory(self):
        if not self.inventory:
            print(f"{self.name}'s inventory is empty.")
        else:
            print(f"{self.name}'s Inventory:")
            for item in self.inventory:
                print(f"- {item.name}: {item.description}")

    def show_stats(self):
        print(f"{self.name}'s Stats:")
        print(f"Health: {self.health}")
        print(f"Attack Power: {self.attack_power}")
        print(f"Defense: {self.defense}")

    def is_alive(self):
        return self.health > 0

class Enemy(Character):
    def __init__(self, name, health, attack_power, defense):
        super().__init__(name, health, attack_power, defense)

class Room:
    def __init__(self, name, description):
        self.name = name
        self.description = description
        self.exits = {}
        self.items = []
        self.enemy = None
        self.npc = None

    def add_exit(self, direction, room):
        self.exits[direction] = room

    def add_item(self, item):
        self.items.append(item)

    def set_enemy(self, enemy):
        self.enemy = enemy

    def set_npc(self, npc):
        self.npc = npc

    def get_item_by_name(self, item_name):
        for item in self.items:
            if item.name.lower() == item_name.lower():
                return item
        return None

    def __str__(self):
        return f"{self.name}: {self.description}"

class NPC:
    def __init__(self, name, greeting, dialogue):
        self.name = name
        self.greeting = greeting
        self.dialogue = dialogue

    def interact(self):
        print(f"{self.greeting}")
        time.sleep(1)
        print(f"{self.dialogue}")

class Player(Character):
    def __init__(self, name):
        super().__init__(name, 100, 10, 5)
        self.quest_items = []

    def move(self, direction):
        if direction in self.current_room.exits:
            self.current_room = self.current_room.exits[direction]
            print(f"\nYou have moved to the {self.current_room.name}.")
            print(self.current_room.description)
        else:
            print("You can't go that way.")

    def take_item(self, item_name):
        item = self.current_room.get_item_by_name(item_name)
        if item:
            self.inventory.append(item)
            self.current_room.items.remove(item)
            print(f"\nYou took the {item.name}.")
        else:
            print("Item not found in this room.")

    def drop_item(self, item_name):
        item = next((i for i in self.inventory if i.name.lower() == item_name.lower()), None)
        if item:
            self.inventory.remove(item)
            self.current_room.add_item(item)
            print(f"\nYou dropped the {item.name}.")
        else:
            print("You don't have that item.")

    def heal(self, item):
        item.use(self)

    def interact_with_npc(self):
        if self.current_room.npc:
            self.current_room.npc.interact()
        else:
            print("\nThere's no one to talk to here.")

class Game:
    def __init__(self):
        self.rooms = {}
        self.player = None

    def create_rooms(self):
        # Room 1: Start Room
        start_room = Room("Start Room", "The beginning of your adventure. There is a door to the north.")
        sword = Item("Sword", "A sharp sword, perfect for battle.", "weapon", 5)
        healing_potion = Item("Healing Potion", "Restores 50 health.", "healing", 50)
        start_room.add_item(sword)
        start_room.add_item(healing_potion)

        # Room 2: Forest
        forest_room = Room("Forest", "A dense forest with trees all around. You hear rustling.")
        monster = Enemy("Goblin", 30, 8, 2)
        forest_room.set_enemy(monster)

        # Room 3: Village
        village_room = Room("Village", "A peaceful village with a well and a small shop.")
        npc = NPC("Merchant", "Welcome to my shop!", "I have potions for sale. Come back soon!")
        village_room.set_npc(npc)

        # Exits between rooms
        start_room.add_exit("north", forest_room)
        forest_room.add_exit("south", start_room)
        forest_room.add_exit("east", village_room)
        village_room.add_exit("west", forest_room)

        self.rooms["start"] = start_room
        self.rooms["forest"] = forest_room
        self.rooms["village"] = village_room

    def create_player(self):
        name = input("Enter your character's name: ")
        self.player = Player(name)
        self.player.current_room = self.rooms["start"]
        print(f"\nWelcome, {self.player.name}!")
        print(self.player.current_room.description)

    def game_loop(self):
        while self.player.is_alive:
            self.show_menu()

    def show_menu(self):
        print("\nWhat would you like to do?")
        print("1. Move")
        print("2. Take Item")
        print("3. Drop Item")
        print("4. Show Inventory")
        print("5. Heal")
        print("6. Show Stats")
        print("7. Talk to NPC")
        print("8. Quit")

        choice = input("\nEnter your choice: ")

        if choice == '1':  # Move
            direction = input("Which direction would you like to go? (north, south, east, west): ").lower()
            self.player.move(direction)

        elif choice == '2':  # Take Item
            item_name = input("Enter the name of the item you want to take: ").strip()
            self.player.take_item(item_name)

        elif choice == '3':  # Drop Item
            item_name = input("Enter the name of the item you want to drop: ").strip()
            self.player.drop_item(item_name)

        elif choice == '4':  # Show Inventory
            self.player.show_inventory()

        elif choice == '5':  # Heal
            if self.player.inventory:
                item_name = input("Which item do you want to use to heal? ").strip()
                item = next((i for i in self.player.inventory if i.name.lower() == item_name.lower()), None)
                if item and item.item_type == "healing":
                    self.player.heal(item)
                else:
                    print("You don't have a healing item or it's not valid.")
            else:
                print("Your inventory is empty.")

        elif choice == '6':  # Show Stats
            self.player.show_stats()

        elif choice == '7':  # Talk to NPC
            self.player.interact_with_npc()

        elif choice == '8':  # Quit
            print("Thanks for playing!")
            break

        else:
            print("Invalid option. Please try

