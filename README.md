## hycov
hyprland overview mode plugin,a new tile window workflow  

### what can it do:
- Tile all workspace windows in one workspace by grid layout

- After quit the overview mode, it can perfect recovery window state (fullscreen, floating, the size, position)

- Support a variety of trigger methods, touch pad gestures, mouse screen hot area, shortcut keys

- Support for multiple monitors

- Supports a variety of focus switching modes, direction switching, single shortcut key cyclic switching

https://github.com/DreamMaoMao/hycov/assets/30348075/59121362-21a8-4143-be95-72ce79ee8e95


Anyone is welcome to fork. If you end up improving the plugin, please let me know, and I'll be happy to use your fork.

### Manual Installation

##### something need to notice
1.You need to manually relogin after it is installed. 
after the installation is complete, it may also trigger logout if you do not logout which is normal.

2.Only supports hyprland source code after 2023-10-22.
because the plugin requires this [commit](https://github.com/hyprwm/Hyprland/commit/a61eb7694df25a75f45502ed64b1536fda370c1d) in hyprland(was commited in 2023-10-21)

##### using meson and ninja:

```shell
git clone https://github.com/DreamMaoMao/hycov.git
cd hycov
sudo meson setup build --prefix=/usr
sudo ninja -C build
sudo ninja -C build install # `libhycov.so` path: /usr/lib/libhycov.so
```

##### using cmake:

```shell
git clone https://github.com/DreamMaoMao/hycov.git
cd hycov
bash install.sh # `libhycov.so` path: /usr/lib/libhycov.so
```

### Usage (hyprland.conf)

```conf
# when enter overview, you can use left-button to jump, right-button to kill or use keybind
plugin = /usr/lib/libhycov.so

# bind key to toggle overview(normal mode)
bind = ALT,tab,hycov:toggleoverview
# bind key to toggle overview(force mode,no affected by `only_active_workspace` and `only_active_monitor`)
bind = ALT,grave,hycov:toggleoverview,forceall #grave key is the '~' key

# The direction switch shortcut key binding.
# calculate the window closest to the direction to switch focus.
# This keybind is applicable not only to the overview  but also to the general layout
bind=ALT,left,hycov:movefocus,l
bind=ALT,right,hycov:movefocus,r
bind=ALT,up,hycov:movefocus,u
bind=ALT,down,hycov:movefocus,d

plugin {
    hycov {
        overview_gappo = 60 # gaps width from screen edge
        overview_gappi = 24 # gaps width from clients
        hotarea_size = 10 # hotarea size in bottom left,10x10
        enable_hotarea = 1 # enable mouse cursor hotarea     
        swipe_fingers = 4 # finger number of gesture,move any directory
        move_focus_distance = 100 # distance for movefocus,only can use 3 finger to move 
        enable_gesture = 0 # enable gesture
        disable_workspace_change = 0 # disable workspace change when in overview mode
        disable_spawn = 0 # disable bind exec when in overview mode
        auto_exit = 1 # enable auto exit when no client in overview
        auto_fullscreen = 0 # auto make active window maximize after exit overview
        only_active_workspace = 0 # only overview the active workspace
        only_active_monitor = 0 # only overview the active monitor
        enable_alt_release_exit = 0 # alt swith mode arg,see readme for detail
        alt_replace_key = Alt_L # alt swith mode arg,see readme for detail
        alt_toggle_auto_next = 0 # auto focus next window when toggle overview in alt swith mode
        click_in_cursor = 1 # target windwo is find by cursor, not the current foucus window.
        hight_of_titlebar = 0 # height deviation of title bar hight

    }
}

```

# suggested additional configuration
- when `auto_fullscreen=1` is set, you can also set the border color to mark the maximize state and bind key to control fullscreen maximize state.
```
windowrulev2 = bordercolor rgb(158833),fullscreen:1 # set bordercolor to green if window is fullscreen maximize
# toggle fullscreen maximize
bind = ALT,a,fullscreen,1
```

https://github.com/DreamMaoMao/hycov/assets/30348075/15ba36c2-1782-4ae0-8ac1-d0ca98e01e0f

- if you use the `hyprland/workspaces` module in waybar,you should change field {id} to {name}. It will let you know you are in overview mode.
```
"hyprland/workspaces": {
    "format": "{name}",
    "on-click":"activate",
},
```
![image](https://github.com/DreamMaoMao/hycov/assets/30348075/332f4025-20c1-4a44-853b-1b5264df986e)
![image](https://github.com/DreamMaoMao/hycov/assets/30348075/500d9fd7-299b-48bc-ab72-146f263044a5)


# Alt switch mode
```conf
enable_alt_release_exit = 1
alt_replace_key = Alt_L # If your mainkey of toggleoverview is ALt, you can ignore it
alt_toggle_auto_next = 0 # auto focus next window when enter overview in alt mode
```
## operation
such as `alt + tab`:

- 1.`alt + tab` will enter overview when you not in overview(please hold alt,don't make it release)

- 2.`alt + tab` will switch window focus circularly when you in overview. (please hold alt,don't make it release)

- 3.when you release `alt` , it will auto exit overview.


## If your mainkey that triggers toggleoverview is not `alt`, you need to use `alt_replace_key` to specify what is the detection key on release.

- use keyname
```conf
alt_replace_key = Alt_L # Alt_L,Alt_R,Super_L,Super_R,Control_L,Control_R,Shift_L,Shift_R
```

- use keycode
```conf
alt_replace_key = code:64 # use `xev` command to get keycode
```

### NixOS with home—manager

```nix
# flake.nix

{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };

    hyprland.url = "github:hyprwm/Hyprland";

    hycov={
      url = "github:DreamMaoMao/hycov";
      inputs.hyprland.follows = "hyprland";
    };
  };

  outputs = { nixpkgs, home-manager, hyprland, hycov, ... }:
    let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in
    {
      homeConfigurations."user@hostname" = home-manager.lib.homeManagerConfiguration {
        pkgs = nixpkgs.legacyPackages.x86_64-linux;

        modules = [
          hyprland.homeManagerModules.default
          {
            wayland.windowManager.hyprland = {
              enable = true;
              plugins = [
                hycov.packages.${pkgs.system}.hycov
              ];
              extraConfig = ''
                bind = ALT,tab,hycov:toggleoverview
                bind=ALT,left,hycov:movefocus,l
                bind=ALT,right,hycov:movefocus,r
                bind=ALT,up,hycov:movefocus,u
                bind=ALT,down,hycov:movefocus,d

                plugin {
                    hycov {
                      overview_gappo = 60 #gaps width from screen
                      overview_gappi = 24 #gaps width from clients
                	    hotarea_size = 10 #hotarea size in bottom left,10x10
                	    enable_hotarea = 1 # enable mouse cursor hotarea
                    }
                }
              '' + ''
                # your othor config
              '';
            };
          }
          # ...
        ];
      };
    };
}
```
## Frequently Asked Questions
- some config not work, or the plugin not work.
```
if you use install hycov at first time,please try logout and relogin again.
```

- The numbers on the waybar are confused

```
1.Please pull the latest waybar source code compilation,
this issue has been fixed in the waybar project, fix date (2023-10-27)

2.Change the {id} field in hyprland/workspace field to {name}
```

- Compilation failure
```
Please pull the latest hyprland source code to compile and install. The plugin relies on a hyprland pr,pr submission date (2023-10-21)
```

- Unable to load
```
Check whether hyprland has been updated, and if so, please recompile hyprcov
```
