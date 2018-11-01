---
layout: post
title: "Opening files in NeoVim within iTerm2 from the macOS Finder"
date: 2018-11-01
---

I wanted a way to automatically open certain file types using the command-line version of [NeoVim](https://neovim.io) (`nvim`) on macOS, and to be able to open NeoVim within a terminal window using a Dock icon. I use [iTerm2](https://iterm2.com) as my terminal application.

Using Automator, it's possible to create an application with this functionality using an AppleScript. A similar process could be used to create an application launcher for other command-line programs, also allowing one or more file paths to be used as arguments.

1. Install [NeoVim](https://neovim.io) and [iTerm2](https://iterm2.com) if you haven't already.

2. Open Automator and create a new Application.

3. Add a `Run AppleScript` action and paste in the code below. If iTerm is already running it will create a new window, and it will optionally open one or more files in different NeoVim tabs. If you run the action within Automator it may give you an error but it should work when you create an Application from it.

   ```applescript
   on run {input, parameters}

   	# seem to need the full path at least in some cases
   	# -p opens files in separate tabs
   	set nvimCommand to "/usr/local/bin/nvim -p "

   	set filepaths to ""
   	if input is not {} then
   		repeat with currentFile in input
   			set filepaths to filepaths & quoted form of POSIX path of currentFile & " "
   		end repeat
   	end if

   	if application "iTerm" is running then
   		tell application "iTerm"
   			create window with default profile command nvimCommand & filepaths
   		end tell
   	else
   		tell application "iTerm"
   			tell current session of current window
   				write text nvimCommand & filepaths
   			end tell
   		end tell
   	end if

   end run
   ```

4. Save the Automator document as an Application in `/Applications`.

5. Open `/Applications` and drag your new application to the Dock if you wish.

6. Right-click a file of the type you want to automatically open in NeoVim (e.g. a Markdown (`.md`) file and click `Get Info`. Select `Other…` in the `Open with:` menu, navigate to your new application and click `Add`. Then click `Change All…` and all files with the same extension should now open in NeoVim.

If you have any questions or suggestions, why not get in touch with me on [Twitter](https://twitter.com/gregrs_uk)?
