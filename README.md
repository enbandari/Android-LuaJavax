[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)


# Android-Luajavax

* Lua 5.3.3
* LuaJava with some bug fixes.
* LuaJavax with Kotlin APIs.

# How to use

## Import

I have deployed these modules to maven central, you may add this in your build.gradle: 
 
```
implementation("com.bennyhuo:luajavax:1.0")
```

### SNAPSHOT

If you want to try the dev version, add this to your build.gradle:

```
repositories {
    maven {
        url "https://oss.sonatype.org/service/local/repositories/snapshots/" 
    }
}

dependencies {
    implementation("com.bennyhuo:luajavax:1.0-SNAPSHOT")
}
```

## API

You can create a Lua state machine via `LuaFactory`:

```kotlin
object LuaFactory {

    @JvmStatic
    fun createLua(context: Context): ILua = Lua(context)

}
```

The `ILua` instance is what you need to execute Lua scripts. See its APIs here: [ILua](luajavax/src/main/java/com/bennyhuo/luajavax/core/ILua.kt).

You can run Lua scripts from string/file/assets directly. It is easy to access Java class and objects.

```kotlin
LuaFactory.createLua(this@MainActivity).use { lua ->
    lua["Hello"] = Hello::class.java // set global value
    // run script, access to Java class
    lua.runText("hello = luajava.new(Hello); hello:a()")
    lua.runFile(File(getExternalFilesDir("lua"), "hello_world.lua"))
    lua.runScriptInAssets("hello_world.lua")
}
```

You can redirect stdout to any file you want:

```kotlin
lua.setOutput(File(getExternalFilesDir("lua_output"), "hello_world.lua.output")
```

The default behavior of Lua function `print` is to write content to the stdout, which does not work in Logcat. So you can use `redirectStdioToLogcat` to redirect `print` to Logcat with a Tag 'luajavax':

```kotlin
LuaFactory.createLua(this).use { lua ->
    // redirect 'print' and 'print_error' to logcat.
    // It take no effects on 'io.write' or 'io.stdout.write' or 'io.stderr.write'.
    lua.redirectStdioToLogcat()
    lua.runText("""
        print("see this in logcat info")
        print_error("see this in logcat warn")
        io.write("this won't work")
        io.stderr:write("this won't work")
    """.trimIndent())
}
```

The Lua state machine will be closed after use block. If you want to hold a global instance, you can:

```kotlin
object GlobalLua {

  val context: Context = ...

  val luaVm by  {
    LuaFactory.createLua(context)
  }

}

...

GlobalLua.luaVm.runFile(...)
```

# Link

- [AndroidLuaExample](https://github.com/haodynasty/AndroidLuaExample)
- [luajava](https://github.com/LuaDist/luajava/)

# Change Log

Bug Fix:

1. [Bug] Wrong class argument passed into jni function when calling javaNew. This will lead to that the 'new' method always returns nil when called from Lua on Java class.
2. [Bug] Falsely removes the Lua state object when closing.
3. [Bug] Error occurred when pass an expression of Java method call as an argument into another Java method call from Lua.
4. [Bug] Use objectIndex instead of classIndex for Java Classes.

Feature:

1. [Feature] The original error message was simply something like 'Not a valid Java Object.'. I have added the Lua script line info when error calling Java api in Lua.
2. [Feature] Add support to access fields and methods with the same name in Java object when calling Java api from Lua. Function in Lua is a first-class type so it is treated equally as other types like number or string, thus making keys in the Lua table must be different whatever types of the values are. However, This is not the case in Java since fields and methods are not the same thing so they can share an identical name. When accessing a key, we can figure out whether it is a function call or a value access from the calling stack to make this work.
3. [Feature] Support Java vararg method call from Lua.
4. [Feature] Support partially redirect stdio to Logcat.
