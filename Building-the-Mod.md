## Once upon a time
You used to just built a dll and put it in your plugins folder and run the game.  
Simple times for simple men. We're big brain now.

## Folder Structure
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/14e6461f-e0fa-4dfd-a9bf-79e1983c61cc)

Here's the folder structure that's needed from your mod as it should be installed in your r2modman profile.  
- The henry template will load your assetbundle from a folder next to your .dll named "AssetBundles".
- If you're doing language files (see the [Generating Language files(under construction)]() page for details), you'll need a folder next to your .dll named "Languages".
- Soundbanks aren't required to be in a folder, so long as they are named with the .sound extension R2API will load them, but it's just nice for organization.

## Installing with r2modman
Take a gander at how we make this possible in the HenryTutorial\Build folder:  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/c2384457-2249-4179-b63c-2cc4e626c01b)

- Open up the manifest.json and change the "name" and "author" fields as you see fit. 
- Now zip this file and the plugins folder and you can import this into r2modman using settings > profile > import local mod. 
- This mimics how r2modman will download and instal your mod from thunderstore when you upload it

Of course, when you go to release your mod, as you (should) know, you'll need to add an `icon.png` and `readme.md`.

## Using a Post Build
When you build a mod, you can set your visual studio .csproj to automatically run some commands after you build. We can use this to automatically copy our built mod into your r2modman profile folder, so you can run the game with your mod

- In the Visual Studio Solution Explorer, right click your .csproj and hit ![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/0ad55db6-5ea5-4db8-9f55-3f2e983eee2b)
- On the left, go to Build Events.
- You'll see that the HenryMod project already has some post-build commands I was using as I developed the tutorial

```batch
REM follow the Building Your Mod page on the henrytutorial wiki for more information on this
REM change this to your username (or add yours if you're working in a team or somethin)
if "$(Username)" == "Erikbir" set build=true

if defined build (

REM copy the built mod to our Build folder
copy "$(TargetPath)" "$(ProjectDir)..\Build\plugins"

REM copy the assetbundle from our unity project to our Build folder
REM change these paths to your (now hopefully renamed) folders
if exist "$(ProjectDir)..\HenryUnityProject\AssetBundles\myassetbundle" (
copy "$(ProjectDir)..\HenryUnityProject\AssetBundles\myassetbundle" "$(ProjectDir)..\Build\plugins\AssetBundles"
)

REM copy the whole Build\plugins folder into your r2modman profile. This mimics how r2modman will install your mod
Xcopy /E /I /Y "$(ProjectDir)..\Build\plugins" "E:\r2Profiles\Blinx Returns\BepInEx\plugins\rob-henrymod\"
)
```