# woa that's wordy
I will come back and make this a little bit more digestible later. For now, the first draft of this article will get you where you need to go. Absolutely reach out to `thetimesweeper` with feedback on your experience here so it can be improved, thanks thanks.
# Introduction
Some background before we begin to let you understand what's going on. Feel free to skip to below for the actual steps

For those uninitiated about Thunderkit, you can head to the [disorganized mess of thunderkit articles](https://risk-of-thunder.github.io/R2Wiki/Mod-Creation/Thunderkit/Getting-Started/) on the modding wiki but I will summarize here.  
Thunderkit is primarily used to work with the game's code and your mod's code all in your unity project, and build the mod from there. This is different from the Henry setup, which is based on the ordinary way of writing a mod. In the Henry setup, most of the character setup is done in code (typically Visual Studio), and we just use the Unity Editor to build our assets that we then load with our code mod.

With all that said, we can take advantage of one part of Thunderkit: Importing the game code into our unity project to use with our assetbundles. 

## Why
With this, we can attach game components to our game objects and use them the way components were intended to be used: in the Unity editor. Projectiles are very component based, and were likely the reason you were linked to this article. Setting those up entirely through code is tedious to the point of limiting.

## Hold on, don't we already do this?
The unity-familiar among you might notice that we are already using RoR2 components in the Henry setup. The prefab in our unity project has the `ChildLocator` and `RagdollController` components attached.  
This was done by having dummy scripts in the unity project with the same names as the components from the game. When serialized, RoR2 will recognize these components as their own, and we're able to take advantage of assigning things in editor.

This works fine, but doesn't scale. We would have to do this for *every* script we want to attach in editor. The reason we are here is to import the whole library of game code into the project, and we will have access to *all* the game's components. 

# Shut Up and Let's Do It Already
ok but one more thing
## Step 0: Update Your Repo's .gitignore
you are using a repo... right?

In your unity project folder, you'll find a .gitignore. if you didn't know this was a thing, as the name implies, this outlines which files should be ignored by git when you upload. This typically includes the unity library and temp files which are constantly changing as you work on your project, and have no need to be pushed up.

The reason this is important, is that we are now going to import the game code into your unity project. That is property of the game devs, and uploading it to a repo would be redistributing game assets, which is a bit of a no-no. 

Add this to the .gitignore
```
#RoR2 Packages Folder
Packages/Risk of Rain 2
#ThunderKit root folder & Logs
ThunderKit/
Assets/ThunderKitSettings/Logs/
```

## Step 1: Install Thunderkit and RoR2ImportExtensions
0. Make sure you have [git](https://git-scm.com/) installed and added to your system path
    1. will probably have to restart your pc
    2. to test, open any command prompt and type `git help` 
1. In your unity project, on the top toolbar, go to Window > Package Manager
1. On the top left of the package manager, click the `+` and hit `Add package from git URL`
1. Input this URL for thunderkit: https://github.com/PassivePicasso/ThunderKit.git
1. Repeat steps 1-3 but with this URL for RoR2ImportExtensions https://github.com/risk-of-thunder/RoR2ImportExtensions.git

## Step 2: Fix My Sins
Scripts existing in the project will ruin the thunderkit import.

1. In the project window, navigate to the scripts folder
1. Delete the whole scripts folder
1. If you at some point added other scripts to this folder and would like to keep them, you will have to make sure they are wrapped in an assembly definition. 
    1. If you don't know how, reach out.

We no longer need the dummy RoR2 scripts. All your references will break, but we will fix it after ror2 is imported  
The helper scripts are very useful and we will add them back after.

## Step 3: Thunderkit Import

1. Go to the top toolbar Tools > Thunderkit > Settings
1. On the left hit Import Configuration and configure as follows: 

|Configuration | Set To | Why |
|---|-|---|
|Check Unity Version | On | |
|Disable Assembly Updater | On | |
|Post Processing Unity Package Installer | On |  |
|Assembly Publicizer | On | |
|MMHook Generator | Off | we're not building the mod from the editor so we don't need extra bloat |
|Import assemblies | On |  |
|Import Project Settings | On | adds things like physics layers for us to use |
|Set Deferred Shading | On |  |
|Create Game Package | On |  |
|Import Addressable Catalog | On | addressable browser is dope |
|Configure Addressable Graphics Settings | On |  |
|Ensure RoR2 Thunderstore Source | Off | not needed|
|Install BepInEx| Off | not needed, but if you want to use simplyaddress or something that depends on it, you can turn this on. |
|R2API Submodule Installer | Off | ah yes who wouldn't want 28 packages slowing down compiling, playing, and building? |
|Install RoR2 Compatible Unity Multiplayer HLAPI | Off | we're not weaving in editor so we don't need this |
|Install RoR2 Editor Kit | On | nebby do gud job |
|the rest | On |  |

3. On the left hit settings, browse for your ror2 executable, and hit import
3. let thunderkit do its thing
    1. in my experience, you may randomly get a file window. browse for ror2 exe again

you're well on your way! with this you will become very powerful

## Step 4: After Import
Now, if you already had a character started, take a look at your prefab and you will see the scripts are missing.
1. On the top toolbar, go to Window > Package Manager again
1. On the top left of the package manager, click the `+` and hit `Add package from git URL` again
1. Input this URL to install HenryEditorTools: https://github.com/TheTimeSweeper/HenryEditorTools.git
1. if all is well, go to the top toolbar, and find Tools > HenryTools > Update Existing Prefabs to RoR2 Scripts
    1. if you don't see this, something went wrong somewhere. 
1. take a look at your prefab again, the scripts should be back

This tool only accounts for the RoR2 scripts that were already in the henry template. If at some point you added more dummy ror2 scripts to use in editor, this won't fix those.

## Step 5: That's It!
You should make sure your asset bundle builds and still works.  
It should be no different to how you were already building the bundle and building the mod.