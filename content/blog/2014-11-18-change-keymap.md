---
layout: post
title: Changing keymap on ubuntu
description: explain how to switch ESC and CAPSLOCK.
category: blog
tags: [Ubuntu, Ubuntu16.04, keymap]
english: true
---

The tools to manipulate the keyboard layout on the virtual consoles are loadkeys, dumpkeys and showkeys. Read their manpages and inform yourself about their intricacies.

Note that these tools only work in a virtual console, not in a terminal emulator in X.

# Save your current keyboard layout:

```
$ dumpkeys > backup.kmap
```

In case something goes wrong you might be able restore your keymap using the command:

```
$ sudo loadkeys backup.kmap
```

If the keyboard is so messed up that you can't even do this then your only option not involving ancient kernel magic is to reboot.

# Check which keycodes are assigned to your keys:

```
$ showkey
```

Now press the ESC key and the CAPSLOCK key. The keycodes should show up on the screen. Note the keycodes. On my system the ESC has the keycode 1 and CAPSLOCK has the keycode 58. showkey will terminate after 10 seconds of inactivity.

Note the names of the ESC and CAPSLOCK keys from dumpkeys:

```
$ dumpkeys | grep 1
...
keycode   1 = Escape
...
$ dumpkeys | grep 58
...
keycode  58 = CtrlL_Lock
...
```

Note the keymap line from dumpkeys:

```
$ dumpkeys | head -1
keymaps 0-127
```

# Create a keymap file which switches ESC and CAPSLOCK:

```
keymaps 0-127
keycode   1 = CtrlL_Lock
keycode  58 = Escape
```

# Load the keymap:

```
$ sudo loadkeys swap_esc_capslock.kmap
```
