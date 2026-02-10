# DM Claude - Technical Breakdown and Reconstruction Guide

This document provides a comprehensive technical breakdown of the DM Claude codebase. It is designed to allow a developer to understand the architecture, dependencies, and file structure, enabling them to recreate the system from scratch if necessary.

## 1. System Architecture

DM Claude is a hybrid system combining:
1.  **Python Core**: The business logic (D&D rules, state management, RAG) resides in Python modules.
2.  **Bash CLI**: Shell scripts act as the primary interface, wrapping Python calls and handling environment setup.
3.  **JSON Persistence**: All game state is stored in human-readable JSON files within a `world-state` directory.
4.  **Claude Code Integration**: The system is designed to be used by an AI agent (Claude Code) which executes the CLI tools based on user intent.

### Data Flow
`User Input` -> `Claude Code Agent` -> `Bash Tool (tools/dm-*.sh)` -> `Python Script (lib/*.py)` -> `JSON Data (world-state/campaigns/*)`

---

## 2. Prerequisites

To recreate this environment, the following are required:
-   **OS**: Linux or macOS (Bash shell required).
-   **Python**: Version 3.11 or higher.
-   **Package Manager**: `uv` (recommended for speed and lockfile management) or `pip`.
-   **JSON Processor**: `jq` (used by some shell scripts for quick parsing).
-   **Node.js**: Required to run `@anthropic-ai/claude-code`.

---

## 3. Directory Structure

The project follows a specific directory layout which must be preserved for the tools to function correctly.

```
.
├── .env                  # Configuration (API keys, defaults)
├── pyproject.toml        # Python dependencies and project metadata
├── install.sh            # Setup script
├── CLAUDE.md             # Operational guide for the AI agent
├── README.md             # High-level documentation
├── tools/                # Bash CLI wrappers
│   ├── common.sh         # Shared environment logic
│   └── dm-*.sh           # Individual tools (e.g., dm-player.sh)
├── lib/                  # Core Python libraries
│   ├── world.py          # Central entry point
│   ├── dice.py           # RNG logic
│   └── *_manager.py      # Entity managers (NPC, Location, etc.)
├── features/             # Feature-specific modules
│   ├── dnd-api/          # Integration with D&D 5e API
│   ├── character-creation/ # Character generation logic
│   └── ...               # Other features (loot, spells, etc.)
└── world-state/          # Data storage (git-ignored content)
    ├── active-campaign.txt # Name of currently loaded campaign
    └── campaigns/        # One folder per campaign
        └── [Campaign Name]/
            ├── campaign-overview.json
            ├── npcs.json
            ├── locations.json
            ├── ...
            └── saves/    # Backup snapshots
```

---

## 4. Key Components

### 4.1. Configuration (`pyproject.toml`)
The project uses `pyproject.toml` for dependency management.
**Core Dependencies:**
-   `anthropic`: For AI integration.
-   `python-dotenv`: To load `.env`.
-   `pdfplumber`, `pypdf2`, `python-docx`: For reading source material (RAG).
-   `requests`: For fetching data from the D&D 5e API.
-   `sentence-transformers`, `chromadb`: For the RAG system (vector search).

### 4.2. Core Library (`lib/`)
This is the heart of the application.
-   **`world.py`**: The `World` class acts as a facade, providing access to all other managers (`npcs`, `locations`, `player`, etc.). It initializes the `CampaignManager` to find the active campaign directory.
-   **`json_ops.py`**: Handles safe reading and writing of JSON files. It ensures data integrity.
-   **`dice.py`**: specific module for parsing and rolling dice notation (e.g., "1d20+5").
-   **`*_manager.py`**: Each entity type (NPC, Location, Plot, Player) has a dedicated manager class in `lib/` that encapsulates CRUD operations for its specific JSON file.

### 4.3. Feature Modules (`features/`)
Self-contained modules for specific gameplay features.
-   **`dnd-api`**: Wraps calls to `https://www.dnd5eapi.co/` to retrieve official rules, monsters, and spells.
-   **`character-creation`**: Logic for generating D&D 5e characters (stats, classes, races).
-   **`rules`**: Local implementation of specific rule checks.

### 4.4. CLI Tools (`tools/`)
These scripts provide the command-line interface.
-   **`common.sh`**: Sourced by all other scripts. It:
    -   Detects the Python interpreter (preferring `uv`).
    -   Sets up paths (`PROJECT_ROOT`, `LIB_DIR`).
    -   Determines the active campaign by reading `world-state/active-campaign.txt`.
-   **`dm-*.sh`**: Each script (e.g., `dm-player.sh`) typically parses arguments and then calls a specific Python script or module method using `uv run python`.

---

## 5. Data Persistence Model

Data is stored in `world-state/campaigns/[CampaignName]/`. Key files include:

-   **`campaign-overview.json`**:
    ```json
    {
      "name": "Campaign Name",
      "current_date": "Date string",
      "time_of_day": "Morning",
      "player_position": { "current_location": "Location Name" }
    }
    ```
-   **`npcs.json`**: Dictionary of NPC objects keyed by name.
    ```json
    {
      "Name": { "description": "...", "attitude": "neutral", "stats": {...} }
    }
    ```
-   **`locations.json`**: Dictionary of Location objects.
-   **`character.json`**: The player's character sheet.
-   **`session-log.md`**: A Markdown file appending the narrative history of the session.

---

## 6. Step-by-Step Reconstruction

To rebuild this system from scratch:

1.  **Initialize Project**:
    -   Create the root directory.
    -   Run `git init`.
    -   Create `world-state/campaigns` directory.
    -   Create `tools`, `lib`, `features` directories.

2.  **Setup Environment**:
    -   Create `pyproject.toml` with the dependencies listed above.
    -   Run `uv sync` to install dependencies and create the virtual environment.
    -   Create `.env` file for configuration.

3.  **Implement Data Layer (`lib/`)**:
    -   Write `json_ops.py` to handle JSON I/O.
    -   Write `campaign_manager.py` to manage `active-campaign.txt` and campaign folders.
    -   Write `world.py` to coordinate the managers.

4.  **Implement Feature Logic (`features/`)**:
    -   Implement `dnd-api` connectors using `requests`.
    -   Implement character creation logic.

5.  **Create Bash Wrappers (`tools/`)**:
    -   Write `common.sh` to handle pathing and python execution.
    -   Create `dm-player.sh`, `dm-npc.sh`, etc., that call the corresponding python functions.
    -   Ensure scripts are executable (`chmod +x tools/*.sh`).

6.  **Documentation**:
    -   Create `CLAUDE.md` to instruct the AI agent on how to use the tools. This is critical for the "AI Dungeon Master" experience.

7.  **Testing**:
    -   Run `bash tools/dm-campaign.sh new "Test Campaign"` to verify directory creation.
    -   Run `bash tools/dm-player.sh` to verify python integration.

---

## 7. Operational Notes

-   **State Management**: The system relies on the file system for state. There is no running database server; `world.py` reloads state from JSON on every invocation.
-   **AI Context**: The `CLAUDE.md` file is essential. It tells the AI model how to interpret the tool outputs and how to format its responses (narrative style, HP bars, etc.).
