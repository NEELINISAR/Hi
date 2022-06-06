Support for android 5 and 6 was re-added to termux-app in beginning of 2022, but no support or package updates are planned.

Until termux-app 0.119 is released users can install an apt-android-5 artifact from github CI builds: https://github.com/termux/termux-app/actions

The updated, android-5 compatible app uses these repos:

```
deb https://packages.termux.dev/termux-main-21 stable main
deb https://termux.dev/science-packages-21-bin science stable
deb https://termux.dev/game-packages-21-bin games stable
deb https://termux.dev/termux-root-packages-21-bin root stable
```

while the old android-5 compatible app subscribed to:

```
deb https://termux.net stable main
deb https://dl.bintray.com/grimler/science-packages-21 science stable
deb https://dl.bintray.com/grimler/game-packages-21 games stable
deb https://dl.bintray.com/grimler/termux-root-packages-21 root stable
```

All of which are down now.
