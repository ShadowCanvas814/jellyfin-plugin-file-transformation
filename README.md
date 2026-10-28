<h1 align="center">File Transformation</h1>
<h2 align="center">A Jellyfin Plugin</h2>
<p align="center">
	<img alt="Logo" src="https://raw.githubusercontent.com/ShadowCanvas814/jellyfin-plugin-file-transformation/main/src/logo.png" />
	<br />
	<br />
	<a href="https://github.com/ShadowCanvas814/jellyfin-plugin-file-transformation/?tab=License-1-ov-file">
		<img alt="GPL 3.0 License" src="https://img.shields.io/github/license/ShadowCanvas814/jellyfin-plugin-file-transformation.svg" />
	</a>
	<a href="https://github.com/ShadowCanvas814/jellyfin-plugin-file-transformation/releases">
		<img alt="Current Release" src="https://img.shields.io/github/release/ShadowCanvas814/jellyfin-plugin-file-transformation.svg" />
	</a>
</p>

## Used By

- [Media Bar (ShadowCanvas814's fork, Jellyfin 12.4)](https://github.com/ShadowCanvas814/jellyfin-plugin-media-bar)

## Jellyfin 12.4 Compatibility Notes

This fork targets Jellyfin 12.4 (see `.github/workflows/build.yml`, which builds against `JellyfinVersion=12.4.0`). Two things worth knowing if you're running on 12.4:

- **Build packaging:** `dotnet publish` always copies the plugin's full transitive dependency tree (including Jellyfin's own `MediaBrowser.Common`/`Controller`/`Model` assemblies) into the output folder. If those get zipped up alongside the plugin and deployed, Jellyfin loads its own bundled copies into the plugin's assembly context, the plugin's `IPlugin` interface then resolves against a different assembly identity than the server's, and Jellyfin's `PluginManager` silently excludes the plugin from its instantiation list — no error is logged, it just never loads. The release zip built by this fork's workflow only includes the plugin's own `.dll`/`.deps.json`/`.pdb`/`logo.png`, which avoids this.
- **Home Screen Sections incompatibility:** once File Transformation is actually loading and running its registered transformations on 12.4, the [Home Screen Sections](https://github.com/IAmParadox27/jellyfin-plugin-home-sections) plugin's injected script (`home-screen-sections.js`) throws an uncaught `TypeError: Cannot read properties of undefined (reading 'tv')` while patching the home tab. This crashes Jellyfin's web client before it finishes mounting (`#reactRoot` stays empty — no home page content, nothing to scroll to), even though File Transformation and other plugins using it (e.g. Media Bar, Jellyfin Enhanced) work fine. Until Home Screen Sections has its own 12.4 compatibility fix, disable it (Dashboard → Plugins → Home Screen Sections) if you run into a blank/frozen home page after installing this build.

## Introduction
File Transformation is a Jellyfin Plugin that can be used to modify the served [jellyfin-web](https://github.com/jellyfin/jellyfin-web) content without having to modify the files directly.

The use cases for this can be seen in plugins like [Media Bar (this fork)](https://github.com/ShadowCanvas814/jellyfin-plugin-media-bar), which takes advantage of this to inject its slideshow into the Jellyfin web client.

### Credit
The original code and concept for this plugin came from this [Pull Request](https://github.com/jellyfin/jellyfin/pull/9095) from [JPVenson](https://github.com/JPVenson). A lot of the code is unmodified (bar personal code standards).

### Benefits

> Why would I use this rather than just asking installers to change the files?

Well, this plugin is non destructive and allows multiple plugins to manipulate the served data. The install remains clean and free for users to update their Jellyfin server whenever they want.

## Installation

1. Trigger the `Build for Jellyfin 12.4` workflow on this repo (or download the latest successful run's artifact from the Actions tab).
2. Extract the downloaded zip into a new folder under your Jellyfin `plugins` directory (e.g. `plugins/FileTransformation-12.4.0`). No configuration is required.

### FAQ

#### I've updated Jellyfin to latest version but I can't see the plugin available in the catalogue

The likelihood is the plugin hasn't been updated for that version of Jellyfin and the plugins are strictly 1 version compatible. Please wait until an update has been pushed. If you can see the version number in the release assets then please make an issue, but if its not in the assets, please wait. I know Jellyfin has updated, I'll update when I can.

### Referencing this as a library
Due to issues with Jellyfin's plugins being loaded into different load contexts this cannot be referenced directly.

Instead you can use reflection to invoke the plugin directly to register your transformation.

1. Prepare your payload
```json
{
    "id": "00000000-0000-0000-0000-000000000000", // Guid
	"fileNamePattern": "", // Regex Patterm for the file to patch
	"callbackAssembly": GetType().Assembly.FullName, // Example value is a string from C# that should be resolved before adding to json
	"callbackClass": "", // The name of the class that should be invoked from the above assembly
	"callbackMethod": "" // The name of the function that should be invoked from the above class
}
```
2. Send your payload to the file transformation assembly
```csharp

Assembly? fileTransformationAssembly =
	AssemblyLoadContext.All.SelectMany(x => x.Assemblies).FirstOrDefault(x =>
		x.FullName?.Contains(".FileTransformation") ?? false);

if (fileTransformationAssembly != null)
{
	Type? pluginInterfaceType = fileTransformationAssembly.GetType("Jellyfin.Plugin.FileTransformation.PluginInterface");

	if (pluginInterfaceType != null)
	{
		pluginInterfaceType.GetMethod("RegisterTransformation")?.Invoke(null, new object?[] { payload });
	}
}
```

When your transformation method is invoked you will receive a object representing the following json format
```json
{
  "contents": "" // String containing the current state of the file being requested.
}
```

## Contribution

### Code Contributions

You're more than welcome to contribute to this plugin in any way that betters it! I only ask that you follow the same code style as myself. A few points to note:

- Please don't commit with any whitespace changes, might be worth turning off auto-linters
- Please don't use `var` unless you have to due to differing namespaces between JF versions (honestly, I'm not going to gripe for the odd one, but it's good to try at least)
- Please at least check the plugin compiles with 10.10.7 and the latest version of JF
- Please put braces on new lines and use them even for 1 line statements

As a general rule of thumb, please try to blend in with the codebase, I use a mutated hungarian notation for my coding style, I will ask for this to be followed.

After following these guidelines, please create a pull request and I'll review it as soon as I can.

Please declare your AI usage, if any, and if you have used AI for your change, please also declare your coding experience without AI. This won't impact whether the change is merged, it will allow me to properly and correctly review the change to ensure that nothing slips through the net.

## Requests
If any functionality is desired to be overridden from Jellyfin's server please open an issue on [this fork's GitHub repo](https://github.com/ShadowCanvas814/jellyfin-plugin-file-transformation/issues).

## FAQ
> Frequent questions will be added here as they are asked.

Ensure that you check the closed issues on GitHub before asking any questions as they may have already been answered.
