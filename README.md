# PantryManager

An application that manages your pantry inventory, hosts your recipes, and manages your shopping list. All in one.

This app will first have you inventory your pantry, including how much of something you have. It will use conversions as needed. Then you enter your recipes, and it will automatically make shopping lists for each recipe, based on what you do and don't have in your pantry. There will be notifications to review/update your inventory, defaults to every Sunday.

Stretch goals:

- use inventory discover new recipes
- Analytics
  - Ingredient use over time
  - Price of ingredients over time
  - (?) Price Book: track prices over time at multiple supermarkets
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

    User->>+UI: Check item (item, quantity, ?expirationDate)
    UI->>+Server: Update pantry (item, quantity, ?expirationDate)
    Server->>+DB: Update inventory (item, quantity. ?expirationDate)
    DB-->>-Server: Confirmation
    Server-->>-UI: Update status
    UI-->>-User: Display success message
```

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite" // Now using SQLite as the database provider
  url      = "file:./dev.db" // The database file will be created in the current directory
}

model Ingredient {
  id            Int       @id @default(autoincrement())
  name          String    @unique
  quantity      Float     // Quantity in a specific unit
  unit          String    // e.g., 'grams', 'liters', 'pieces'
  pantryId      Int
  pantry        Pantry    @relation(fields: [pantryId], references: [id])
  recipes       Recipe[]  @relation("RecipeIngredients")
  expirationDate DateTime? // Optional field for perishable ingredients
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Pantry {
  id            Int         @id @default(autoincrement())
  name          String
  ingredients   Ingredient[]
  shoppingLists ShoppingList[]
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
}

model ShoppingList {
  id            Int          @id @default(autoincrement())
  name          String
  pantryId      Int
  pantry        Pantry       @relation(fields: [pantryId], references: [id])
  items         ShoppingListItem[]
  createdAt     DateTime     @default(now())
  updatedAt     DateTime     @updatedAt
}

model ShoppingListItem {
  id             Int        @id @default(autoincrement())
  shoppingListId Int
  ingredientId   Int
  quantity       Float
  shoppingList   ShoppingList @relation(fields: [shoppingListId], references: [id])
  ingredient     Ingredient    @relation(fields: [ingredientId], references: [id])
}

model Recipe {
  id            Int        @id @default(autoincrement())
  name          String
  instructions  String
  ingredients   Ingredient[] @relation("RecipeIngredients")
  pantryId      Int
  pantry        Pantry      @relation(fields: [pantryId], references: [id])
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
}
```