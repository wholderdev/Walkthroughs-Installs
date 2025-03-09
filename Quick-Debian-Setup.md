# Debian install setup

Quick instructions I'll build on for setting up my environment

## Adding terminal shortcut

	Add Ctrl + Alt + t shortcut for gnome-terminal

	Windows key, type “shortcuts”, select the shortcuts option

	Scroll to bottom, View and Customize Shortcuts, Custom Shortcuts

	Add a new shortcut, name it whatever you want like “Terminal”

	Type “gnome-terminal” into the command field

	Type ctrl + alt + t after selecting the shortcut option.

## Change themes to dark in OS

	Go to settings, Appearance, and change to the dark theme

	Select a background you like if you want

	Change theme and color scheme to gnome-terminal to dark and solar

	Open a terminal (ctrl + alt + t if you set the shortcut)

	Select the triple bars button next to the X (top right probably)

	Select preferences

	Change theme variant to Dark

	Select the unnamed profile, select colors, change Built-in schemes to solar (or whatever you want).

	Change the font size or anything else you want

## Setting up your environment

Give sudo access to user

```
# Make sure to use su - and not su root, as the latter doesn't provide us the usermod binary
su -
usermod -aG sudo myUser
exit
```

Log out then log back in
