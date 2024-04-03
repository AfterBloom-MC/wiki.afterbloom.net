# Using External Java/Python Libraries

PySpigot allows you to use external Python and Java libraries when writing scripts. There are many open-source Python and Java libraries that simplify or otherwise hasten code writing. The [Apache Commons Libraries](https://commons.apache.org/), a collection of Java utility libraries that provide an abundance of useful utility functions, are one such example.

Of course, a more advanced usage of this functionality might include writing your own external libraries for frequently used or complicated code. See the [Handy Usage of External Python Modules](#Handy-Usage-of-External-Python-Modules) section below for more information on this.

## General Information

PySpigot will create two folders (if they don't already exist) called `python-libs` and `java-libs` when it loads, where external libraries should be placed.

PySpigot automatically places a helper module called `pyspigot.py` into the `python-libs` folder on plugin load. For more information on this helper library, see the [Writing Scripts](writingscripts#the-pyspigot-helper-module) page.

## External Python Libraries

PySpigot currently supports the usage of a *single-module file* for use as an external library. Currently, fully-fledged libraries/packages (I.E. those with an `__init__.py`) are not supported, but support for these is planned in a future release. Thus, from this point onwards, "external Python library" will be referred to as **module**.

### Loading and Using an External Python Module

Using an external Python module is relatively simple and does not require usage of one of PySpigot's managers or any commands, unlike Java libaries. The external Python module you plan to use should be a single `.py` file. To use it, drag and drop it into the `python-libs` folder within PySpigot's main plugin folder. Then, simply `import` the module into your script, and you're good to go.

?> Adding new modules or deleting old modules does not require a server restart or plugin reload. However, if the code of a module is changed, all scripts that use said module should be reloaded.

### Handy Usage of External Python Modules

Suppose there is a certain piece of code that you use frequently, across multiple scripts. This code could be something like converting a player's location to a config-friendly `dict`, formatting an `str`, or sending a player a message in a neat format. Instead of writing the same code multiple times across different scripts, you might consider breaking this code out into its own external Python module, then add it to the `python-libs` folder, so that you can access it from all your scripts.

## External Java Libraries

Under normal circumstances, a Java library would be "shaded" into a Bukkit/Spigot plugin (with the end result sometimes called a "fat" Jar or "Uber" Jar) so that the plugin has access to the dependency at runtime. We obviously can't do this with scripts.

Instead, we can take advantage of Jython's capabilities. As stated before elsewhere in the documentation, Jython provides access to all loaded Java classes at runtime. Therefore, a collection of Java classes (the library) can be manually loaded into the classpath at runtime, which in turn gives scripts access to those classes. PySpigot's LibraryManager provides this functionality.

When a JAR library is loaded, PySpigot creates a copy of the library that ends with "-relocated". For example, the library `jython-annotation-tools-0.9.0.jar` is copied to `jython-annotation-tools-0.9.0-relocated.jar`. This is done not only to accommodate relocation rules (see the [section on relocation rules](#jar-relocation-rules) below), but also to speed up loading of external libraries in future plugin loads. This behavior occurs even if you don't specify any relocation rules for the library.

### Loading and Using Java Libraries

!> It's possible that another plugin on the server is already using the library/dependency that you'd like to use. You should check if this is the case before you attempt to load the library yourself. You can do this by trying to `import` the dependency into your script. If you're able to import it, **stop here!** You won't need to load it yourself, as it is already loaded (likely by another plugin).

All Java libraries/dependencies *should* have a Jar file available for download somewhere, perhaps on its Github repository or on its official website. You may need to download it manually from the repository where the dependency is hosted. If you can't find the Jar file, reach out on PySpigot's Discord for help.

Once you have the physical Jar file for the dependency you'd like to use, drag and drop it into the `java-libs` folder of the main PySpigot plugin folder. The dependency will be loaded when the PySpigot plugin is loaded and enabled (on server startup). From here, you can use `import` statements in your script to import the desired code from the library.

?> All Jar files present in the `java-libs` folder are loaded *on plugin load* (when the server starts). Loading new Jar files while PySpigot is already running requires usage of a command (see below), or a complete reload of the plugin (`/ps reloadall`).

#### Loading a Java library when PySpigot is already running

Like before, obtain the Jar file for the library you would like to use. Then, drag and drop it into the `java-libs` folder of the main PySpigot plugin folder.

Next, you'll need to load the library manually with the command `/ps loadlibrary <filename>`, where `<filename>` is the name of the Jar file you are loading. Be sure to include the `.jar` extension in the file name. This will import the library.

The library should now be loaded, and you can use `import` statements to import the desired code.

!> PySpigot does not have a way to unload previously loaded libraries due to limitations within Java's class loading system. Nevertheless, a library *will not* be loaded more than once; if it already loaded, PySpigot will not load it again.

### Jar Relocation Rules

You may need to change the name/classpath of classes within a library/dependency. This is called "relocation". There are many reasons why you might want to do this, but that is beyond the scope of this documentation. If you need to relocate a classpath of your library/dependency, you'll likely know that you need to do so already. In that case, read on for information on how to specify relocation rules.

Within PySpigot's `config.yml`, you'll find a `library-relocations` value. You may specify your own relocations rules here. Relocation rules should be in the format `<path>|<relocated-path>`. For example, to relocate `com.apache.commons.lang3.stringutils` to `lib.com.apache.commons.lang3.stringutils`:

```yaml
...
# List of relocation rules for libraries in the libs folder. Format as <pattern>|<relocated pattern>
library-relocations:
  - 'com.apache.commons.lang3.stringutils|lib.com.apache.commons.lang3.stringutils'
...
```

An abbreviated list form is also permitted in YAML syntax:

```yaml
...
# List of relocation rules for libraries in the libs folder. Format as <pattern>|<relocated pattern>
library-relocations: ['com.apache.commons.lang3.stringutils|lib.com.apache.commons.lang3.stringutils']
...
```

To add multiple relocation rules in the abbreviated list form, separate each with a comma.

!> As stated previously, PySpigot creates a copy that ends with "-relocated" of every external JAR library. This file represents the relocated JAR, with all relocation rules applied to it. This copy is only created once, or, put differently, relocation rules are only applied once. In subsequent plugin loads, the relocated JAR library is loaded. Therefore, if changes are made to the relocation rules, it is necessary to delete the "-relocated.jar" copy of the relevant library/libraries so that the updated relocation rules are applied.