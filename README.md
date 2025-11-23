# Battlestar RPG

## Project Overview

Battlestar RPG is a terminal-based, turn-based combat game implemented in C++ with a focus on clean architecture and object-oriented design principles. The project demonstrates modular class design, separation of concerns, and systematic application of SOLID principles across a complete game engine. The architecture centers on a state-driven game loop that manages combat encounters, inventory systems, map navigation, and persistent game state through serialization. The implementation leverages custom data structures including a MaxHeap for turn order determination and multiple sorting algorithms for inventory management, all built on C++17 standards with modern memory management practices using smart pointers.

## Features

- Turn-based combat system with speed-based turn ordering using MaxHeap data structure
- Difficulty scaling system with configurable stat multipliers affecting attack and health modifiers
- Inventory management with dynamic capacity, item stacking, and multiple sorting strategies
- Stat-based progression system tracking health, defense, damage, speed, and experience
- Map and room navigation system with enemy and item distribution
- State machine architecture for game flow management (MainMenu, InGame, GameOver)
- Serialization and deserialization for save/load game functionality
- Polymorphic item system with inheritance hierarchy (Weapon, Armour, Potion)
- Merge sort and insertion sort implementations for inventory organization
- Template-based save/load system with file I/O abstraction

## Tech Stack

- **Language:** C++17
- **Build System:** CMake 3.13+, Make
- **Testing Framework:** GoogleTest
- **Standard Library:** STL containers (vector, unique_ptr, map, string), algorithms, filesystem
- **Development Tools:** Git, VS Code, Terminal

## Architecture

The codebase is organized into distinct class groups that handle specific responsibilities:

**Core Game Engine:**
- `Game`: Central state machine managing game loop, state transitions, and component coordination
- `GameState`: Enum-based state management for MainMenu, InGame, and GameOver states

**Combat System:**
- `Combat`: Manages battle encounters, turn order calculation, and combat resolution
- `Character`: Encapsulates player and enemy entities with stats, equipment, and inventory references
- `MaxHeap`: Custom heap implementation for O(n log n) turn order sorting based on character speed

**Item System:**
- `Item`: Abstract base class defining polymorphic interface for all game items
- `Weapon`, `Armour`, `Potion`: Derived classes implementing specific item behaviors
- `ItemStack`: Wrapper class managing item quantities and ownership via unique_ptr
- `Inventory`: Container class managing item storage with dynamic capacity and sorting capabilities

**Map and Navigation:**
- `Map`: Manages room collection, player position tracking, and spatial state
- `Room`: Encapsulates room-specific data including enemy vectors and item vectors

**Game Configuration:**
- `Difficulty`: Enum-based difficulty system with multiplier calculations for attack and health
- `MainMenu`: Handles user input normalization, menu navigation, and option selection

**Persistence:**
- `SaveGame`: Template-based serialization system writing game state to files
- `LoadGame`: Template-based deserialization system reading game state from files

**Sorting System:**
- `AbstractItemSort`: Base class defining sorting interface following Strategy pattern
- `MergeSort`: O(n log n) merge sort implementation for inventory organization
- `CompareItem`: Comparator utility supporting multiple comparison modes (name, time, type)

Class relationships follow composition and aggregation patterns. `Game` aggregates `Character`, `Map`, `Inventory`, `Difficulty`, and `MainMenu`. `Character` contains a unique_ptr to `Inventory` and manages equipment via unique_ptr to `Weapon` and `Armour`. `Inventory` stores a vector of unique_ptr to `ItemStack`, which in turn owns unique_ptr to `Item`. The polymorphic item hierarchy enables runtime type dispatch through virtual methods. `Combat` receives a vector of Character pointers and uses `MaxHeap` for turn order calculation without owning the characters.

Game state flows through the `Game::run()` method, which switches between states and delegates to appropriate handlers. The combat system integrates with the map system to retrieve room-specific enemies, and the inventory system provides item access during combat encounters.

## Core Systems

### Combat System

The combat system implements turn-based battle resolution using a speed-based priority queue. `Combat::startBattle()` initializes a `MaxHeap` and uses `heapsort()` to order fighters by speed stat. The heap maintains max-heap property where parent nodes have higher speed than children, enabling O(log n) insertion and O(n log n) sorting. Each combat round iterates through the sorted turn order, with enemies automatically targeting the player and the player selecting targets interactively. Attack calculations incorporate equipped weapon damage, character base damage, and difficulty multipliers. Defense reduces incoming damage before health deduction. The system tracks battle state through boolean flags and vector size checks, removing defeated characters from the active fighters vector.

### Inventory & Items

The inventory system uses a vector of `unique_ptr<ItemStack>` to manage item storage with automatic memory management. `ItemStack` wraps `unique_ptr<Item>` and tracks quantity, enabling stackable items. The `Inventory` class provides O(n) linear search for item lookup by name, type, or index. Capacity management includes dynamic resizing and percentage-based capacity increases. Sorting operations support multiple strategies: `MergeSort` provides O(n log n) performance for large inventories, while `InsertionSort` offers O(n^2) but simpler implementation for smaller collections. The `CompareItem` utility supports sorting by name (lexicographic), time earned (chronological), or type (categorical), with ascending or descending order via `SortOrder` enum. Item polymorphism enables runtime dispatch through virtual `useItem(Character&)` method, allowing weapons to modify damage, armour to modify defense, and potions to restore health.

### Movement / Map / Room System

The map system represents game space as a collection of `Room` objects stored in a vector within the `Map` class. Each `Room` contains vectors of `Character*` (enemies) and `Item*` (items), with the map maintaining a `playerIndex` tracking current room position. Navigation uses directional string input normalized to lowercase, with validation ensuring valid room transitions. The `Map::distributeEnemiesAndItems()` method initializes room contents based on weapon type selection, creating appropriate enemy and item distributions. Room state queries (`roomHasEnemies()`, `roomHasItems()`, `roomHasPotions()`) enable conditional game logic. Experience calculation aggregates room-specific experience values when rooms are cleared.

### Difficulty System

The difficulty system implements a strategy pattern through enum-based configuration. Three difficulty levels (Rookie, Elite, Battlestar) each provide distinct multiplier values via `getAttackMultiplier()` and `getHealthModifier()`. Rookie mode applies 1.5x attack and 1.2x health multipliers favoring the player, while Battlestar mode applies 0.8x multipliers increasing challenge. The `Difficulty` class normalizes input strings to lowercase for case-insensitive matching and throws `invalid_argument` exceptions for invalid difficulty strings. Multipliers are applied during game initialization and after save/load operations through `Game::applyDifficulty()`.

### Data Structures

**MaxHeap:** Custom heap implementation using a vector of `Character*` pointers. The heap maintains max-heap property through `heapifyUp()` and `heapifyDown()` methods, both operating in O(log n) time. `heapsort()` implements the standard heap sort algorithm: first building a max-heap in O(n) time by heapifying from the last parent node, then extracting maximum elements in O(n log n) time. The implementation uses zero-based indexing with parent-child relationships calculated as `parent = (index - 1) / 2`, `leftChild = 2 * index + 1`, `rightChild = 2 * index + 2`.

**STL Usage:** The codebase extensively uses `std::vector` for dynamic arrays, `std::unique_ptr` for automatic memory management following RAII principles, `std::string` for text processing, `std::map` for key-value lookups in menu systems, and `std::filesystem` for save file management. Smart pointer usage eliminates manual memory management and prevents memory leaks.

## SOLID Principle Implementation

**Single Responsibility Principle (SRP):** Each class encapsulates one well-defined responsibility. `Combat` handles only battle logic, `Inventory` manages only item storage, `Difficulty` handles only difficulty configuration, `Map` manages only spatial navigation, and `SaveGame`/`LoadGame` handle only serialization concerns. The `Character` class manages character state and delegates inventory operations to the `Inventory` class rather than implementing storage logic itself.

**Open/Closed Principle (OCP):** The sorting system demonstrates OCP through the `AbstractItemSort` base class. New sorting algorithms can be added by inheriting from `AbstractItemSort` and implementing the `sort()` method without modifying existing code. The `Item` hierarchy allows extension through inheritance (adding new item types like `Weapon`, `Armour`, `Potion`) without modifying the base `Item` class. The `Difficulty` class can be extended with new difficulty levels by adding enum values and corresponding multiplier cases.

**Liskov Substitution Principle (LSP):** All derived item classes (`Weapon`, `Armour`, `Potion`) properly implement the `Item` interface, ensuring they can be used interchangeably through base class pointers. The virtual `useItem(Character&)` method maintains behavioral consistency across subclasses. `MergeSort` and any future sorting implementations can substitute for `AbstractItemSort` without breaking client code expectations.

**Interface Segregation Principle (ISP):** The codebase separates concerns into focused interfaces. `Combat` does not depend on inventory management details, `Inventory` does not depend on combat logic, and `Map` does not depend on item internals. The `CompareItem` utility provides a minimal interface for comparison operations without exposing internal sorting implementation details. `SaveGame` and `LoadGame` use template parameters to work with any serializable type without requiring a common base interface.

**Dependency Inversion Principle (DIP):** High-level modules depend on abstractions rather than concrete implementations. `Combat` depends on `Character*` pointers and uses `MaxHeap` as an abstraction for turn ordering rather than hardcoding sorting logic. The inventory sorting system depends on `AbstractItemSort` abstraction rather than concrete sorting implementations. `Game` depends on `MainMenu`, `Difficulty`, `Map`, and `Character` interfaces rather than implementation details. The save/load system uses template-based serialization, allowing any type implementing `serialize()` and `deserialize()` methods to be persisted without coupling to specific game state structures.

## Installation & Usage

### Prerequisites

- C++17 compatible compiler (GCC 7+, Clang 5+, MSVC 2017+)
- CMake 3.13 or higher
- Make
- GoogleTest (included as subdirectory)

### Building the Project

```bash
# Clone the repository
git clone https://github.com/aryamohammadi/rpgGame.git
cd rpgGame

# Generate build files
cmake .

# Compile the project
make

# Run the game
./bin/Game
```

The build process compiles all source files from `CPP_Files/` and links them into the `Game` executable in the `bin/` directory. Header files are included from the `header/` directory.

## Testing

The project uses GoogleTest for unit testing with comprehensive test coverage across core systems. Test files are located in the `Tests/` directory and cover:

- **Combat System:** `combat_test.cpp` validates turn order, attack calculations, and battle state management
- **Character System:** `CharacterTest.cpp` and `HungryCharacterTest.cpp` test stat management, equipment, and item usage
- **Inventory & Items:** `HungryInventory&ItemTest.cpp` verifies item storage, retrieval, and stacking behavior
- **Sorting Algorithms:** `HungrySortTest.cpp` validates merge sort and insertion sort correctness
- **Difficulty System:** `DifficultyTests.cpp` tests multiplier calculations and input normalization
- **Map System:** `Test-Map.cpp` verifies room navigation and state management
- **Game State:** `GameTests.cpp` tests game loop, state transitions, and initialization
- **Persistence:** `SaveGameTests.cpp` and `LoadGameTests.cpp` validate serialization and deserialization
- **Menu System:** `MainMenuTests.cpp` tests input normalization and option selection

To run tests, uncomment the appropriate test executable sections in `CMakeLists.txt`, rebuild, and execute test binaries from the `bin/` directory. Tests follow the Arrange-Act-Assert pattern and use GoogleTest assertions (`EXPECT_EQ`, `ASSERT_TRUE`, etc.) for validation.

## Project Structure

```
Battlestar-RPG/
├── CMakeLists.txt          # CMake build configuration
├── LICENSE                  # MIT License
├── README.md               # Project documentation
├── CPP_Files/              # Implementation files
│   ├── armour.cpp
│   ├── character.cpp
│   ├── combat.cpp
│   ├── compare.cpp
│   ├── Difficulty.cpp
│   ├── Game.cpp
│   ├── Heap.cpp
│   ├── inventory.cpp
│   ├── item.cpp
│   ├── itemStack.cpp
│   ├── LoadGame.cpp
│   ├── main.cpp
│   ├── MainMenu.cpp
│   ├── Map.cpp
│   ├── mergeSortItem.cpp
│   ├── potion.cpp
│   ├── Room.cpp
│   ├── SaveGame.cpp
│   └── weapon.cpp
├── header/                 # Header files
│   ├── AttackType.h
│   ├── character.h
│   ├── combat.h
│   ├── compare.h
│   ├── compareBy.h
│   ├── Difficulty.h
│   ├── Game.h
│   ├── GameState.h
│   ├── Heap.h
│   ├── insertionSort.h
│   ├── inventory.h
│   ├── item.h
│   ├── itemStack.h
│   ├── itemType.h
│   ├── LoadGame.h
│   ├── MainMenu.h
│   ├── Map.h
│   ├── mergeSort.h
│   ├── MockGameState.h
│   ├── Room.h
│   ├── SaveGame.h
│   ├── sort.h
│   └── sortorder.h
├── Tests/                  # GoogleTest unit tests
│   ├── CharacterTest.cpp
│   ├── combat_test.cpp
│   ├── DifficultyTests.cpp
│   ├── GameTests.cpp
│   ├── HungryCharacterTest.cpp
│   ├── HungryInventory&ItemTest.cpp
│   ├── HungrySortTest.cpp
│   ├── LoadGameTests.cpp
│   ├── MainMenuTests.cpp
│   ├── SaveGameTests.cpp
│   └── Test-Map.cpp
├── googletest/             # GoogleTest framework
├── Images/                 # Screenshots and diagrams
│   ├── Navigation_Diagram.png
│   ├── Sample_Output.png
│   ├── Screenshot_1.png
│   ├── Screenshot_2.png
│   ├── Screenshot_3.png
│   ├── Screenshot_4.png
│   ├── UML_Diagram.png
│   └── User_Interface_Diagram.png
└── bin/                    # Compiled executables (generated)
```

## Screenshots

The project includes visual documentation of the game interface and system architecture:

| Main Menu | Combat View | Victory Screen | Inventory Menu |
|-----------|-------------|----------------|----------------|
| ![Main Menu](Images/Screenshot_1.png) | ![Combat View](Images/Screenshot_2.png) | ![Victory Screen](Images/Screenshot_3.png) | ![Inventory Menu](Images/Screenshot_4.png) |

### Architecture Diagrams

- **Navigation Flow:** [Navigation_Diagram.png](Images/Navigation_Diagram.png) - Illustrates game state transitions and user flow
- **Sample Output:** [Sample_Output.png](Images/Sample_Output.png) - Example terminal output during gameplay
- **UML Class Structure:** [UML_Diagram.png](Images/UML_Diagram.png) - Class relationships and inheritance hierarchy
- **User Interface Layout:** [User_Interface_Diagram.png](Images/User_Interface_Diagram.png) - Terminal UI component organization

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
