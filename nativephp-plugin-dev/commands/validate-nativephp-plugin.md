---
name: validate-nativephp-plugin
description: Validate the current NativePHP plugin structure, manifest, and code patterns. Checks for common issues and ensures the plugin follows NativePHP conventions.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# Validate NativePHP Plugin

Validate the NativePHP plugin in the current directory or specified path.

## Validation Process

Perform a comprehensive validation of the plugin, checking each area and reporting issues.

### 1. Find the Plugin

First, locate the plugin to validate:

1. Look for `nativephp.json` in the current directory
2. If not found, check `./packages/*/nativephp.json`
3. If multiple found, ask user which one to validate
4. If none found, report error and suggest running `/create-nativephp-plugin`

### 2. Manifest Validation (nativephp.json)

Read and validate the manifest:

**Required fields:**
- [ ] `namespace` - Must be PascalCase
- [ ] `bridge_functions` - Must be array (can be empty)

**Note**: Package metadata (`name`, `version`, `description`) comes from `composer.json`, not the manifest.

**Bridge function validation (for each function):**
- [ ] `name` - Must be `Namespace.Action` format
- [ ] `android` - Must be full class path (e.g., `com.vendor.plugins.myplugin.MyPluginFunctions.Execute`)
- [ ] `ios` - Must be `EnumName.ClassName` format (e.g., `MyPluginFunctions.Execute`)
- [ ] `description` - Should exist for documentation

**Platform sections validation:**
- [ ] `android.permissions` - If present, must be array of valid permission strings
- [ ] `android.dependencies.implementation` - If present, must be array of gradle dependencies
- [ ] `android.repositories` - If present, must be array of repository objects with `url`
- [ ] `ios.info_plist` - If present, must be object with valid Info.plist keys
- [ ] `ios.dependencies.swift_packages` - If present, must be array of objects with `url` and `version`
- [ ] `ios.dependencies.pods` - If present, must be array of strings
- [ ] `events` - If present, must be array of fully-qualified class names
- [ ] `secrets` - If present, must be object with description and required flag

### 3. Composer Validation (composer.json)

Read and validate:

- [ ] `type` - Must be `"nativephp-plugin"`
- [ ] `name` - Must be `vendor/plugin-name` format
- [ ] `require.nativephp/mobile` - Should be present
- [ ] `autoload.psr-4` - Must define namespace matching manifest
- [ ] `extra.laravel.providers` - Should register the service provider
- [ ] `extra.laravel.aliases` - Should register the facade (optional)

### 4. PHP Code Validation

**Service Provider:**
- [ ] File exists at path matching provider in `composer.json` extra.laravel.providers
- [ ] Extends `ServiceProvider`
- [ ] Has `register()` method
- [ ] Registers the main class as singleton

**Main API Class:**
- [ ] File exists
- [ ] Has methods matching bridge functions
- [ ] Uses `nativephp_call()` with correct method names and JSON-encoded params
- [ ] Wraps calls in `if (function_exists('nativephp_call'))`

**Facade:**
- [ ] File exists in `Facades/` directory
- [ ] Extends `Facade`
- [ ] `getFacadeAccessor()` returns correct class

**Events:**
- [ ] Each event in manifest has corresponding PHP class
- [ ] Event classes use `Dispatchable` trait
- [ ] Event classes do NOT implement `ShouldBroadcast`

### 5. Native Code Validation

**Kotlin (Android):**
- [ ] File exists at `resources/android/{Namespace}Functions.kt` (or `resources/android/src/` for legacy structure)
- [ ] Package follows `com.{vendor}.plugins.{pluginname}` format
- [ ] Object name matches `{Namespace}Functions`
- [ ] Each bridge function has matching class
- [ ] Classes implement `BridgeFunction` from `com.nativephp.mobile.bridge`
- [ ] `execute()` method returns `Map<String, Any>`
- [ ] Uses `BridgeResponse.success()` for success
- [ ] Uses `BridgeResponse.error(BridgeError("CODE", "message"))` for errors (BridgeError required!)

**Swift (iOS):**
- [ ] File exists at `resources/ios/{Namespace}Functions.swift` (or `resources/ios/Sources/` for legacy structure)
- [ ] Enum name matches `{Namespace}Functions`
- [ ] Each bridge function has matching class
- [ ] Classes conform to `BridgeFunction`
- [ ] `execute()` method returns `[String: Any]`
- [ ] Uses `BridgeResponse.success(data:)` for success
- [ ] Uses `BridgeResponse.error(code: "CODE", message: "message")` for errors (both params required!)

### 6. Consistency Checks

**Bridge function alignment:**
- [ ] Every function in manifest has Kotlin implementation
- [ ] Every function in manifest has Swift implementation
- [ ] Method names in PHP match manifest names
- [ ] Event class names in native code match manifest

**Naming consistency:**
- [ ] Namespace is consistent across all files
- [ ] Vendor name is consistent
- [ ] Plugin name is consistent

### 7. Best Practice Checks

**Security:**
- [ ] No hardcoded API keys or secrets
- [ ] Permissions are minimal and justified

**Code quality:**
- [ ] All bridge functions validate required parameters
- [ ] Error responses include meaningful messages
- [ ] Async operations dispatch events on main thread

**Documentation:**
- [ ] README.md exists
- [ ] Bridge functions have description in manifest
- [ ] Code has documentation comments

## Report Format

Generate a validation report:

```
## NativePHP Plugin Validation Report

**Plugin:** vendor/plugin-name
**Version:** 1.0.0
**Path:** ./packages/plugin-name

### Summary
- ✅ Passed: X checks
- ⚠️ Warnings: X items
- ❌ Errors: X issues

### Manifest (nativephp.json)
✅ Valid JSON structure
✅ Required fields present
⚠️ Missing description for bridge function: MyPlugin.GetStatus

### Composer (composer.json)
✅ Type is nativephp-plugin
✅ Namespace matches manifest
❌ Missing nativephp/mobile in require

### PHP Code
✅ Service provider exists and is valid
✅ Main class implements all bridge functions
✅ Facade is properly configured
⚠️ Event ActionCompleted missing docblock

### Kotlin Code
✅ Functions file exists
✅ All bridge functions implemented
✅ Proper error handling

### Swift Code
✅ Functions file exists
✅ All bridge functions implemented
⚠️ Missing documentation comments

### Consistency
✅ Bridge functions aligned across platforms
✅ Naming is consistent

### Recommendations
1. Add description to MyPlugin.GetStatus bridge function
2. Add nativephp/mobile to composer.json require
3. Add documentation comments to Swift code
```

## After Validation

1. If errors found: Offer to help fix them
2. If only warnings: Explain which are important vs. optional
3. If all passed: Confirm plugin is ready for use