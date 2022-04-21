Support for android 5 and 6 was dropped on 2020-01-01.  The android-5 compatible apks are available from https://archive.org/details/termux-repositories-legacy as well as [F-Droid's archive repository](https://forum.f-droid.org/t/archive-repositories/10556).

All the repositories that was used for the android-5/6 version of the app have ceased to exist for different reasons since support was dropped.  To be able to use apt users need to change the repo urls in $PREFIX/etc/apt/sources.list and $PREFIX/etc/apt/sources.list.d/*.list. 

The old URLs for all the repos were:

```
deb https://termux.net stable main
deb https://dl.bintray.com/grimler/science-packages-21 science stable
deb https://dl.bintray.com/grimler/game-packages-21 games stable
deb https://dl.bintray.com/grimler/termux-root-packages-21 root stable
```

and these can be changed to:

```
deb https://packages.termux.org/termux-main-21 stable main
deb https://termux.org/science-packages-21-bin science stable
deb https://termux.org/game-packages-21-bin games stable
deb https://termux.org/termux-root-packages-21-bin root stable
```

after which termux on android-5/6 should be fully working (but with old package versions that will not receive any updates).
