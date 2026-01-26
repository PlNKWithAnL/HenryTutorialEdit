## Folder Structure

Once you have your built .dll file, you will need to arrange the folder for the mod in such a way that mod managers can import it correctly for testing.

Here's a quick visualizer for how it will look in your files.

 - Start in the folder named `HenryTutorial/build`.
   
 - Located next to the `plugins` folder, open up the manifest.json and change the "name" and "author" fields as you see fit.
 
 - Inside the `plugins` folder, you will need to place the mod's .dll file. Place your built asset bundle into the folder named `AssetBundles`, and your audio file in `Soundbanks`.

Your folder structure should now look like this:
```
/HenryTutorial/Build
|_ /Plugins
|   |_ /AssetBundles
|   | |_ HenryModAssetBundle
|   |_ /Soundbanks
|   | |_ HenryMod.sound
|   |_HenryMod.dll
|_manifest.json
```

Now that your your mod's folder matches the structure above, highlight everything in the `Build` folder and zip it up. Once you have the .zip file of your completed mod, manually import it into your mod manager to test that it works correctly. 

 - In r2modman, this is done by going to `Settings -> Profile -> Import Local Mod.` 
 - In Gale, click the `Import` dropdown from the top menu navigation bar, and select `...Local Mod`

Navigate to the file location for your mod's .zip file, select it, and confirm.

Depending on your mod manager, you may need to download the mod's dependencies manually. These are listed in the mod's `manifest.json` file. Make sure to complete any updates the mods may need after downloading dependencies.

After importing the zip into your modmanager, this should be the resulting folder structure.

![The correct folder structure after import.](https://github.com/PlNKWithAnL/HenryTutorialEdit/blob/master/%7B86ED79BF-541C-4938-82CF-6A572EA68F34%7D.png?raw=true "The correct folder structure after import.")

With the mod successfully imported with the correct file structure, boot the game to test that all your dependencies are present, and that the mod functions as intended.

    The henry template will load your assetbundle from a folder next to your .dll named "AssetBundles". 
    If you change this location in your mod's code, make sure the change is reflected in the file structure of your mod.
    If you're doing language files, you'll need a folder next to your .dll named "Languages".
    Soundbanks aren't required to be in a folder, so long as they are named with the .sound extension R2API will load them, but it's just nice for organization.

### Final warning about conflicts
DO NOT upload a mod with the default henry assetbundle, as this will cause conflicts. Simply renaming the file will not suffice. Follow the tutorial to make sure your assetbundle has been rebuilt from unity with a different tag, or we will Thanos snap your thunderstore upload.


## A Fancier Method

Once upon a time, you used to just build the .dll, zip it up, import it, and run the game, as shown above.
Simple times for simple men. We're big brain now.

## Using a Post Build
When you build a mod, you can set your visual studio .csproj to automatically run some commands after you build. We can use this to automatically copy our built mod into your r2modman profile folder, so you can run the game with your mod

- In the Visual Studio Solution Explorer, right click your .csproj and hit ![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/0ad55db6-5ea5-4db8-9f55-3f2e983eee2b)
- On the left, go to Build Events.
- You'll see that the HenryMod project already has some post-build commands I was using as I developed the tutorial.
  - You can go [here](https://risk-of-thunder.github.io/R2Wiki/Mod-Creation/IDE/Visual-Studio/Post-Build-Events/) for a much more in depth guide on this. For now I'll just go through what we have here.

Username:
```batch
REM follow the Building Your Mod page on the henrytutorial wiki for more information on this
REM change this to your username (or add yours if you're working in a team or somethin)
if "$(Username)" == "Erikbir" set build=true
```  
- set this to your PC username. If your repo is public careful not to doxx yourself

Copy your build
```batch
REM copy the built mod to our Build folder
copy "$(TargetPath)" "$(ProjectDir)..\Build\plugins"
```  
- The weird "$()" stuff are macros. TargetPath is the path to your built dll, and ProjectDir is the directory of your project, clearly.
- Here we're stepping up one folder, to go to the HenryTutorial\Build\plugins folder, and copying our built .dll there.

Copy your AssetBundle
```batch
REM copy the assetbundle from our unity project to our Build folder
REM change these paths to your (now hopefully renamed) folders
if exist "$(ProjectDir)..\HenryUnityProject\AssetBundles\myassetbundle" (
copy "$(ProjectDir)..\HenryUnityProject\AssetBundles\myassetbundle" "$(ProjectDir)..\Build\plugins\AssetBundles"
)
```  
- as the comments say, we also copy the assetbundle from the unity project to the build folder, so we don't have to manually drag it to our mod install every time we make a change in unity
- be sure to change the unity project folder and assetbundle names to what you renamed them
  - you did rename them, right?

Copy your AssetBundle
```batch
REM copy the whole Build\plugins folder into your r2modman profile. This mimics how r2modman will install your mod
Xcopy /E /I /Y "$(ProjectDir)..\Build\plugins" "E:\r2Profiles\Blinx Returns\BepInEx\plugins\rob-henrymod\"
```
- now with a magical function called Xcopy, it will copy everything in your `plugins` folder to the your r2modman profile folder
  - this mimics how r2modman will install your mod when it's downloaded from thunderstore.
- find your r2modman profile folder by going to r2modman settings > locations > browse profile folder
- of course, change "rob-henrymod" to "author-name" according to the fields in your manifest.
  - if you have followed the "Installing with r2modman" section above, a folder with this name should already exist that you can verify
    - if it doesn't, the Xcopy command will create a folder at the specified path anyway, so just be mindful of that

## Getting Ready for Upload 

When you have completed testing and are ready to upload your mod to Thunderstore, follow the instructions [HERE](https://thunderstore.io/package/create/docs/) for info on the required files.

Place the mod's icon, readme, and changelog in the `/HenryTutorial/Build` folder right next to the `plugins` folder and the `manifest.json` file as shown below:
```
/HenryTutorial/Build
|_ /Plugins
|   |_ /AssetBundles
|   | |_ HenryModAssetBundle
|   |_ /Soundbanks
|   | |_ HenryMod.sound
|   |_ HenryMod.dll
|_ manifest.json
|_ readme.md
|_ icon.png
|_ changelog.md
```
Zip the contents of the build folder up, and upload.
___
happy building! any questions go and bother `thetimesweeper`
