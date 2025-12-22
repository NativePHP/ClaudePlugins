---
name: create-nativephp-plugin
description: Scaffold a new NativePHP plugin with complete directory structure, manifest, service provider, facade, bridge functions, JavaScript module, Boost AI guidelines, and tests.
argument-hint: vendor/plugin-name
allowed-tools:
  - Bash
  - Write
  - Read
---

# Create NativePHP Plugin

Create a complete, working NativePHP plugin with the specified name. This matches the output of `php artisan native:plugin:create`.

## Input

The user provides a plugin name in `vendor/plugin-name` format: `$ARGUMENTS`

## Validation

1. Verify the name follows `vendor/plugin-name` format (e.g., `acme/plugin-barcode-scanner`)
2. Extract vendor name and plugin name from the input
3. Convert to appropriate formats:
   - Composer name: `vendor/plugin-name` (lowercase, hyphenated)
   - PHP Namespace: `Vendor\PluginName` (PascalCase, strip "plugin-" prefix)
   - Kotlin Package: `com.vendor.plugin.pluginname` (lowercase, underscores instead of hyphens)
   - Directory: `plugin-name`
   - Namespace: `PluginName` (PascalCase, strip "plugin-" prefix)

If the format is invalid, explain the correct format and ask for a valid name.

## Plugin Location

The default path should be `./packages/{vendor}/{plugin-name}` (e.g., `./packages/acme/plugin-barcode-scanner`).

Ask the user to confirm this path or provide a custom one.

## Files to Create

Create the following complete file structure:

### Directory Structure

```
plugin-name/
├── composer.json
├── nativephp.json
├── README.md
├── .gitignore
├── src/
│   ├── {PluginName}ServiceProvider.php
│   ├── {PluginName}.php
│   ├── Facades/
│   │   └── {PluginName}.php
│   ├── Events/
│   │   └── {PluginName}Completed.php
│   └── Commands/
│       └── CopyAssetsCommand.php
├── resources/
│   ├── android/
│   │   └── {PluginName}Functions.kt
│   ├── ios/
│   │   └── {PluginName}Functions.swift
│   ├── js/
│   │   └── {camelName}.js
│   └── boost/
│       └── guidelines/
│           └── core.blade.php
└── tests/
    ├── Pest.php
    └── PluginTest.php
```

### composer.json

```json
{
    "name": "{vendor}/{plugin-name}",
    "description": "A NativePHP plugin",
    "type": "nativephp-plugin",
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "you@example.com"
        }
    ],
    "require": {
        "php": "^8.2",
        "nativephp/mobile": "^2.0"
    },
    "require-dev": {
        "pestphp/pest": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "{Vendor}\\{PluginName}\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "{Vendor}\\{PluginName}\\{PluginName}ServiceProvider"
            ]
        },
        "nativephp": {
            "manifest": "nativephp.json"
        }
    },
    "scripts": {
        "test": "pest"
    },
    "config": {
        "allow-plugins": {
            "pestphp/pest-plugin": true
        }
    }
}
```

### nativephp.json

```json
{
    "name": "{vendor}/{plugin-name}",
    "version": "1.0.0",
    "description": "A NativePHP plugin",
    "namespace": "{PluginName}",

    "keywords": [],
    "category": "utilities",
    "license": "MIT",
    "pricing": {
        "type": "free"
    },
    "author": {
        "name": "Your Name",
        "email": "you@example.com",
        "url": ""
    },
    "homepage": "",
    "repository": "",
    "funding": [],

    "platforms": ["android", "ios"],

    "icon": "resources/icon.png",
    "screenshots": [],

    "bridge_functions": [
        {
            "name": "{PluginName}.Execute",
            "android": "{kotlinPackage}.{PluginName}Functions.Execute",
            "ios": "{PluginName}Functions.Execute",
            "description": "Execute the plugin functionality"
        },
        {
            "name": "{PluginName}.GetStatus",
            "android": "{kotlinPackage}.{PluginName}Functions.GetStatus",
            "ios": "{PluginName}Functions.GetStatus",
            "description": "Get the current status"
        }
    ],

    "android": {
        "permissions": [],
        "dependencies": {
            "implementation": []
        },
        "activities": [],
        "services": [],
        "receivers": [],
        "providers": [],
        "assets": {}
    },

    "ios": {
        "permissions": {},
        "dependencies": {
            "swift_packages": [],
            "pods": []
        },
        "assets": {}
    },

    "events": [
        "{Vendor}\\{PluginName}\\Events\\{PluginName}Completed"
    ],
    "service_provider": "{Vendor}\\{PluginName}\\{PluginName}ServiceProvider",
    "hooks": {
        "copy_assets": "nativephp:{kebab-plugin-name}:copy-assets"
    }
}
```

### ServiceProvider

```php
<?php

namespace {Vendor}\{PluginName};

use Illuminate\Support\ServiceProvider;
use {Vendor}\{PluginName}\Commands\CopyAssetsCommand;

class {PluginName}ServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton({PluginName}::class, function () {
            return new {PluginName}();
        });
    }

    public function boot(): void
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                CopyAssetsCommand::class,
            ]);
        }
    }
}
```

### Main Implementation Class

```php
<?php

namespace {Vendor}\{PluginName};

class {PluginName}
{
    /**
     * Execute the plugin functionality
     */
    public function execute(array $options = []): mixed
    {
        if (function_exists('nativephp_call')) {
            $result = nativephp_call('{PluginName}.Execute', json_encode($options));

            if ($result) {
                $decoded = json_decode($result);
                return $decoded->data ?? null;
            }
        }

        return null;
    }

    /**
     * Get the current status
     */
    public function getStatus(): ?object
    {
        if (function_exists('nativephp_call')) {
            $result = nativephp_call('{PluginName}.GetStatus', '{}');

            if ($result) {
                $decoded = json_decode($result);
                return $decoded->data ?? null;
            }
        }

        return null;
    }
}
```

### Facade

```php
<?php

namespace {Vendor}\{PluginName}\Facades;

use Illuminate\Support\Facades\Facade;

/**
 * @method static mixed execute(array $options = [])
 * @method static object|null getStatus()
 *
 * @see \{Vendor}\{PluginName}\{PluginName}
 */
class {PluginName} extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return \{Vendor}\{PluginName}\{PluginName}::class;
    }
}
```

### Event Class

```php
<?php

namespace {Vendor}\{PluginName}\Events;

use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class {PluginName}Completed
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public string $result,
        public ?string $id = null
    ) {}
}
```

### CopyAssetsCommand

```php
<?php

namespace {Vendor}\{PluginName}\Commands;

use Native\Mobile\Plugins\Commands\NativePluginHookCommand;

class CopyAssetsCommand extends NativePluginHookCommand
{
    protected $signature = 'nativephp:{kebab-plugin-name}:copy-assets';

    protected $description = 'Copy assets for {PluginName} plugin';

    public function handle(): int
    {
        if ($this->isAndroid()) {
            $this->copyAndroidAssets();
        }

        if ($this->isIos()) {
            $this->copyIosAssets();
        }

        return self::SUCCESS;
    }

    protected function copyAndroidAssets(): void
    {
        // Example: Copy a TensorFlow Lite model to Android assets
        // $this->copyToAndroidAssets('models/model.tflite', 'models/model.tflite');

        $this->info('Android assets copied for {PluginName}');
    }

    protected function copyIosAssets(): void
    {
        // Example: Copy a Core ML model to iOS bundle
        // $this->copyToIosBundle('models/model.mlmodelc', 'models/model.mlmodelc');

        $this->info('iOS assets copied for {PluginName}');
    }
}
```

### Kotlin Bridge Functions

```kotlin
package {kotlinPackage}

import androidx.fragment.app.FragmentActivity
import android.content.Context
import com.nativephp.mobile.bridge.BridgeFunction
import com.nativephp.mobile.bridge.BridgeResponse

object {PluginName}Functions {

    class Execute(private val activity: FragmentActivity) : BridgeFunction {
        override fun execute(parameters: Map<String, Any>): Map<String, Any> {
            // TODO: Implement your native functionality here
            val option1 = parameters["option1"] as? String ?: ""

            return BridgeResponse.success(mapOf(
                "result" to "executed",
                "option1" to option1
            ))
        }
    }

    class GetStatus(private val activity: FragmentActivity) : BridgeFunction {
        override fun execute(parameters: Map<String, Any>): Map<String, Any> {
            // TODO: Return current status
            return BridgeResponse.success(mapOf(
                "status" to "ready",
                "version" to "1.0.0"
            ))
        }
    }
}
```

### Swift Bridge Functions

```swift
import Foundation

enum {PluginName}Functions {

    class Execute: BridgeFunction {
        func execute(parameters: [String: Any]) throws -> [String: Any] {
            // TODO: Implement your native functionality here
            let option1 = parameters["option1"] as? String ?? ""

            return BridgeResponse.success(data: [
                "result": "executed",
                "option1": option1
            ])
        }
    }

    class GetStatus: BridgeFunction {
        func execute(parameters: [String: Any]) throws -> [String: Any] {
            // TODO: Return current status
            return BridgeResponse.success(data: [
                "status": "ready",
                "version": "1.0.0"
            ])
        }
    }
}
```

### JavaScript Module

```javascript
/**
 * {PluginName} Plugin for NativePHP Mobile
 *
 * @example
 * import { {camelName} } from '@{vendor}/{plugin-name}';
 *
 * // Execute functionality
 * const result = await {camelName}.execute({ option1: 'value' });
 *
 * // Get status
 * const status = await {camelName}.getStatus();
 */

const baseUrl = '/_native/api/call';

async function bridgeCall(method, params = {}) {
    const response = await fetch(baseUrl, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]')?.content || ''
        },
        body: JSON.stringify({ method, params })
    });

    const result = await response.json();

    if (result.status === 'error') {
        throw new Error(result.message || 'Native call failed');
    }

    const nativeResponse = result.data;
    if (nativeResponse && nativeResponse.data !== undefined) {
        return nativeResponse.data;
    }

    return nativeResponse;
}

export async function execute(options = {}) {
    return bridgeCall('{PluginName}.Execute', options);
}

export async function getStatus() {
    return bridgeCall('{PluginName}.GetStatus');
}

export const {camelName} = {
    execute,
    getStatus
};

export default {camelName};
```

### Boost AI Guidelines (resources/boost/guidelines/core.blade.php)

```blade
## {vendor}/{plugin-name}

A NativePHP plugin.

### Installation

```bash
composer require {vendor}/{plugin-name}
```

### PHP Usage (Livewire/Blade)

Use the `{PluginName}` facade:

@verbatim
<code-snippet name="Using {PluginName} Facade" lang="php">
use {Vendor}\{PluginName}\Facades\{PluginName};

// Execute the plugin functionality
$result = {PluginName}::execute(['option1' => 'value']);

// Get the current status
$status = {PluginName}::getStatus();
</code-snippet>
@endverbatim

### Available Methods

- `{PluginName}::execute()`: Execute the plugin functionality
- `{PluginName}::getStatus()`: Get the current status

### Events

- `{PluginName}Completed`: Listen with `#[OnNative({PluginName}Completed::class)]`

@verbatim
<code-snippet name="Listening for {PluginName} Events" lang="php">
use Native\Mobile\Attributes\OnNative;
use {Vendor}\{PluginName}\Events\{PluginName}Completed;

#[OnNative({PluginName}Completed::class)]
public function handle{PluginName}Completed($result, $id = null)
{
    // Handle the event
}
</code-snippet>
@endverbatim

### JavaScript Usage (Vue/React/Inertia)

@verbatim
<code-snippet name="Using {PluginName} in JavaScript" lang="javascript">
import { {camelName} } from '@{vendor}/{plugin-name}';

// Execute the plugin functionality
const result = await {camelName}.execute({ option1: 'value' });

// Get the current status
const status = await {camelName}.getStatus();
</code-snippet>
@endverbatim
```

### tests/Pest.php

```php
<?php

uses()->in('.');
```

### tests/PluginTest.php

```php
<?php

beforeEach(function () {
    $this->pluginPath = dirname(__DIR__);
    $this->manifestPath = $this->pluginPath . '/nativephp.json';
});

describe('Plugin Manifest', function () {
    it('has a valid nativephp.json file', function () {
        expect(file_exists($this->manifestPath))->toBeTrue();

        $content = file_get_contents($this->manifestPath);
        $manifest = json_decode($content, true);

        expect(json_last_error())->toBe(JSON_ERROR_NONE);
    });

    it('has required fields', function () {
        $manifest = json_decode(file_get_contents($this->manifestPath), true);

        expect($manifest)->toHaveKeys(['name', 'namespace', 'bridge_functions']);
    });

    it('has valid bridge functions', function () {
        $manifest = json_decode(file_get_contents($this->manifestPath), true);

        expect($manifest['bridge_functions'])->toBeArray();

        foreach ($manifest['bridge_functions'] as $function) {
            expect($function)->toHaveKeys(['name']);
            expect($function)->toHaveAnyKeys(['android', 'ios']);
        }
    });
});

describe('Native Code', function () {
    it('has Android Kotlin file', function () {
        $kotlinFile = $this->pluginPath . '/resources/android/{PluginName}Functions.kt';
        expect(file_exists($kotlinFile))->toBeTrue();
    });

    it('has iOS Swift file', function () {
        $swiftFile = $this->pluginPath . '/resources/ios/{PluginName}Functions.swift';
        expect(file_exists($swiftFile))->toBeTrue();
    });
});

describe('PHP Classes', function () {
    it('has service provider', function () {
        $file = $this->pluginPath . '/src/{PluginName}ServiceProvider.php';
        expect(file_exists($file))->toBeTrue();
    });

    it('has facade', function () {
        $file = $this->pluginPath . '/src/Facades/{PluginName}.php';
        expect(file_exists($file))->toBeTrue();
    });

    it('has main implementation class', function () {
        $file = $this->pluginPath . '/src/{PluginName}.php';
        expect(file_exists($file))->toBeTrue();
    });
});

describe('Composer Configuration', function () {
    it('has valid composer.json', function () {
        $composerPath = $this->pluginPath . '/composer.json';
        expect(file_exists($composerPath))->toBeTrue();

        $content = file_get_contents($composerPath);
        $composer = json_decode($content, true);

        expect(json_last_error())->toBe(JSON_ERROR_NONE);
        expect($composer['type'])->toBe('nativephp-plugin');
    });
});
```

### README.md

```markdown
# {PluginName} Plugin for NativePHP Mobile

A NativePHP plugin.

## Installation

```bash
composer require {vendor}/{plugin-name}
```

## Usage

```php
use {Vendor}\{PluginName}\Facades\{PluginName};

// Execute functionality
$result = {PluginName}::execute(['option1' => 'value']);

// Get status
$status = {PluginName}::getStatus();
```

## Listening for Events

```php
use Livewire\Attributes\On;

#[On('native:{Vendor}\{PluginName}\Events\{PluginName}Completed')]
public function handle{PluginName}Completed($result, $id = null)
{
    // Handle the event
}
```

## License

MIT
```

### .gitignore

```
/vendor/
/node_modules/
.DS_Store
*.log
```

## After Creation

1. Display the created file structure
2. Show next steps:
   - Implement native functions in `resources/android/` and `resources/ios/`
   - Edit `nativephp.json` to add permissions and dependencies
   - Customize the copy_assets hook in `src/Commands/CopyAssetsCommand.php`
3. Show how to install:
   - Add to composer.json repositories: `{"type": "path", "url": "./packages/{vendor}/{plugin-name}"}`
   - Run `composer require {vendor}/{plugin-name}`
   - Verify with `php artisan native:plugin:list`

## Important Notes

- The plugin MUST be created in `./packages/{vendor}/{plugin-name}` format (e.g., `./packages/nativephp/plugin-ble`)
- Do NOT create in `./packages/{plugin-name}` - this is incorrect
- The vendor directory (e.g., `nativephp/`, `acme/`) must exist as a parent folder