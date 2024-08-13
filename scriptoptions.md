# Script Options

In PySpigot's main plugin folder, you'll find a file titled `script_options.yml`. This file enables you to specify various script-specific options, all of which are covered here.

!> Defining script options for each script is *optional*; see the [Defaults](#defaults) section for more info.

## Basic Format

If you would like to specify options for a script, do it like so:

``` yaml
test.py:
  enabled: true
```

Each section in the `script_options.yml` file should be the name of a script. The name must *exactly* match the script's name, (including `.py`), or the options won't be parsed for that script. All options within the section apply to the script by which the section is named. In the above example, the option `enabled` applies to the script `test.py`.

If you would like to define options for multiple scripts, do it like so:

``` yaml
test.py:
  enabled: true
  file-logging-enabled: true
test2.py:
  enabled: true
  depend: ['test.py']
```

As you can see, in the above code snippet, we define a new section titled `test2.py`, and all options within this section will apply to the `test2.py` script. All options within the `test.py` script apply to the script `test.py`.

## Defaults

Specifying script options for each script is not a requirement. Furthermore, you may omit certain script options if you do not wish to set them. All options have default values, which can be customized in the PySpigot `config.yml` file, under the `script-option-defaults` section.

If a script does not have any script options defined in `script_options.yml`, then PySpigot will fall back to whatever default values are defined in the `config.yml`. If a default value is not defined in the `config.yml`, then an internal fallback value is used. These internal fallback values are listed below under each option.

## Options

### enabled

Used to enable or disable a script. To disable a script, set this value to `false`.

Default: `true` (enabled)
Usage:
``` yaml
test.py:
  enabled: true
```

### depend

Used to specify a list of other scripts that this script requires to load. This will ensure the script loads *after* all of its dependency scripts are loaded. Furthermore, if any of the script's dependencies are missing, the script will not load. To list a dependency, use the full name of the script (including `.py`).

?> This value is a list (even if there is only one item!), and it should be formatted as `depend: ['item1', 'item2', 'item3', ...]`

Default: None
Usage:
``` yaml
test.py:
  depend: ['test1.py', 'test2.py']
```

### plugin-depend

Used to specify a list of plugins that this script requires to load. The script will not load if any of the plugin dependencies are not loaded and running on the server. Additionally, when a plugin is unloaded/disabled, any scripts that depend on that plugin as specified under this option are automatically unloaded (if the `script-unload-on-plugin-disable` option in the `config.yml` is set to `true`).

?> If you are working with ProtocolLib or PlaceholderAPI in your script, you **do not** need to specify either of them here. PySpigot has built-in support for these two plugins, and the dependency management is handled internally.

?> This value is a list (even if there is only one item!), and it should be formatted as `plugin-depend: ['item1', 'item2', 'item3', ...]`

Default: None
Usage:
``` yaml
test.py:
  plugin-depend: ['test1.py', 'test2.py']
```

### file-logging-enabled

Specify if script file logging should be enabled for the script. If this option is `true`, a script log file will be generated for the script, and any error messages (and print messages sent to the script's logger) will be logged to this file. If this option is `false`, no messages will be logged to file, but messages will still be printed to the server console.

Default: `log-to-file` value in PySpigot's `config.yml`, which is `true` by default
Usage:
``` yaml
test.py:
  file-logging-enabled: true
```

### min-logging-level

Specify the minimum logging level that should be logged to the script's log file and the console for the script. Options can be found on the [Javadocs](https://docs.oracle.com/javase/8/docs/api/java/util/logging/Level.html).

Default: `min-log-level` value in PySpigot's `config.yml`, which is `INFO` by default
Usage:
``` yaml
test.py:
  min-log-level: 'INFO'
```

### permissions

Specify a list of permissions that the script uses. This is useful for scripts that want to restrict access to certain features. This section is defined in the [exact same way](https://docs.papermc.io/paper/dev/plugin-yml#permissions) that permissions are defined in the `plugin.yml` file for a Bukkit plugin. See usage code example below for how to define permissions, defaults, and child permissions.

Default: No permissions defined (empty list)
Usage:
```yaml
test.py:
  permissions:
    permission.node:
      description: 'This is a permission node'
      default: op
      children:
        permission.node.child: true
    another.permission.node:
      description: 'This is another permission node'
      default: not op
```

- `description` is a description of the permission node, and this is what will be displayed in the permissions list. The default value is the name of the permission node.
- `default` is the default value of the permission node, or, in other words, who should have the permission node by default. There are four possible values for `default`: 
  - `op`: Only server operators will have the permission node by default.
  - `not op`: Players who are not operators will have the permission node by default.
  - `true`: All players will have the permission (I.E. it is a default permission).
  - `false`: No players will have the permission (I.E. it is *not* a default permission). The default value is the value of `default_permission` (outlined below).
- `children` is a list of child permissions that should inherit from the parent permission. Each permission node may have children. When set to `true`, the child will inherit the parent permission.

### default-permission

Specify a default value that permissions should have, if they do not have a `default` value defined.

Default: `op`
Usage:
```yaml
test.py:
  permission-default: true
```

The allowed values for `permission-default` are `op`, `not op`, `true`, and `false`:

- `op`: Only server operators will have the permission node by default.
- `not op`: Players who are not operators will have the permission node by default.
- `true`: All players will have the permission (I.E. it is a default permission).
- `false`: No players will have the permission (I.E. it is *not* a default permission). 