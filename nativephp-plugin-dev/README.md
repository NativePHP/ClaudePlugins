# NativePHP Plugin Developer Assistant

A Claude Code plugin that helps developers create NativePHP mobile plugins.

## What This Plugin Does

When you're building a NativePHP plugin (a composer package that adds native iOS/Android functionality to NativePHP apps), this Claude Code plugin provides:

- **Knowledge Skills**: Understanding of NativePHP architecture, plugin structure, and native code patterns
- **Scaffolding Commands**: Generate new plugin boilerplate with working examples
- **Expert Agents**: Specialized assistance for Kotlin, Swift, JavaScript bridge code
- **Validation**: Check your plugin follows NativePHP patterns correctly

## Installation

```bash
# Option 1: Use directly
claude --plugin-dir /path/to/nativephp-plugin-dev

# Option 2: Add to your project's .claude/plugins
cp -r nativephp-plugin-dev /your-project/.claude/plugins/
```

## Usage

### Ask Questions (Skills auto-activate)

```
"How does the NativePHP bridge work?"
"What's the structure of a nativephp.json manifest?"
"Show me the Kotlin bridge function pattern"
```

### Create a New Plugin

```
/create-nativephp-plugin vendor/my-awesome-plugin
```

### Validate Your Plugin

```
/validate-nativephp-plugin
```

### Get Expert Help

The plugin includes specialized agents that activate when you need:

- **Kotlin/Android code**: Bridge functions, Activities, sensors, ML
- **Swift/iOS code**: Bridge functions, ViewControllers, AVFoundation, Core ML
- **JavaScript bridge**: TypeScript interfaces, client-side code
- **Plugin orchestration**: Full plugin creation workflow

## Components

### Skills

| Skill | Purpose |
|-------|---------|
| `nativephp-architecture` | Core concepts: god method, bridge registration, EDGE, events |
| `plugin-structure` | Plugin layout, manifest schema, service providers |
| `native-code-patterns` | Kotlin/Swift bridge function templates |

### Commands

| Command | Purpose |
|---------|---------|
| `/create-nativephp-plugin` | Scaffold a new plugin with working example |
| `/validate-nativephp-plugin` | Check plugin structure and patterns |

### Agents

| Agent | Purpose |
|-------|---------|
| `nativephp-plugin-writer` | Main orchestrator for plugin creation |
| `kotlin-android-expert` | Android/Kotlin native code |
| `swift-ios-expert` | iOS/Swift native code |
| `js-bridge-expert` | JavaScript/TypeScript bridge |
| `nativephp-plugin-validator` | Validation and quality checks |

## Requirements

- Claude Code CLI
- Basic familiarity with Laravel, Kotlin, and Swift

## License

MIT