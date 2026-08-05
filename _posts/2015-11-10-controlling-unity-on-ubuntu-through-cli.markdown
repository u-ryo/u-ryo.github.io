---
layout: post
title: "Controlling Unity on ubuntu through CLI"
date: "2015-11-10 00:29"
author: 'u-ryo'
categories: [Unity, ubuntu, Linux]
comments: true
published: true
---
### Clock on the menu bar
```
$ dconf write /com/canonical/indicator/datetime/show-date "true"
$ dconf write /com/canonical/indicator/datetime/show-day "true"
$ dconf write /com/canonical/indicator/datetime/show-seconds "true"
$ dconf write /com/canonical/indicator/datetime/show-year "true"
```

### battery on the menu bar
```
$ dconf write /com/canonical/indicator/power/show-time "true"
$ dconf write /com/canonical/indicator/power/show-percentage "true"
```

### "En" on the menu bar(default language selection)
```
$ dconf write /org/gnome/desktop/input-sources/current "uint32 1"
```

### Wallpaper
```
$ dconf write /org/gnome/desktop/background/picture-uri "'file:///usr/share/backgrounds/wallpaper_univcoop.jpg'"
```

### The way to suppress other icons on the launcher
```
$ dconf write /com/canonical/unity/launcher/favorites "['application://firefox.desktop']"
$ dconf read /com/canonical/unity/launcher/favorites
```

### forbidden hud
```
$ dconf write /org/compiz/integrated/show-hud "['disabled']"
```

### To suppress remote content search
```
$ dconf write /com/canonical/unity/lenses/remote-content-search "'none'"
```

### Supress auto-mount
```
$ dconf write /org/gnome/desktop/media-handling/automount "'false'"
$ dconf write /org/gnome/desktop/media-handling/automount-open "'false'"
```