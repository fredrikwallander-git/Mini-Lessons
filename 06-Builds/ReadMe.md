# Builds - How to

When you play your game inside the Unity Editor, Unity is simulating your game through the editor’s environment.

When you build your game:

- Compiles your scripts into a standalone executable.
- It packs your scenes, assets, and resources into deployable files.
- The result is a playable game — just like any commercial title.

In other words:

>Building = turning your Unity project into a real, playable game.

## Building for Windows

We’ll focus on Windows first.

🪟 Step-by-Step: Building for Windows

1. **Save all your work**
- Press `Ctrl + S` to save the scene.
- Go to _File → Save_ Project.
2. **Open Build Settings**
- Go to **File → Build Settings**...
- Select **PC, Mac & Linux Standalone**
3. **Choose Target Platform**
- Under _Platform_, select **Windows**
- Make sure Architecture is set to:
  - x86_64 (64-bit)
- Click **Switch Platform** (if not already selected)
4. **Add Your Scenes**
- Under _Scenes in Build_, click **Add Open Scenes**
- You can also drag other scenes into the list (menu, level1, etc.)
5. **Choose Build Folder**
- Click Build
- Choose or create a new folder, e.g., `Builds/Windows`
- Unity will generate:
  - `YourGame.exe`
  - A `YourGame_Data` folder
6. **Test the Build**
- Double-click the `.exe` file to test the standalone version.
- Check that sound, inputs, and UI all work outside the editor.

---

Small tips:

- **Development Build** → Includes the console & debugging info
- **Compression** → Reduces size, slightly longer load times
- **Auto-connect Profiler** → Allows performance profiling in playtest builds

---

### Folder Structure Example

```
SmolTheftAuto/
├── Assets/
├── Builds/
│   ├── Windows/
│   │   ├── SmolTheftAuto.exe
│   │   └── SmolTheftAuto_Data/
└── ProjectSettings/
```

## Side Note: Other Build Targets (for the curious)

Unity is extremely flexible, once you understand the Windows build process, you can apply the same logic to other targets:

| Platform     | Requirements                  | Notes                                     |
| ------------ | ----------------------------- | ----------------------------------------- |
| **Mac**      | macOS, Apple ID (optional)    | Build → macOS (creates `.app`)            |
| **Linux**    | Linux support module          | Creates `.x86_64` executable              |
| **WebGL**    | WebGL module                  | Exports playable HTML game for browser    |
| **Android**  | Android module + SDK + device | Exports `.apk` for phones                 |
| **iOS**      | Mac + Xcode                   | Generates Xcode project for Apple devices |
| **Consoles** | Special Unity license         | Only available with platform dev kits     |

You can install these from:

>Unity Hub → Installs → Add Modules

### Tips Before Building

✅ Remove unused test scenes and debug UI

✅ Check console for red errors before build

✅ Set a proper resolution or full-screen toggle

✅ Include a quit button using Application.Quit()

✅ Disable “Development Build” for final submission

## Bonus: Simple Build Script Example

You can automate your builds using a script!

```csharp
using UnityEditor;
using UnityEngine;

public class BuildTool
{
    [MenuItem("Build/Build Windows")]
    public static void BuildGame()
    {
        string path = "Builds/Windows/SmolTheftAuto.exe";
        string[] scenes = { "Assets/Scenes/MainMenu.unity", "Assets/Scenes/City.unity" };

        BuildPipeline.BuildPlayer(scenes, path, BuildTarget.StandaloneWindows64, BuildOptions.None);
        Debug.Log("✅ Build completed successfully!");
    }
}
```

**Remember** that _any script_ that uses the `using UnityEditor;` **has** to reside in a folder named `Editor` as it is an Editor only script and can not be included in a build.