# IniReflector
This [RagePluginHook](https://ragepluginhook.net) script provides object-oriented reading and writing of .ini config files for your RPH/LSPDFR plugins!\
The idea is that all you do is create a class that resembles your .ini file and the script reads the .ini values into their respective class members.\
The script will generate missing settings (even the whole file if it is missing), so when you add a new .ini setting to your plugin, you just need to add it to the settings class: the script will do the rest on the users end!\
If this wasn't already enough, the script supports providing descriptions for settings which will be auto-generated when a setting value is serialized.

# Setting up a config class
Class fields/properties that represent a .ini setting need to be marked with the\
`IniReflectorValue(string sectionName = null, string name = null, object defaultValue = null, string description = null)` attribute:
- `sectionName`: The section of the setting. If `null`, see the `IniReflectorSection` attribute below.
- `name`: The name of the setting in the .ini. If `null`, it defaults to the name of the field/property.
- `defaultValue`: The value that should be used incase it doesn't exist in the .ini. If `null`, read below how default values are handled.
- `description`: The description of the setting. If `null`, no description is written during serialization.
- Example:
```cs
internal class Config
{
    [IniReflectorValue(sectionName: "Keybinds", defaultValue: Keys.T, description: "The key to open the main menu.")]
    internal Keys OpenMenu;
}
```

\
If you got several settings of the same section, it might be easier to declare a `IniReflectorSection(string name)` on the class.
This way you can include the name of the section into the name of the class member and it will automatically resolve the section:
- `name`: The name of the section.
- Example:
```cs
[IniReflectorSection("Keybinds")]
internal class Config
{
    [IniReflectorValue(defaultValue: Keys.T, description: "The key to open the main menu.")]
    internal Keys KeybindsOpenMenu;
}
```

\
If you don't provide any `defaultValue` then the script will attempt to look for a class member with the **same name and a `Default` prefix**.
If it cannot find any it will fallback to `null` for reference types and `default` for value types.
- Example:
```cs
[IniReflectorSection("Keybinds")]
internal class Config
{
    private const Keys DefaultKeybindsOpenMenu = Keys.T;
    
    [IniReflectorValue(description: "The key to open the main menu.")]
    internal Keys KeybindsOpenMenu;
}
```

\
Generally speaking, the `IniReflectorValue` attribute on a class member is prioritized, meaning that the script will always check the parameters of the attribute first before attempting to resolve the parameters using the above mentioned methods.\
The expected .ini from the example above would look like this:
```ini
[Keybinds]
//The key to open the main menu.
KeybindsOpenMenu=T
```

# Reading config
All you do is create an instance of the `IniReflector<T>(string path)` and an instance of the `T` config class that the values should be written to:
- `path`: GTA5 relative path to your config file.
- Example:
```cs
public static class EntryPoint
{
    internal static readonly Config UserConfig = new();
    
    public static void Main()
    {
        IniReflector<Config> iniReflector = new("plugins/MyPlugin.ini");
        iniReflector.Read(UserConfig, true);

        while (true)
        {
            GameFiber.Yield();
            
            if (Game.IsKeyDown(UserConfig.KeybindsOpenMenu))
            {
                // Open menu
            }
        }
    }
}
```

\
Simply copy-paste the [IniReflector.cs](https://github.com/Sprayxe/IniReflector/blob/main/IniReflector.cs) to your project and follow the tutorial above.\
You might also wanna check out my [LSPDFR Plugin Reloader](https://github.com/Sprayxe/LSPDFRPluginReloader).\
Enjoy! :)

# API
- `IniReflector<T>(string path)`: Creates instance
    - `T`: Config class
    - `path`: GTA5 relative path to .ini file
- `IniReflector(string path, Type iniModel)`: Creates instance
    - `path`: GTA5 relative path to .ini file
    - `iniModel`: Type of config class
- `void Read(object obj, bool withLogging)`: Reads the .ini file into the provided object
    - `obj`: Instance of config class to deserialize into
    - `withLogging`: Whether to have verbose logging output or not
- `void Write(object obj, bool withLogging)`: Writes the provided object to the .ini file
    - `obj`: Instance of config class to serialize
    - `withLogging`: Whether to have verbose logging output or not
- `bool WriteSingle(string memberName, object newValue)`: Writes a single setting to the .ini file
    - `memberName`: Name of the field/property that is being written (required for getting `IniReflectorValue` info)
    - `newValue`: The new value of the setting

# Full Example Code
```cs
[assembly: Plugin("MyPlugin", Description = "Example plugin using IniReflector!", PrefersSingleInstance = true)]
namespace MyPlugin;

public static class EntryPoint
{
    internal static readonly Config UserConfig = new();
    
    public static void Main()
    {
        IniReflector<Config> iniReflector = new("plugins/MyPlugin.ini");
        iniReflector.Read(UserConfig, true);

        while (true)
        {
            GameFiber.Yield();
            
            if (Game.IsKeyDown(UserConfig.KeybindsOpenMenu))
            {
                // Open menu
            }
        }
    }
}

[IniReflectorSection("Keybinds")]
internal class Config
{
    private const Keys DefaultKeybindsOpenMenu = Keys.T;
    
    [IniReflectorValue(description: "The key to open the main menu.")]
    internal Keys KeybindsOpenMenu;
}
```
