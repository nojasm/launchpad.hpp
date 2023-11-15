# launchpad.hpp
A simple, RtMidi-Based, header only C++ library for interacting with the Novation Launchpad (Tested on mk2 only). Also contains general reverse engineered information about the Launchpad.

# Usage
Include and compile with RtMidi.cpp and RtMidi.h.

## Examples
```C++
// Initialize launchpad and pass in port number
Launchpad* lp = new Launchpad(1);

// Reset all lights
lp->resetGridLights();
lp->resetTopLights();
lp->resetRowLights();

// Push all light values to the launchpad
lp->updateLights(true);

// Set the bottom light on the right row
lp->setRowLight(0, 95);

// Push changed light values to the launchpad
lp->updateLights();

// Some kind of example main loop
while (true) {
    // Let's light up pressed buttons on the 8x8 grid
    // Go through every new event
    while (!lp->gridQueue.empty()) {
        // Get event from event queue
        LaunchpadGridButtonState gbs = lp->gridQueue.at(lp->gridQueue.size() - 1);
        
        // Light up or turn of grid cell
        lp->setGridLight(gbs.x, gbs.y, gbs.pressed ? 50 : 0);

        // Remove event from event queue
        lp->gridQueue.pop_back();
    }
}
```

## Finding the correct port number
You can use tools like `amidi --dump -l` or a well-proven system of trial-and-error to find the corrent port number to pass into the Launchpad constructor.

## Function overview
```C++
// An overview of important (but not all) methods and attributes
class Launchpad {
    vector<LaunchpadGridButtonState> gridQueue;
    vector<LaunchpadTopButtonState> topQueue;
    vector<LaunchpadRowButtonState> rowQueue;

    Launchpad(int portNumber);
    void setGridLight(int x, int y, unsigned char color);
    void setRowLight(int rowIndex, unsigned char color);
    void setTopLight(LaunchpadTopButton btn, unsigned char color);
    
    void resetGridLights();
    void resetRowLights();
    void resetTopLights();
    
    void updateLights(bool updateAll = false);
}
```

# Button Mappings
There are three button categories: Grid-Buttons, Row-Buttons and Top-Buttons.

**Grid Buttons** are identified by their `x` and `y` coordinates (0-7). The origin `0/0` is at the bottom left grid cell of the launchpad. By default, when reading raw midi signals, the origin is at the top left grid cell. This can be easily changed in the updateLights() and the launchpadEventHandler() functions.

**Row Buttons** are identified by their index `rowIndex`. The origin `0` is at the bottom.

**Top Buttons** are identified by their enumeration type `LaunchpadTopButton`. Valid values are `UP, DOWN, LEFT, RIGHT, SESSION, USER1, USER2 and MIXER`. Using type casting, you can also use integer values from 0 to 7.

# Light Outputting
## Light Caching
Instead of pushing every light event to the launchpad directly, they are cached and only pushed when launchpad->updateLights() is called. This function only pushes light values that are changed. If you want to update all values, such as for initializing the lights, you can pass `true` as a parameter to the updateLights() function.

## Light values
Light values are unsigned char values between 0 and 127 inclusive. To find out which integers map to which RGB-combination, you can take a look at [Kaskobi's Novation Palette](https://www.kaskobi.com/palettes), or at (https://github.com/nojasm/rgb2launchpadcolor).

Here is a shortened list containing the world's most famous colors:
| Color | RGB | Velocity Value |
| - | - | - |
| **OFF** | 0, 0, 0 | 0 |
| White | 1, 1, 1 | 3 |
| Red | 1, 0, 0 | 5 |
| Green | 0, 1, 0 | 21 |
| Blue | 0, 0, 1 | 45 |

# Advanced Stuff
The following things are reverse engineered and you probably only need them to develop your own Launchpad midi library.

## Raw MIDI Signals
Every midi signal/event is composed of three numbers from 0 to 127 *(CHECK)*:
```
[127, 127, 127]
```

The first number is the type of the event. When pressing buttons on the launchpad, this number is `144` for grid and row buttons and `176` for top buttons. The second number can be seen as a button identifier. The third number is the velocity. When reading events, it is either `0` or `127` if pressed or released. When writing events it can be set to any value for different colors. See [Light Values](#light-values)

TODO: Add what numbers relate to grid and row buttons


# Resources
- RtMidi docs
- https://www.kaskobi.com/palettes