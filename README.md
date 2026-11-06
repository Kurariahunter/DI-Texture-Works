# DI Texture Works

A Windows desktop tool for finding, previewing, and extracting character textures from **Disney Infinity PC Gold Edition** assets.

DI Texture Works follows texture references in material bundles (`.mtb` files), links them to available texture files, and organizes the results in a searchable character tree. A saved JSON index lets you return to your findings without rebuilding the index every time.

This is a standalone **C# / .NET 8 WPF** application. Unity, QuickBMS, and AssetWiiUdio are not required to use it.

## Features

- **Character-focused indexing:** discovers material references under the character folder, with targeted lookup of textures stored elsewhere in the assets directory.
- **Material reference details:** shows texture IDs, material/shader information, sampler usage, source archives, and candidate matches.
- **Searchable character tree:** browse characters and texture references; trees start collapsed.
- **Texture preview:** view supported images with Fit and 100% controls.
- **Original extraction:** save the resolved source texture without converting it.
- **PNG export:** export the supported decoded image at its original top-mip dimensions.
- **Batch extraction:** export all resolved textures and report failures.
- **Persistent JSON index:** save/load findings, source information, and supported archive-member locations.
- **Background operations:** progress messages, cancellation, and background preview loading.
- **Performance caches:** bounded in-memory caches and targeted archive-member reads reduce repeated loading work.

The tool reads game assets and writes indexes, exports, and settings. It does not edit or repack the game's archives.

## Requirements

- Windows.
- A local Disney Infinity PC Gold Edition assets folder.
- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) to build from source.

The project uses WPF, with Windows Forms for folder selection. No external NuGet packages are declared in the reviewed project.

## Build and run

Open a terminal in the folder containing `DITextureWorks.csproj`:

```powershell
dotnet build -c Release
dotnet run -c Release
```

Alternatively, open the project in Visual Studio with the .NET desktop development workload installed.

To publish a self-contained 64-bit Windows build:

```powershell
dotnet publish -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -o publish
```

The published application is `publish\DITextureWorks.exe`. A self-contained build does not require users to install the .NET runtime separately.

## Getting started

1. Click **Choose Assets Folder** and select the game's `assets` directory or its `characters` subfolder.
2. Click **Build / Rebuild Index**. The scanner discovers character material references and searches for their texture sources.
3. Expand a character and select a texture reference.
4. Review its resolution status, source details, and preview.
5. Use **Extract Selected** for the original file, **Export PNG** for a decoded image, or **Export All Resolved** for batch extraction.

Use the search field to narrow the tree. **Load JSON** opens a previously saved index; **Save JSON** lets you keep a separate copy.

## Saved indexes and loading performance

Settings and the default index are stored under:

```text
%LOCALAPPDATA%\DITextureWorks\
```

The application remembers the selected assets folder and index, and reloads an available saved index on startup.

The current index format stores supported DI archive-member locations, sizes, compression metadata, CRCs, and source-file fingerprints. Preview/extraction can read the selected member without loading an entire archive into memory. Source changes invalidate saved locations and cause reindexing as needed.

Older indexes can gain location metadata as textures are viewed. Rebuilding once populates the locations during the scan. Decoded preview images are cached in memory; they are not embedded in the JSON index.

## Supported preview formats

- DDS DXT1 / BC1.
- DDS DXT5 / BC3.
- Supported uncompressed 32-bit BGRA/RGBA DDS, including alpha and row pitch.
- PNG.

These include the DDS data used by tested PC `.tbody` files. Preview displays the texture image itself, not a rendered character or full game material. Normal maps may appear purple, and tiny solid-colour textures can be valid assets.

Unsupported, missing, or ambiguous sources are reported rather than silently replaced with a different image. Original extraction remains available for resolved files even when their image format cannot be previewed.

## Scope and limitations

- The primary target is **PC Gold Edition**. This is not a general Wii U/PS3/PS4 texture converter or a browser for arbitrary hashed console packages.
- Reads loose files, ordinary ZIPs, and supported proprietary DI archives. Additional compression methods, encryption variants, and shader profiles may need further support.
- Material references can be decoded without proving every texture's rendering role. PC colour-preview selection uses the reviewed slot convention; it does not reproduce all game shader behavior.
- An unresolved reference means the tool did not resolve it within its supported lookup scope—not necessarily that the game is missing the asset.
- Preview exports the top mip, not a complete mip chain or streaming-texture reconstruction.
- The complete PC asset library has not been exhaustively validated. Scan and extraction diagnostics should be retained when reporting problems.
- Texture editing, archive repacking, and 3D character rendering are outside the current tool's scope.

## Source layout

| File or folder | Purpose |
|---|---|
| `MainWindow.xaml` / `.cs` | Interface, indexing operations, tree, and preview controls |
| `AssetScanner.cs` | Character discovery and texture-source resolution |
| `MtbTextureTable.cs` | Material texture-reference decoding |
| `IndexModels.cs` | Saved index and source-reference models |
| `TargetedArchive.cs` | Validated targeted DI archive-member reads |
| `TexturePreview.cs` | Supported image decoding and preview caching |
| `BoundedCache.cs` | Cache budgeting and eviction |
| `Engine/` | Supporting format and archive code |

## Contributing and reporting issues

Contributions are welcome for additional texture formats, verified material/sampler profiles, archive compatibility, and interface improvements.

For a useful issue report, include the application revision, game version, operation attempted, texture ID, resolution details, and exact diagnostic message. Note whether the source is loose or archived and whether rebuilding the index changes the result. Avoid uploading copyrighted game assets; diagnostic metadata or synthetic test files are preferable.

## Acknowledgments

Developed through Disney Infinity archive, material, and texture research, including work shared with the AssetWiiUdio toolchain. Preserve applicable author credits and licenses when reusing source components.

Disney Infinity and its assets belong to their respective rights holders. This is an unofficial community tool and is not affiliated with or endorsed by Disney or Avalanche Software. Game assets are not included.
