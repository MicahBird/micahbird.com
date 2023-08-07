+++
title = "Switching from Linux to macOS and creating a bastardized OS in the process"
author = "Micah Bird"
date = "2023-08-01"
categories = [
    "Linux",
    "macOS"
]
image = "cover.jpg"
+++

## Why..?

In my transition to going to university, I came to the realization that I will have to forgo my desktop for something that is portable **and** powerful. I spent many weeks trying to find the perfect "gamer" laptop that would fit my compute need (it felt a lot like those "pick two" triangles between performance, portability, and battery life) but I could not find anything that could compare to battery life and performance of the Apple Silicon lineup.

## Dealing with the `defaults`

MacOS's default settings definitely force you to "think different" however, you can change all settings with the `defaults` command which under the hood writes to Apple's `.plist` files. This can be especially useful for automating your macOS setup!

Here are some commands that work as of macOS 13 Ventura:
```bash
# Enable Reduce Motion
defaults write com.apple.Accessibility ReduceMotionEnabled -int 1

# Show Airplay in menu bar when active
defaults write com.apple.airplay showInMenuBarIfPresent -int 1

# Save to disk (not to iCloud) by default
defaults write NSGlobalDomain NSDocumentSaveNewDocumentsToCloud -bool false

# Save screenshots to Pictures/Screenshots folder.
mkdir -p "${HOME}/Pictures/Screenshots"
defaults write com.apple.screencapture location -string "${HOME}/Pictures/Screenshots"

# # Set a blazingly fast keyboard repeat rate, and make it happen more quickly.
# # (The KeyRepeat option requires logging out and back in to take effect.)
defaults write NSGlobalDomain InitialKeyRepeat -int 20
defaults write NSGlobalDomain KeyRepeat -int 1
```

Although I've learned the hard way that all defaults commands are not equal, and can change with each macOS update. So, if your copy/pasting a `defaults` command be sure that your not adding unnecessary clutter to your entries!

Also, if you don't know what the specific `defaults` command would be for an option that you would like to change, simply follow these steps:

   1. `cd /tmp`
   2. Store current defaults in file: `defaults read > before`
   3. Make a change to your system.
   4. Store new defaults in file: `defaults read > after`
   5. Diff the files: `diff before after`

## The positives of macOS!

### Loving Homebrew

When I was first introduced to `brew` I was put off by the idea of using Ruby for package management, but it has slowly but surely grown on me! If you are used to `apt`, `brew` will make you feel right at home! 

### Yay CMUS! Yay!

CMUS is one of my favorite music players for Linux systems, and to my surprise, it has native macOS support! It's a simple `brew install cmus` away!

Pro-tip: If you are experiencing odd audio crackling when skipping tracks or during playback, try running the command `set audio=ao` to see if it fixes it! The only downside to this option is when switching audio devices (e.g. headphones to builtin speakers) you must stop playback with `v` and then play the track again!

### GUI Linux Applications on macOS

While I will probably flesh this out into a full blown guide later down the line, for those that are technically inclined, it's possible to run normal Linux GNOME, Qt, and other GUI apps almost like they are native through [XQuartz](https://www.xquartz.org/) and [Lima](https://github.com/lima-vm/lima)!

---

But, it wasn't all sunshine and rainbows, in my opinion, there is a lot to be desired coming from the KDE desktop environment and the Linux realm.

## Restoring 'basic functionality'

### External Mouse Scrolling Direction

Something that you will notice almost immediately when using an external mouse is that it uses natural scrolling directions (or the same scrolling direction as the trackpad). While you can disable natural scrolling to fix the issue, I prefer to keep natural scrolling enabled on the trackpad, but normal scrolling on the mouse. Sadly there is no native way to accomplish this, [but you can install an open source app to fix it!](https://github.com/pilotmoon/Scroll-Reverser/releases) You can also install Scroll Reverser via `brew` with the following command:

`brew install --cask scroll-reverser`

### Disabling Bluetooth When Sleeping

Something you may also notice if you don't have AirPods (like myself) is that Bluetooth headphones stay connected to your computer even when asleep. To quote NakeyJakey:
> ["I don't want that."](https://youtu.be/8vfbVVkwdQw?t=113)

Luckily there is another open source app that can solve that! Simply install [Bluesnooze](https://github.com/odlp/bluesnooze/) with the command `brew install bluesnooze`.

### An Actual Alt-Tab

While macOS does have a builtin application switcher (triggered with `command + tab`), it leaves a lot to be desired because it only allows you switch applications, not windows. Fear not! There is yet another open source app called [AltTab])(https://alt-tab-macos.netlify.app) that solves exactly this issue!

[![AltTab Screenshot](https://d33wubrfki0l68.cloudfront.net/23fa1c980411cdc7c2967d25b132497fa9f596d4/e533a/public/demo/frontpage.jpg)](https://alt-tab-macos.netlify.app)

---

But now, onto the fun stuff...

## Creating a tiling window manager experience with `yabai` + `skhd`

While there are many guides that cover this setup in detail, if you are getting started, I would highly recommend following along with this tutorial from Josean Martinez.

{{< youtube k94qImbFKWE >}}

This is where a lot of the OS bastardization comes from, as once you got a basic setup going there is an infinite amount of ways to customize your setup, and make it foreign from macOS. In my experience, I have found the following configuration options particularly useful:

##### `~/.config/skhd/skhdrc`
```bash
# Displaying a notification when restating Yabai
ctrl + alt - r : yabai --restart-service && osascript -e 'display notification "Yabai has been restarted." with title "Yabai"'

# Open a new Kitty terminal window without starting another instance of the app:
alt - a : osascript -e 'tell application "System Events" to tell process "kitty" to click menu item "New OS Window" of menu "Shell" of menu bar 1'

# Open a new Firefox window without starting another instance of the app:
alt - w : osascript -e 'tell application "System Events" to tell process "Firefox" to click menu item "New Window" of menu "File" of menu bar 1'

# Open a new terminal instance
alt - return : open -n /Applications/kitty.app

# Put computer to sleep as a shortcut
cmd - escape : osascript -e 'tell app "System Events" to sleep'

# Balance out tree of windows (resize to occupy same area)
shift + alt - e : yabai -m space --balance

# Move window to space # and focus it without disabling SIP
shift + alt - 1 : yabai -m window --space 1; index=1; eval "$(yabai -m query --spaces | jq --argjson index "${index}" -r '(.[] | select(.index == $index).windows[0]) as $wid | if $wid then "yabai -m window --focus \"" + ($wid | tostring) + "\"" else "skhd --key \"ctrl - " + (map(select(."native-fullscreen" == 0)) | index(map(select(.index == $index))) + 1 % 10 | tostring) + "\"" end')"
```
##### `~/.config/yabai/yabairc`
```bash
# Focus window after another window is closed rather than losing focus
yabai -m signal --add event=window_destroyed active=yes action="yabai -m query --windows --window &> /dev/null || yabai -m window --focus mouse &> /dev/null || yabai -m window --focus \$(yabai -m query --windows --space | jq .[0].id) &> /dev/null"

# Disable management for certain apps
yabai -m rule --add app="System Information" manage=off
yabai -m rule --add app="^System Settings$" manage=off
yabai -m rule --add app="^Raycast$" manage=off
```

## Conclusion

There are many more open sources tools that I didn't go into in this post, such as [Hammerspoon](https://hammerspoon.org/) for creating more advanced automation written with Lua, or even [Sketchybar](https://github.com/FelixKratz/SketchyBar) for creating your own custom menu bar! I'd highly recommend taking a gander at those tools, and maybe someday I'll make a part 2 two this post, time will tell!
