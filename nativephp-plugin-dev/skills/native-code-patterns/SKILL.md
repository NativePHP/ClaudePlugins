---
name: Native Code Patterns
description: This skill provides Kotlin and Swift code patterns for NativePHP plugins. Use when the user asks about "bridge function pattern", "kotlin bridge", "swift bridge", "BridgeFunction class", "BridgeResponse", "execute method", "parameter extraction", "dispatch event", "NativeActionCoordinator", "LaravelBridge", "Activity pattern", "ViewController pattern", threading in native code, or how to write native code for NativePHP plugins.
version: 1.0.0
---

# Native Code Patterns for NativePHP Plugins

This skill provides complete, production-ready patterns for Kotlin (Android) and Swift (iOS) bridge functions in NativePHP plugins.

---

## CRITICAL: BridgeResponse Helper

**BridgeResponse is a REAL helper object that EXISTS in NativePHP and MUST be used.**

Do NOT write code that returns plain `Map<String, Any>` or `[String: Any]` directly. Always use:

### Kotlin
```kotlin
import com.nativephp.mobile.bridge.BridgeResponse
import com.nativephp.mobile.bridge.BridgeError

// Success
return BridgeResponse.success(mapOf("key" to "value"))

// Error (always use BridgeError with code and message)
return BridgeResponse.error(BridgeError("ERROR_CODE", "Error message"))
```

**IMPORTANT: Android BridgeResponse.error requires a `BridgeError` object with both `code` and `message` parameters.**

### Swift
```swift
// Success
return BridgeResponse.success(data: ["key": "value"])

// Error (always include code and message)
return BridgeResponse.error(code: "ERROR_CODE", message: "Error message")
```

**IMPORTANT: iOS BridgeResponse.error ALWAYS requires both `code` and `message` parameters.**

**BridgeResponse is defined in:**
- Android: `com.nativephp.mobile.bridge.BridgeResponse` (BridgeResponse object)
- iOS: Built into the NativePHP bridge

**The official NativePHP plugin stubs use BridgeResponse. Follow this pattern.**

---

## Kotlin Bridge Function Pattern

### Basic Template

```kotlin
package com.myvendor.plugins.myplugin

import androidx.fragment.app.FragmentActivity
import android.content.Context
import com.nativephp.mobile.bridge.BridgeFunction
import com.nativephp.mobile.bridge.BridgeResponse
import com.nativephp.mobile.bridge.BridgeError

object MyPluginFunctions {

    /**
     * Brief description of what this function does.
     *
     * Parameters:
     * - paramName: Type - Description
     *
     * Returns:
     * - resultKey: Type - Description
     */
    class Execute(private val activity: FragmentActivity) : BridgeFunction {
        override fun execute(parameters: Map<String, Any>): Map<String, Any> {
            // 1. Extract and validate parameters
            val param1 = parameters["param1"] as? String
                ?: return BridgeResponse.error(BridgeError("INVALID_PARAMETERS", "param1 is required"))

            // 2. Perform the native operation
            try {
                val result = performOperation(param1)

                // 3. Return success response
                return BridgeResponse.success(mapOf(
                    "result" to result
                ))
            } catch (e: Exception) {
                return BridgeResponse.error(BridgeError("OPERATION_FAILED", e.message ?: "Unknown error"))
            }
        }

        private fun performOperation(param: String): String {
            // Implementation
            return "processed: $param"
        }
    }

    // All bridge functions use FragmentActivity - get context from it if needed
    class GetStatus(private val activity: FragmentActivity) : BridgeFunction {
        override fun execute(parameters: Map<String, Any>): Map<String, Any> {
            // If you need context: val context: Context = activity
            return BridgeResponse.success(mapOf(
                "status" to "ready",
                "version" to "1.0.0"
            ))
        }
    }
}
```

### Package Naming

Use your own vendor-namespaced package for your plugin code:

```kotlin
package com.myvendor.plugins.myplugin
```

Where `myvendor` is your vendor name and `myplugin` is your plugin name (lowercase, no hyphens - use underscores if needed). The `plugins` segment groups all your plugins together. This keeps your plugin code isolated from other plugins.

### Constructor Parameters - ALWAYS Use FragmentActivity

**Use `FragmentActivity` for ALL bridge functions.** The NativePHP bridge registration passes `activity` to all functions. You can always get `Context` from `activity`:

```kotlin
// CORRECT - works with the bridge registration
class MyFunction(private val activity: FragmentActivity) : BridgeFunction {
    override fun execute(parameters: Map<String, Any>): Map<String, Any> {
        // Get context when needed
        val context: Context = activity
        val prefs = context.getSharedPreferences("my_prefs", Context.MODE_PRIVATE)
        // ...
    }
}
```

**Do NOT use `Context` as constructor parameter** - the bridge registers functions with `activity`, not `context`.

### Parameter Extraction Patterns

```kotlin
// Required string
val name = parameters["name"] as? String
    ?: return BridgeResponse.error(BridgeError("INVALID_PARAMETERS", "name is required"))

// Required number (always comes as Number)
val count = (parameters["count"] as? Number)?.toInt()
    ?: return BridgeResponse.error(BridgeError("INVALID_PARAMETERS", "count is required"))

// Optional with default
val quality = (parameters["quality"] as? Number)?.toInt() ?: 80
val enabled = parameters["enabled"] as? Boolean ?: false

// Double/Float
val amount = (parameters["amount"] as? Number)?.toDouble() ?: 0.0

// Arrays/Lists
val items = (parameters["items"] as? List<*>)?.filterIsInstance<String>() ?: emptyList()
val numbers = (parameters["numbers"] as? List<*>)?.mapNotNull { (it as? Number)?.toInt() } ?: emptyList()

// Nested objects
val config = parameters["config"] as? Map<*, *>
val configValue = config?.get("key") as? String ?: "default"
```

### Response Patterns

```kotlin
// Success with data
return BridgeResponse.success(mapOf(
    "path" to filePath,
    "size" to fileSize,
    "metadata" to mapOf(
        "width" to width,
        "height" to height
    )
))

// Success with array
return BridgeResponse.success(mapOf(
    "items" to listOf(
        mapOf("id" to 1, "name" to "Item 1"),
        mapOf("id" to 2, "name" to "Item 2")
    )
))

// Error (always use BridgeError with code and message)
return BridgeResponse.error(BridgeError("OPERATION_FAILED", "Something went wrong"))

// Error with specific code
return BridgeResponse.error(BridgeError("FILE_NOT_FOUND", "The specified file does not exist"))
```

### Event Dispatching (Kotlin)

Events notify PHP when async operations complete:

```kotlin
import android.os.Handler
import android.os.Looper
import org.json.JSONObject
import com.nativephp.mobile.NativeActionCoordinator

// MUST dispatch on main thread for JavaScript execution
Handler(Looper.getMainLooper()).post {
    val payload = JSONObject().apply {
        put("path", filePath)
        put("mimeType", mimeType)
        put("id", operationId)
    }
    NativeActionCoordinator.dispatchEvent(
        activity,
        "Vendor\\MyPlugin\\Events\\OperationCompleted",
        payload.toString()
    )
}
```

### Async Operations (Kotlin)

For long-running operations, return immediately and dispatch event when done:

```kotlin
import kotlinx.coroutines.*
import java.util.UUID

class AsyncOperation(private val activity: FragmentActivity) : BridgeFunction {
    override fun execute(parameters: Map<String, Any>): Map<String, Any> {
        val id = UUID.randomUUID().toString()

        CoroutineScope(Dispatchers.IO).launch {
            try {
                val result = performLongOperation()

                // Dispatch event on main thread
                withContext(Dispatchers.Main) {
                    val payload = JSONObject().apply {
                        put("id", id)
                        put("result", result)
                        put("success", true)
                    }
                    NativeActionCoordinator.dispatchEvent(
                        activity,
                        "Vendor\\MyPlugin\\Events\\OperationCompleted",
                        payload.toString()
                    )
                }
            } catch (e: Exception) {
                withContext(Dispatchers.Main) {
                    val payload = JSONObject().apply {
                        put("id", id)
                        put("error", e.message)
                        put("success", false)
                    }
                    NativeActionCoordinator.dispatchEvent(
                        activity,
                        "Vendor\\MyPlugin\\Events\\OperationFailed",
                        payload.toString()
                    )
                }
            }
        }

        // Return immediately with tracking ID
        return BridgeResponse.success(mapOf("id" to id))
    }

    private suspend fun performLongOperation(): String {
        delay(1000) // Simulate work
        return "completed"
    }
}
```

### Launching an Activity (Kotlin)

For camera, scanners, complex UI:

```kotlin
import android.content.Intent

class OpenScanner(private val activity: FragmentActivity) : BridgeFunction {
    override fun execute(parameters: Map<String, Any>): Map<String, Any> {
        val config = parameters["config"] as? String

        val intent = Intent(activity, ScannerActivity::class.java).apply {
            putExtra("config", config)
        }
        activity.startActivity(intent)

        return BridgeResponse.success(mapOf("launched" to true))
    }
}
```

The Activity dispatches events when done:

```kotlin
class ScannerActivity : AppCompatActivity() {
    private fun onScanComplete(result: String) {
        Handler(Looper.getMainLooper()).post {
            val payload = JSONObject().apply {
                put("result", result)
            }
            NativeActionCoordinator.dispatchEvent(
                this,
                "Vendor\\MyPlugin\\Events\\ScanCompleted",
                payload.toString()
            )
        }
        finish()
    }
}
```

### Permission Checking (Kotlin)

```kotlin
import android.Manifest
import android.content.pm.PackageManager
import androidx.core.content.ContextCompat

class RequiresCamera(private val activity: FragmentActivity) : BridgeFunction {
    override fun execute(parameters: Map<String, Any>): Map<String, Any> {
        if (ContextCompat.checkSelfPermission(activity, Manifest.permission.CAMERA)
            != PackageManager.PERMISSION_GRANTED) {
            return BridgeResponse.error(BridgeError("PERMISSION_DENIED", "Camera permission required"))
        }

        // Proceed with camera operation
        return performCameraOperation()
    }
}
```

---

## Swift Bridge Function Pattern

### Basic Template

```swift
import Foundation

enum MyPluginFunctions {

    /// Brief description of what this function does.
    ///
    /// - Parameters:
    ///   - paramName: Description of parameter
    ///
    /// - Returns: Dictionary containing:
    ///   - resultKey: Description of return value
    class Execute: BridgeFunction {
        func execute(parameters: [String: Any]) throws -> [String: Any] {
            // 1. Extract and validate parameters
            guard let param1 = parameters["param1"] as? String else {
                return BridgeResponse.error(code: "INVALID_PARAMETERS", message: "param1 is required")
            }

            // 2. Perform the native operation
            do {
                let result = try performOperation(param1)

                // 3. Return success response
                return BridgeResponse.success(data: [
                    "result": result
                ])
            } catch {
                return BridgeResponse.error(code: "OPERATION_FAILED", message: error.localizedDescription)
            }
        }

        private func performOperation(_ param: String) throws -> String {
            return "processed: \(param)"
        }
    }

    class GetStatus: BridgeFunction {
        func execute(parameters: [String: Any]) throws -> [String: Any] {
            return BridgeResponse.success(data: [
                "status": "ready",
                "version": "1.0.0"
            ])
        }
    }
}
```

### File Structure

- One `{Namespace}Functions.swift` file per plugin
- Use `enum` as namespace container (prevents instantiation)
- Each bridge function is a `class` inside the enum

### Parameter Extraction Patterns

```swift
// Required string
guard let name = parameters["name"] as? String else {
    return BridgeResponse.error(code: "INVALID_PARAMETERS", message: "name is required")
}

// Required number
guard let count = parameters["count"] as? Int else {
    return BridgeResponse.error(code: "INVALID_PARAMETERS", message: "count is required")
}

// Optional with default
let quality = parameters["quality"] as? Int ?? 80
let enabled = parameters["enabled"] as? Bool ?? false

// Double/Float
let amount = parameters["amount"] as? Double ?? 0.0

// Arrays
let items = parameters["items"] as? [String] ?? []
let numbers = parameters["numbers"] as? [Int] ?? []

// Nested dictionary
if let config = parameters["config"] as? [String: Any],
   let configValue = config["key"] as? String {
    // Use configValue
}

// Array of dictionaries
let people = parameters["people"] as? [[String: Any]] ?? []
for person in people {
    if let name = person["name"] as? String,
       let age = person["age"] as? Int {
        // Process person
    }
}
```

### Response Patterns

```swift
// Success with data
return BridgeResponse.success(data: [
    "path": filePath,
    "size": fileSize,
    "metadata": [
        "width": width,
        "height": height
    ]
])

// Success with array
return BridgeResponse.success(data: [
    "items": [
        ["id": 1, "name": "Item 1"],
        ["id": 2, "name": "Item 2"]
    ]
])

// Error (always include code and message)
return BridgeResponse.error(code: "OPERATION_FAILED", message: "Something went wrong")

// Error with specific code
return BridgeResponse.error(code: "FILE_NOT_FOUND", message: "The specified file does not exist")
```

### Event Dispatching (Swift)

```swift
// Dispatch on main thread (usually already there)
DispatchQueue.main.async {
    let payload: [String: Any] = [
        "path": filePath,
        "mimeType": mimeType,
        "id": operationId
    ]
    LaravelBridge.shared.send?(
        "Vendor\\MyPlugin\\Events\\OperationCompleted",
        payload
    )
}
```

### Async Operations (Swift)

```swift
class AsyncOperation: BridgeFunction {
    func execute(parameters: [String: Any]) throws -> [String: Any] {
        let id = UUID().uuidString

        // Dispatch to background
        DispatchQueue.global(qos: .userInitiated).async {
            do {
                let result = self.performLongOperation()

                // Dispatch event back on main thread
                DispatchQueue.main.async {
                    LaravelBridge.shared.send?(
                        "Vendor\\MyPlugin\\Events\\OperationCompleted",
                        [
                            "id": id,
                            "result": result,
                            "success": true
                        ]
                    )
                }
            } catch {
                DispatchQueue.main.async {
                    LaravelBridge.shared.send?(
                        "Vendor\\MyPlugin\\Events\\OperationFailed",
                        [
                            "id": id,
                            "error": error.localizedDescription,
                            "success": false
                        ]
                    )
                }
            }
        }

        // Return immediately with tracking ID
        return BridgeResponse.success(data: ["id": id])
    }

    private func performLongOperation() -> String {
        Thread.sleep(forTimeInterval: 1.0) // Simulate work
        return "completed"
    }
}

// Modern Swift concurrency version
class ModernAsyncOperation: BridgeFunction {
    func execute(parameters: [String: Any]) throws -> [String: Any] {
        let id = UUID().uuidString

        Task {
            let result = await performAsyncWork()

            await MainActor.run {
                LaravelBridge.shared.send?(
                    "Vendor\\MyPlugin\\Events\\OperationCompleted",
                    ["id": id, "result": result]
                )
            }
        }

        return BridgeResponse.success(data: ["id": id])
    }

    private func performAsyncWork() async -> String {
        try? await Task.sleep(nanoseconds: 1_000_000_000)
        return "completed"
    }
}
```

### Presenting a ViewController (Swift)

```swift
import UIKit

class OpenScanner: BridgeFunction {
    func execute(parameters: [String: Any]) throws -> [String: Any] {
        // Get the key window's root view controller
        guard let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
              let rootVC = windowScene.windows.first?.rootViewController else {
            return BridgeResponse.error(code: "VIEW_CONTROLLER_ERROR", message: "Cannot present view controller")
        }

        // Find the topmost presented controller
        var topVC = rootVC
        while let presented = topVC.presentedViewController {
            topVC = presented
        }

        // Create and present your view controller
        let scannerVC = ScannerViewController()
        scannerVC.modalPresentationStyle = .fullScreen

        DispatchQueue.main.async {
            topVC.present(scannerVC, animated: true)
        }

        return BridgeResponse.success(data: ["presented": true])
    }
}

class ScannerViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }

    private func onScanComplete(result: String) {
        dismiss(animated: true) {
            LaravelBridge.shared.send?(
                "Vendor\\MyPlugin\\Events\\ScanCompleted",
                ["result": result]
            )
        }
    }
}
```

### Permission Handling (Swift)

```swift
import AVFoundation

class RequiresCamera: BridgeFunction {
    func execute(parameters: [String: Any]) throws -> [String: Any] {
        let status = AVCaptureDevice.authorizationStatus(for: .video)

        switch status {
        case .authorized:
            return performCameraOperation()

        case .notDetermined:
            // Request permission asynchronously
            AVCaptureDevice.requestAccess(for: .video) { granted in
                DispatchQueue.main.async {
                    LaravelBridge.shared.send?(
                        "Vendor\\MyPlugin\\Events\\PermissionResult",
                        ["granted": granted, "permission": "camera"]
                    )
                }
            }
            return BridgeResponse.success(data: ["pending": true])

        case .denied, .restricted:
            return BridgeResponse.error(
                code: "PERMISSION_DENIED",
                message: "Camera permission denied. Please enable in Settings."
            )

        @unknown default:
            return BridgeResponse.error(code: "UNKNOWN_STATUS", message: "Unknown permission status")
        }
    }
}
```

---

## Common Patterns for Both Platforms

### Synchronous vs Asynchronous

**Synchronous** (returns data immediately):
```php
$status = nativephp_call('MyPlugin.GetStatus', []);
// Returns: ['status' => 'ready']
```

**Asynchronous** (returns ID, dispatches event later):
```php
$result = nativephp_call('MyPlugin.StartLongOperation', ['data' => $data]);
// Returns: ['id' => 'uuid-here']
// Later: native:Vendor\MyPlugin\Events\OperationCompleted fires
```

### Error Handling Best Practices

1. **Validate all required parameters** at the start
2. **Use try-catch** around operations that can fail
3. **Return meaningful error codes** for programmatic handling
4. **Include helpful error messages** for debugging
5. **Never throw exceptions** from bridge functions - always return error responses

### Threading Rules

**Android:**
- Bridge functions execute on main thread
- Use `Dispatchers.IO` for I/O operations
- Event dispatch MUST be on main thread: `Handler(Looper.getMainLooper()).post { }`

**iOS:**
- Bridge functions execute on main thread
- Use `DispatchQueue.global(qos: .userInitiated)` for background work
- Event dispatch should be on main thread: `DispatchQueue.main.async { }`

### File Paths

When returning file paths to PHP:

```kotlin
// Android
return BridgeResponse.success(mapOf(
    "path" to file.absolutePath  // Full path like /data/data/com.app/files/photo.jpg
))
```

```swift
// iOS
return BridgeResponse.success(data: [
    "path": fileURL.path  // Full path like /var/mobile/.../Documents/photo.jpg
])
```

PHP can then use these paths directly for file operations.