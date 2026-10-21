# TaskLaunchTree

**TaskLaunchTree** adds a sidebar view for discovering and managing tasks defined in the VS Code workspace, including tasks from `tasks.json` and `.code-workspace` files. Optionally, it can find executable files in the workspace folder tree.

## Demo

![TaskLaunchTree demo](https://raw.githubusercontent.com/eugenelk/task-launch-tree-media/main/media/task-launch-tree-demo.gif)

## Use

Select **TaskLaunchTree** in the **Activity Bar** to browse  and run available tasks. Select **Run** to start a task, **Stop** to terminate a running task, or **double-click** its row to run it. Use the **expand-all/collapse-all** and **refresh** icons in the view title to fold or unfold every group and reload the task list, or use a group header to fold it individually.

Tasks can be **starred**. Use star icon or right-click a task or executable leaf (including a starred shortcut) for **Run**, **Stop**, and **Star** or **Un-Star**.

The view has its own **Activity Bar** container by default. Use **Move View** from its header context menu or drag its header to stack it inside **Explorer** or another container. Use **View: Reset View Locations** to restore the default layout (this resets locations for all views).

The **last used task** will be mapped to **Run** icon on the view toolbar and to **Status** bar for quick and convenient re-run. The last successfully started task is remembered per workspace across VS Code restarts and window reloads, including discovered executable files. The rerun controls are restored at startup without opening the sidebar, but stay hidden while that task is unavailable.

If the remembered task is unavailable, an error is written to the **TaskLaunchTree** Output channel. Repeated refreshes do not repeat the error unless the task becomes available and then unavailable again.

# Task Tree Layout

The tree has these top-level groups:

- **starred** 
  
  appears first, only while at least one starred task is available. It contains flat shortcuts to starred tasks and executables without moving them out of their original groups.

- **vscode** 
  
  contains VS Code tasks, explicitly configured in workspace `tasks.json` and `.code-workspace` files.

- **workspace** 
  
  contains discovered executable files and is shown only when `taskLaunchTree.detectExecutables` is enabled.

# Controlling Node View State

Groups start collapsed and can each be folded independently. Expanded state is preserved across sessions and when running or stopping tasks and refreshing the view.

# Group: starred

Hover over a task under **vscode** or **workspace** and select the yellow outline **Star** icon immediately left of its **Run** button. Select it again to remove the star, including from the shortcut under **starred**. A starred task displays a yellow star outline over its green task icon in both locations. The shortcuts support the same **Run**, **Stop**, and **double-click** actions as the original leaves.

Star selections and the **starred** group's expanded state are saved per workspace across VS Code restarts and window reloads. Unavailable tasks and executables keep their saved stars but are hidden until available again.

# Group: vscode

Tasks under **vscode** can be divided into nested groups using task labels and separators. Each separator creates a branch or sub-group. Leading and trailing whitespace is stripped from each task label segment.

Configure separators with `taskLaunchTree.delimiters`. The default is `:`. For example, `02-Build:Frontend:Watch:Development` appears as foldable groups

```
02-Build
  |
  -Frontend
    |
    -Watch
      > Development (as the runnable leaf)
```

Each group header has an **expand-all/collapse-all** icon that controls that branch and all its nested groups without changing siblings. Tasks and groups are sorted alphabetically at each level.

# Group: workspace

Under **workspace**, each workspace folder is a group, followed by its subfolders and executable file leaves. Folder and file names are kept literal. Only files that match file extension list in `taskLaunchTree.detectExecutables` will be shown. **Run**, **Stop**, and **double-click** work just like for VS Code tasks. 

Files run as shell tasks with **workspace root directory as the working directory**, using the configured VS Code task shell. The configured file extensions select files, they do not grant execution permissions or install required interpreters.

## Executable discovery

Executable discovery is disabled by default. Enable `taskLaunchTree.detectExecutables` and configure
`taskLaunchTree.executableExtensions` with file  extension patterns:

```json
{
  "taskLaunchTree.detectExecutables": true,
  "taskLaunchTree.executableExtensions": ["*.bat", "*.cmd", "*.sh"]
}
```

The default filter is `["*.bat"]`. Matching is case-insensitive. An empty list matches no files. Every workspace folder is scanned recursively, including external folders referenced by an open `.code-workspace` file. Symbolic links are not followed. Use **Refresh** after adding or removing files.

## Exclusions

Set `taskLaunchTree.executableExclusions` to an array of JavaScript regex source strings (no surrounding `/` delimiters). The default is `[]`. Each expression is tested against both the absolute path and the path relative to its workspace root, using native separators and `/`-normalized separators. Matching is case-sensitive. Any matching expression excludes the path. Matching a folder skips its entire subtree. Workspace roots can also be excluded by absolute path. Empty regex strings are ignored and match nothing.

```json
{
  "taskLaunchTree.executableExclusions": [
    "(^|/)(node_modules|excluded)(/|$)",
    "^tools/private(/|$)",
    "^D:/Shared/generated(/|$)",
    "\\.generated\\.bat$"
  ]
}
```

An invalid regex is reported as an error (and logged, see [**Logging**](#Logging) below) but does not stop discovery. The rest of the configured exclusions still apply and scanning continues.
Setting changes refresh discovery automatically. Exclusions affect only executable discovery, not tasks under **vscode** group.

Set `taskLaunchTree.flattenWorkspaceNode` to `true` to initially show only executable leaves directly under **workspace**, without folder groups. The default is `false`. 

The **workspace** node shows **list-flat** in tree mode and **list-tree** in flat mode. Click corresponding icon it to switch modes independently of the setting. Manual view mode changes survive refreshes and task updates for the current VS Code session. Setting changes re-apply on extension restart. 

In flat mode, leaf tooltips show **Path** and the folder relative to its workspace root using `/` separators (`.` for the root).

# Logging

The extension writes to its own **TaskLaunchTree** Output channel (**View: Output**, then select **TaskLaunchTree**). Errors always appear there. Set `taskLaunchTree.logging` to control detail:

- `"errors"` (default) logs only failures.
- `"full"` also logs each executable discovery scan: workspace folders scanned, configured patterns, every found file, every excluded file, every pruned directory, the matched file count, and elapsed time.
