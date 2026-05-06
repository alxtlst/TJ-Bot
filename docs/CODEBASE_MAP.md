# Codebase map

TJ-Bot is a Java 25 Discord bot for the Together Java community. It uses JDA for Discord, SQLite with Flyway migrations and jOOQ for persistence, and Gradle multi-project builds. The runnable artifact is the **`application`** module (fat JAR via Shadow, optional container image via Jib).

## Repository structure

| Path | Role |
|------|------|
| `application/` | Discord bot implementation: config, features, logging integration, resources (including SQL migrations on classpath). |
| `database/` | JDBC/Flyway/jOOQ **`Database`** façade and small utilities; consumed by `application`. |
| `formatter/` | Standalone Java code formatter (lexer/rules) used by bot features that format code snippets. |
| `utils/` | Shared minimal utilities (e.g. nullability annotations package). |
| `buildSrc/` | Gradle convention plugins; **`database-settings`** drives Flyway + jOOQ codegen for the app DB schema. |
| `gradle/` | Gradle wrapper and version catalog support files. |
| `meta/` | Spotless Eclipse formatter config (`google-style-eclipse.xml`). |
| `scripts/` | `pre-commit` hook copied into `.git/hooks` by the root `installLocalGitHook` task. |
| `wiki/` | Markdown wiki pages (project docs; sync may be automated via CI). |
| `.github/` | GitHub workflows (checks, Docker verify, analysis, releases, wiki sync, etc.), templates, Dependabot. |
| `.devcontainer/` | Dev container definition for local/editor setup. |
| `.woodpecker.yml` | Woodpecker CI pipeline (status badge referenced from README). |
| `docker-compose.yml.example` | Example compose layout for running with Docker. |
| Root `build.gradle` | Root Java toolchain (25), Spotless, SonarQube/SonarLint, shared subproject configuration. |
| `settings.gradle` | Includes `application`, `database`, `formatter`, `utils`. |
| `README.md`, `CONTRIBUTING.md` | Overview and pointer to GitHub wiki for setup and contribution. |
| Legal / policy | `LICENSE`, `CLA.md`, `PP.md`, `TOS.md`. |

## Modules / major areas

### application

- **Purpose**: Entry point, Discord wiring, all bot behavior (`features`), Jackson-backed **`Config`**, Discord log forwarding, packaged Flyway migrations.
- **Location**: `application/`
- **Notes**: Applies **`database-settings`** plugin (`application/build.gradle`) so builds run Flyway against a temporary SQLite file and generate **jOOQ** classes into package `org.togetherjava.tjbot.db.generated` for compile-time typed SQL. Runtime migrations load from classpath **`classpath:/db/`**, matching **`application/src/main/resources/db/*.sql`**. Uses Shadow (`TJ-Bot` JAR) and Jib (`org.togetherjava.tjbot.Application`). Major dependency surfaces: JDA, OpenAI Java client, GitHub API, jOOQ, Jackson, Log4j.

### database

- **Purpose**: **`org.togetherjava.tjbot.db.Database`** — SQLite datasource, WAL, Flyway migrate on startup, thread-safe jOOQ **`DSLContext`** accessors.
- **Location**: `database/src/main/java/org/togetherjava/tjbot/db/`
- **Notes**: Depends on **`utils`**. Does not ship SQL scripts itself; migrations come from the **`application`** resource set on the runtime classpath.

### formatter

- **Purpose**: Format Java-like source for display (tokenizer, formatting rules); library module with tests.
- **Location**: `formatter/src/main/java/org/togetherjava/tjbot/formatter/`
- **Notes**: Depends on **`utils`**.

### utils

- **Purpose**: Shared **`annotations`** package (e.g. default nullability for APIs).
- **Location**: `utils/src/main/java/org/togetherjava/tjbot/annotations/`

### buildSrc

- **Purpose**: **`database-settings`** Gradle plugin (Flyway + jOOQ generation wiring for schema evolution tied to SQL under `application`).
- **Location**: `buildSrc/src/main/groovy/database-settings.gradle`

## Key entry points

| Kind | Path | Role |
|------|------|------|
| JVM main | `application/src/main/java/org/togetherjava/tjbot/Application.java` | Loads **`config.json`** (or path from args), starts **`DiscordLogging`**, opens SQLite **`Database`**, builds **JDA**, constructs **`BotCore`**, **`CommandReloading.reloadCommands`**, schedules routines, registers **`BotCore`** as listener. |
| Feature registry | `application/src/main/java/org/togetherjava/tjbot/features/Features.java` | **`createFeatures`** builds all **`Feature`** instances (commands, listeners, routines); primary place to register new behavior; respects **`FeatureBlacklistConfig`**. |
| Event/command hub | `application/src/main/java/org/togetherjava/tjbot/features/system/BotCore.java` | **`ListenerAdapter`** implementing **`CommandProvider`**: registers slash/user/message commands with Discord, dispatches interactions and messages to **`SlashCommand`**, **`Routine`**, **`MessageReceiver`**, component-ID handlers, etc. |
| Slash command reload | `application/src/main/java/org/togetherjava/tjbot/CommandReloading.java` | Pushes command definitions to Discord (global vs guild visibility). |
| Configuration | `application/src/main/java/org/togetherjava/tjbot/config/Config.java` | **`Config.load(Path)`** — JSON configuration model used across features. |
| Persistence bootstrap | `database/src/main/java/org/togetherjava/tjbot/db/Database.java` | Flyway + jOOQ lifecycle for the bot’s SQLite file. |

### Feature layout (high level)

Behavior lives under **`application/.../features/`**, grouped by domain (each may mix commands, listeners, routines):

`analytics`, `basic`, `bookmarks`, `chatgpt`, `code`, `componentids`, `filesharing`, `github`, `help`, `jshell`, `mathcommands` (incl. Wolfram Alpha), `mediaonly`, `messages`, `moderation` (incl. audit, modmail, scam, temp actions), `projects`, `reminder`, `rss`, `system`, `tags`, `tophelper`, `utils`, `voicechat`.

Supporting packages at the **`tjbot`** root (alongside **`features`**): **`config`**, **`logging/discord`**.

## Configuration & environment

- **Primary config**: JSON file, default path **`config.json`** next to the working directory; template at **`application/config.json.template`** (tokens, GitHub/OpenAI keys, DB path, channel name patterns, role patterns, feature toggles, webhooks, RSS, JShell, etc.).
- **Database file**: Path from config (`databasePath`); parent directories created on startup if needed.
- **Run / build**: Standard Gradle **`application`** plugin (`mainClass` **`org.togetherjava.tjbot.Application`**); JVM args include **`--enable-native-access=ALL-UNNAMED`**. Tests use JUnit 5 platform (configured on subprojects in root **`build.gradle`**).
- **Quality/format**: Spotless (Java, Google-style Eclipse config under **`meta/`**); SonarQube/SonarLint plugins on root and subprojects.

## Relationships & data flow

**Module dependency graph (compile-time)**

```text
application ──► database ──► utils
     │              │
     ├──────────────┘
     └──► formatter ──► utils
```

**Startup**

1. **`Application.main`** → **`Config.load`** → **`DiscordLogging.startDiscordLogging`**.
2. **`Database`** constructor → Flyway **`classpath:/db/`** → jOOQ **`DSLContext`**.
3. **JDA** connects with token and intents (`GUILD_MEMBERS`, `MESSAGE_CONTENT`).
4. **`BotCore(jda, database, config, metrics)`** pulls features from **`Features.createFeatures`**.
5. **`CommandReloading.reloadCommands`** updates Discord application commands.
6. **`core.scheduleRoutines`** + **`jda.addEventListener(core)`** attach periodic work and Discord events.

**Request path (conceptual)**

Discord gateway events → **`BotCore`** → typed **`Feature`** implementations (slash/context commands, **`Routine`** timers, **`MessageReceiver`** / **`EventReceiver`** paths) → **`Database`**, **`Config`**, external APIs (OpenAI, GitHub, Wolfram, webhooks) as needed.

**Schema evolution**

- Author SQL under **`application/src/main/resources/db/`**.
- Generated access layer: **`org.togetherjava.tjbot.db.generated`** (from **`application`** build + **`database-settings`**).

## Related documentation

- **`README.md`** — badges, community links, wiki pointer, JitPack coordinates.
- **`CONTRIBUTING.md`** — short intro; detailed contribution flow on **[GitHub wiki](https://github.com/Together-Java/TJ-Bot/wiki)**.
- **`wiki/`** — repository-held wiki markdown (e.g. releases, component IDs, log viewer setup).

## Map metadata

- Generated from repo inspection; git **`0506b1a`**, commit date **`2026-05-01`** (authoritative for “current tree” only until the next regenerate).
