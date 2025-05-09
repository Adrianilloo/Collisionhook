# CollisionHook

## Original preface from VoiDED
https://forums.alliedmods.net/showthread.php?t=197815

This extension provides a straightforward and easy way to hook and modify collision rules between entities.

You might say "but SDKHooks provides this functionality already!", and you would be partly right. However, I wasn't happy with how SDKHooks' `ShouldCollide` works, and here's why:

If you're familiar with SDKHooks, this code won't be new:


```c++
// Somewhere in your code
SDKHook(someEntity, SDKHook_ShouldCollide, ShouldCollide);

bool ShouldCollide(int entity, int collisionGroup, int contentsMask, bool originalResult)
{
	// Modify collision rules
}
```

As you can see, the hook doesn't provide much information about exactly which entities are colliding; only the collision group and contents mask of the other entity are given to the plugin developer.

To solve this limitation, I wrote **CollisionHook**.

**CollisionHook** provides 2 forwards to plugins, both of which can modify the collision rules of certain types of collisions.

`forward Action CH_ShouldCollide(int ent1, int ent2, bool &result);`

`CH_ShouldCollide` is called when **VPhysics** is determining whether two **VPhysics** objects (and their associated entities) should collide. Examples of these kinds of collisions include projectiles colliding with the world entity.

`forward Action CH_PassFilter(int ent1, int ent2, bool &result);`

`CH_PassFilter` is called when the game is performing a ray/hull trace and is making use of an entity filter. Examples of these kinds of collisions include player-player collisions.

- To modify the behavior of the collision check, set `result` to `true` if you wish the entities to collide, otherwise `false`. Then, return `Plugin_Handled`, `Plugin_Changed` or `Plugin_Stop` in the forward. The extension will skip the game's default collision rules and your `result` value will be returned for the ongoing collision-related call.
- If you want the game to use default collision rules, simply return `Plugin_Continue` in the forward.

The benefits of both of these forwards is that you can strictly control which entities can collide, instead of just a specific type of entity.

## Installation
Grab the latest release from **Releases** page. Drag & Drop.

## Building
Prepare a SM extensions environment, following these common [requirements](https://wiki.alliedmods.net/Building_SourceMod#Requirements).

Next, create the following symbolic link to SourceMod's **SafetyHook** component, in this project's root directory:

- On Linux: `ln -s <SM directory>/public/safetyhook`
- On Windows: `mklink /D safetyhook <SM directory>/public/safetyhook`

Then, enter the following commands in the same directory:

```
mkdir build
cd build
```

Now, it's time to effectively compile the extension. If you're building for Windows, you need to open **Visual Studio 2022 Developer Command Prompt** to continue with the remaining steps.

For Windows, execute:

```
python ..\configure.py --hl2sdk-root=<HL2SDKs parent directory> --mms-path=<MM:S directory> --sm-path=<SM directory> -s present --enable-optimize --targets x86,x64
```

For Linux, type in a normal shell:

```
python ../configure.py --hl2sdk-root=<HL2SDKs parent directory> --mms-path=<MM:S directory> --sm-path=<SM directory> -s present --enable-optimize --targets x86,x64
```

(To force Clang, prepend `CC=clang CXX=clang++` to the above call).

Finally, just enter: `ambuild` (on any OS).

Output files should be created at `package` folder under active `build` directory, with the same hierarchy than classic `addons` server folder for SourceMod.

Not tried on Mac OS X.

### Current Build supports CSGO Linux

## Limitations / Known Issues / Notes
- Some collisions are not handled by this extension, those include collisions that are checked with traces that don't use an entity filter. Many of these can be handled by SDKHooks' `Touch` hook.
- Modifying collisions that are also performed on the client will lead to prediction errors. This is no different from other server-only methods of overriding collisions.
- This extension should work for any OB-based mod (TF2 CS:S, DoD:S), but it's only been tested on TF2.

### Thanks
- AzuiSleet, for suggesting `PassServerEntityFilter` for trace based collisions
- psychonic, for providing the Linux build, Makefile, and generally being the most helpful person on the planet
