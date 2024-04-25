# PantryManager
An application that manages your pantry inventory, hosts your recipes, and manages your shopping list. All in one.

This app will first have you inventory your pantry, including how much of something you have. It will use conversions as needed. Then you enter your recipes, and it will automatically make shopping lists for each recipe, based on what you do and don't have in your pantry. There will be notifications to review/update your inventory, defaults to every Sunday.

Stretch goals: 
- use inventory discover new recipes
- integration with Flipp
- integration with Instacart
- integration with Notion/Obsidian/Logseq/Zettlr
- Integration with Todoist/TickTick
- Nextcloud integration

Up for debate: whether to use AI to generate recipes based on the pantry.

When you make a recipe, mark it as Complete and the app will automatically deduct the amounts from your inventory.

---

When you fill out your shopping list, this will update your inventory. Current planned implementation looks like this:

```mermaid
sequenceDiagram
    participant User
    participant UI as Shopping List UI
    participant Server as Backend Server
    participant DB as Database

    User->>+UI: Check item (item, quantity)
    UI->>+Server: Update pantry (item, quantity)
    Server->>+DB: Update inventory (item, quantity)
    DB-->>-Server: Confirmation
    Server-->>-UI: Update status
    UI-->>-User: Display success message
```

```mermaid
classDiagram
    class ShoppingList {
      +List~Item~ items
      +addItem(item : Item)
      +removeItem(item : Item)
      +checkOffItem(item : Item)
    }

    class Item {
      +String name
      +int quantity
      +String category
    }

    class Pantry {
      +List~Item~ inventoryItems
      +addToInventory(item : Item)
      +removeFromInventory(item : Item)
      +updateQuantity(item : Item, quantity : int)
    }

    class User {
      +String name
      +String email
    }

    class Database {
      +connect()
      +update(query : String)
      +query(query : String) : List~Item~
    }

    User -- ShoppingList
    User -- Pantry
    Database "1" -- "1..*" Item : stores
```

