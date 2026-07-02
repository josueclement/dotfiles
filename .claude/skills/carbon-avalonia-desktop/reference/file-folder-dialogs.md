# File & Folder pickers

`IFileDialogService` / `IFolderDialogService` (`…Services`) wrap Avalonia's `IStorageProvider`. They need
the provider set once at startup: `service.SetStorageProvider(mainWindow.StorageProvider)` in
`App.OnFrameworkInitializationCompleted` (see `setup.md`). Using them before that throws
`InvalidOperationException("Storage provider is not set")`.

## Two API layers

**Raw interface** (returns Avalonia storage items, takes options objects):
- `IFileDialogService`: `Task<IReadOnlyList<IStorageFile>> ShowOpenFileDialogAsync(FilePickerOpenOptions)`,
  `Task<IStorageFile?> ShowSaveFileDialogAsync(FilePickerSaveOptions)`, `IStorageProvider? StorageProvider`,
  `void SetStorageProvider(IStorageProvider)`.
- `IFolderDialogService`: `Task<IReadOnlyList<IStorageFolder>> ShowOpenFolderDialogAsync(FolderPickerOpenOptions)`,
  plus the same `StorageProvider` / `SetStorageProvider`.

**String-path extension overloads** (simpler; return local paths) — in `FileDialogServiceExtensions` /
`FolderDialogServiceExtensions`. **Require `using Carbon.Avalonia.Desktop.Services;`** to be visible:
- `Task<IEnumerable<string>> ShowOpenFileDialogAsync(string? title = null, bool allowMultiple = false, string? suggestedStartLocation = null, string? suggestedFileName = null, IReadOnlyList<FilePickerFileType>? fileTypeFilter = null)`
- `Task<string?> ShowSaveFileDialogAsync(string? title = null, string? suggestedStartLocation = null, string? suggestedFileName = null, string? defaultExtension = null, bool showOverwritePrompt = true, IReadOnlyList<FilePickerFileType>? fileTypeChoices = null)`
- `Task<IEnumerable<string>> ShowOpenFolderDialogAsync(string? title = null, bool allowMultiple = false, string? suggestedStartLocation = null, string? suggestedFileName = null)`

Prefer the extension overloads unless you need `IStorageFile` streams.

## Example

```csharp
public class DialogsPageViewModel(
    IFileDialogService fileDialogs,
    IFolderDialogService folderDialogs) : ObservableObject
{
    private readonly IFileDialogService _files = fileDialogs;
    private readonly IFolderDialogService _folders = folderDialogs;

    private async Task OpenOne()
    {
        var files = await _files.ShowOpenFileDialogAsync(title: "Select a file", allowMultiple: false);
        var path = files.FirstOrDefault();                     // null if cancelled
    }

    private async Task OpenTextFiles()
    {
        var txt = new FilePickerFileType("Text Files")         // Avalonia.Platform.Storage
        {
            Patterns = new[] { "*.txt", "*.md" },
            MimeTypes = new[] { "text/plain", "text/markdown" }
        };
        var files = await _files.ShowOpenFileDialogAsync(
            title: "Select text files", allowMultiple: true,
            fileTypeFilter: new[] { txt, FilePickerFileTypes.All });
    }

    private async Task Save()
    {
        var path = await _files.ShowSaveFileDialogAsync(
            title: "Save document", suggestedFileName: "document", defaultExtension: "txt",
            fileTypeChoices: new[] { new FilePickerFileType("Text File") { Patterns = new[] { "*.txt" } } });
        // path is null if cancelled
    }

    private async Task PickFolder()
    {
        var folders = await _folders.ShowOpenFolderDialogAsync(title: "Select a folder");
        var dir = folders.FirstOrDefault();
    }
}
```

## Common mistakes

- **Missing `using Carbon.Avalonia.Desktop.Services;`** → the string-path overloads (`title:`,
  `allowMultiple:`, …) are invisible and the compiler only sees the options-object methods.
- Calling a picker before `SetStorageProvider` → `InvalidOperationException`.
- Expecting a thrown exception on cancel — open dialogs return an empty sequence, save returns `null`.
