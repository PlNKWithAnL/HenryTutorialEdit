# Overview
This tutorial will go over custom character creation for Risk of Rain 2. It assumes a basic understanding of Unity and C# at the very least. If you don't have that and get stuck, look up tutorials! I can't teach you everything here.

# Getting Started
Creating a character is a daunting task. Between modeling, animation, coding, skill icons, sounds, VFX and more there's a whole lot that goes into it. Don't expect it to be an easy process.

The first step would be cloning this repo and opening up `HenryMod.sln` in `Visual Studio`.

The recommended starting point is skill creation. [This](https://risk-of-thunder.github.io/R2Wiki/Mod-Creation/Assets/Skills/) tutorial is a good place to start. Keep in mind, step 1 of that tutorial goes over adding the skill to a body, which is already streamlined with Henry. As you follow the skills tutorial, step 2 and onwards is what will transfer over here.

Creating skills is a pretty specific process that involves a lot of hard coding that greatly differs depending on what you want the skill to do, so there's not a whole lot that can be explained here. 3 basic types of attacks, OverlapAttacks (melee), BulletAttacks (hitscan) and Projectiles have been included as examples. 

Once you have skills down, then it's time to move on to creating assets. Of course you can always start with the assets if that's what you want.

### Model Import
The software generally used to create and animate models is [Blender](https://www.blender.org/)- even Hopoo himself uses it. There's lots of tutorials out there so Blender is the one I recommend. However if you're comfortable with something else feel free to use that.

When creating a character model:
- it's important to note that you're limited to having one material per mesh. 
  - There is, however, no limit to the amount of meshes you can have on your model.
- See the [Survivor Model Tips(under construction)]() page for more info on how we've learned to make survivors over the years.
 
When exporting your character model from blender into Unity, it's important to fix a scaling issue that can happen if your export settings aren't set correctly.  
Here are the recommended FBX export settings:  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/0ff81e54-e4b7-48db-af24-e4de2e85756d)  
The important things to note are to set `Apply Scalings` to `FBX All`, and turn off `Add Leaf Bones`. Other than that, it doesn't have to look exactly like this.

## Setting up your character prefab in Unity

You're gonna want Unity version 2019.4.26f1, the one Risk of Rain 2 currently uses. Download [here](https://unity3d.com/get-unity/download/archive)

Once you have this, if you haven't already, grab Henry's Unity project from the repo. This is recommended over creating a new one so that you can use Henry as a reference as well as add several scripts I've included to make things easier.

Once that's done, follow these steps:

  1. Import your model into unity. Drag it into the scene. Rename the `GameObject` to "mdlYourCharacterName", replacing YourCharacterName with your character's name of course. Remember the name you set here.
  2. Add `Animator` component if one doesn't already exist. I highly recommend copying Henry's animator (`animHenry`) and adding your own animations in.
  3. Add a `ChildLocator` component to your object.
  4. Add a empty `GameObject` as a child of your object. Call this "MainHurtbox" and add a `Capsule Collider`. Scale it to fit your character and make sure the pivot of the object is set around the center of your character. This determines the `corePosition`, where enemies will aim at and where effects like `Berzerker's Pauldron` will show up.
  5. In your `ChildLocator`, create a new entry called "MainHurtbox" and drag your hurtbox object into it. Once you've done this the code will handle the rest.
  6. Create more entries in your `ChildLocator` for every mesh you have in your model and drag those in where necessary. The names don't matter because you'll fill in that part in the code.
  7. Drag your `GameObject` into the `Project` window to create a new prefab. when prompted, make a `Prefab Variant`. Make sure the name of this prefab in the project window is "mdlYourCharacterName" as before.
  8. Duplicate the object in the scene. Drag that object into the `Project` window to create another prefab, and when prompted, make another `Prefab Variant`. Then change the name of this to "YourCharacterNameDisplay". Remember the name you set here as well.  
      - This will be your display prefab used in the lobby. Here we usually the replace `Animator` to a new one that just plays the lobby animation. 
  9. Once this is all done, open the AssetBundle Browser and create a new AssetBundle (If you don't have the AssetBundle Browser window, go to Windows > Packages and there you can install it). Drag whatever assets you're using into there and build it. I like to have everything I need in a single folder and just drag that to keep things simple.
 10. If this was done right, you should be able to navigate to where you built the AssetBundle and see a game ready bundle. 
     - If you've cloned the HenryTutorial repo, I recommend to navigate to `Build\plugins\AssetBundles` and put your new assetbundle into this folder, replacing the `myassetbundle` file that you see there.
        - If you're skipping the character model process and just want to build with henry, you must at least rename `myassetbundle` to something else.
        
Next step is hooking it all up in your code.

## Setting up your character prefab in code

Assuming you've cloned the HenryTutorial repo for this one, and have a basic understanding of how to navigate a C# project. Before you start, there's a few components that you're gonna be working with that it's important to understand.

#### CharacterBody

`CharacterBody` is the main component that handles most things relating to a character. This handles things like base stats, crosshairs, item interactions, etc. There's way too many things for me to go over so it's best to learn about this through experience.  
Henry's code takes care of all of the setup for this when creating your own body so don't stress over it too much.

#### CharacterModel

CharacterModel is the component attached to every character model. It uses something called 'BaseRendererInfos' to handle things like overlays and skins.

A `BaseRendererInfo` consists of mainly a `renderer` (MeshRenderer, SkinnedMeshRenderer, ParticleSystemRenderer, etc.) and a `defaultMaterial`. The `ignoreOverlays` field (self-explanatory what this does) should be set to false if you encounter issues like weird shield outlines.

#### ChildLocator
 
A simple component that's used to easily reference child objects. This is useful for item displays, particle effects, muzzle flashes, and more. I make use of this component a lot to cleanly create characters.

#### InputBankTest

The `InputBank` is used to cache inputs from players as well as fake inputs from AI. You shouldn't need to use this much aside from a few skill interactions, like charge attacks.

* * *

There's plenty more to go over but these are the main ones you need to know. Reading through the game code with `dnSpy` always helps. Now it's time to move on to Henry's code.

# Tutorial
## Step 1 - Initial Setup

This step is very important in order to avoid conflicts with other mods.

- The first thing you wanna do is go through and change the name of the character to your own.  

- Open up `HenryPlugin.cs`  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/ab822eed-3510-49e8-b40b-c5e38014d22e)

- <ins>Rename the `HenryMod` namespace to a name of you're choosing.</ins>

- Then navigate to this section, and change these fields to yours:  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/fda05d8d-9269-4075-8760-2d27622da461)  
  - Standard naming convention is `com.AuthorName.YourModName` so go ahead and fill in the fields accordingly. For the version, it's good practice to follow [Semantic Versioning](https://semver.org/). This isn't enforced but you'll probably be made fun of if you don't follow it.

- Now, in your Solution Explorer. rename all the instances of `Henry` to a name of your choice.  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/6f06d99a-f332-45a1-9741-c50752a31f1f)  
It's a lot, but everything related to henry is contained within the Henry folder there.

- Open up the (now renamed) `HenryTokens.cs`  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/0b32dd81-cb5d-4408-af66-769049e03da7)  
You can save this for later, but this is where all the text for your character is registered into the game. Remember to use this prefix to avoid conflicts with other mods. You'll set this in the next step.

## Step 2 - CharacterBody Setup
- Next you're going to want to create a class for the survivor you're creating. `SurvivorBase` is an abstract class that you can inherit from that takes care of most of the work for you. You can work with or copy the `HenrySurvivor` class which implements this.  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/92220ecf-fc73-4c60-9b08-da59e0f9a2e4)

- Open up this (hopefully renamed) class and you'll see some important fields you need to fill out for your character setup.  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/36033700-d5c3-4a39-9fc3-0d24dd306029)
  - <ins>Set `assetBundleName` to the name of the assetbundle you created before</ins>. This <ins>must</ins> be changed to avoid conflicts.
    - If you skipped the unity section above and just want to build the project with henry, you must still rename this field, and as such rename the file for the assetbundle to match.
  - <ins>Set `bodyName` to a unique name for your character</ins>, conventionally ending in Body. 
    - This is the internal name that's not seen by the player, so it can be whatever you want, but make sure it's unique, as having duplicate bodies in the game tends to break things horribly.
  - <ins>Set `masterName` to a unique name for your character</ins>. This will house the AI of your character when spawned with vengeance or goobo. Again, this is to avoid duplicate conflicts.
  - Set `modelPrefabName` and `displayPrefabName` to the according prefabs you created in unity. This is what the template's code will use to load your character.
  - Rename `HENRY_PREFIX` as you see fit, and use the `DEVELOPER_PREFIX` from earlier. This will make adding unique language tokens easy.
  - Now's a good chance to rename the `HenryMod.Survivors.Henry` namespace if you haven't.

The most important conflict-avoiding part of the process is out of the way. Now we can finally get to the fun stuff.

- This is the `bodyInfo` of your character. This code right here is what sets up your entire `CharacterBody` prefab.
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/4c5ac1ca-3465-450b-8ff7-800dea8a6cc7)
- Feel free to look at the `BodyInfo` class to see more options like other stats and camera stuff. The most important ones are here so you're likely not to need to.

A few things to note:

- The names of all the crosshair prefabs can be found [here](https://risk-of-thunder.github.io/R2Wiki/Mod-Creation/Developer-Reference/Addressables-Assets-Keys/)
- Stat scaling fields aren't shown here as there's a standard for those that should be followed, but there's nothing stopping you from changing it manually


If this is done properly, you'll now have everything in place for the code to build a functioning character body! However there's still a few more steps.

## Step 3 - CharacterModel Setup

This step is crucial if you want your character to look good and authentic ingame. All that `ChildLocator` stuff from before, linking all the meshes, comes into play here.  

Recall that the `CharacterModel` component holds `BaseRendererInfos` which reference to your model's `renderers` and `materials`. Since we're setting up our character in code, we're going to use our `ChildLocator` and to find these renderers and add our `BaseRendererInfos`.

For the Unity inclined, there is a more [editor friendly approach(under construction)](), but continue on if you prefer code.

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/edfb51cf-a8c6-4de4-a65e-f764c46aa0b9)  
This is the magic code that adds a `CharacterModel` component with proper `RendererInfos` to your character:
- `CustomRendererInfo` is a class defined in the `Prefabs` modules that's designed to store basic information that's used to create your `RendererInfos`. You're gonna need one for every renderer you linked in your `ChildLocator`.
  - `childName` would be the name of the transform pair while `material` would be whatever material you created just before this.

- We use `Materials.LoadMaterial` extension to load a material from your assetbundle, and apply Hopoo's shader using the properties of the material as you set it up in unity. 
  - Make sure the name matches here or it'll error and show up white.  
  - To add emission and normal/bump maps, set up all that stuff in editor, and it should seamlessly transfer here.
    


- So, if you had a character with a body mesh and a helmet mesh, you would do the following:

```
public override CustomRendererInfo[] customRendererInfos => new CustomRendererInfo[]
{
        new CustomRendererInfo
        {
            childName = "BodyModel",
            material = assetBundle.LoadMaterial("matBody"),
        },
        new CustomRendererInfo
        {
            childName = "HelmetModel",
            material = assetBundle.LoadMaterial("matHelmet"),
        }
};
```
- Note: if you have all your materials on your model in unity, you may be able to avoid all this by simply specifying the childnames:

```
public override CustomRendererInfo[] customRendererInfos => new CustomRendererInfo[]
{
        new CustomRendererInfo
        {
            childName = "BodyModel",
        },
        new CustomRendererInfo
        {
            childName = "HelmetModel",
        }
};
```
 - If no material is passed in, whatever material that's currently on your renderers in unity will be transferred to hopoo materials and applied to rendererinfos.

### You're done kinda!

With all that taken care of, you should now have a working character that can be selected and played with ingame! If you run into any issues at this point be sure to go back through and make sure you've followed every step properly.  
Follow the [Building the Mod(under construction)](https://github.com/ArcPh1r3/HenryTutorial/wiki/Building-the-Mod) page to make sure you structure your mod install properly.

# Developing your Character
Now that setup part of the character is done there's a few more things to do to truly complete your survivor. These are aspects that have been streamlined with this template, but are ultimately up to you to figure out how you go about!

## 0. About the Template
Past the `CharacterBody` setup and `CharacterModel` setup from earlier, there are a few more fields

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/7d845040-c23e-42a6-8324-1fbe85ed8fee)
- `characterUnlockableDef` and `itemDisplays` will be explained below. It's possible to set these as null for now.
- The `assetBundle` is set on initialization using the `assetBundleName` you set earlier. As you can guess, you'll be using this to load the assets for your character.
  - With this, if you have multiple characters, you can have separate assetbundles for each.
- The fields following are all set in character initialization. They are exposed here for you to use as you further build up your character if you need (mainly the bodyPrefab)
- `CharacterBase` is a singleton. Not shown here is an `instance` field that will allow you to access information from this class from anywhere else (mainly to use the assetBundle).
  - for exampe `HenrySurvivor.instance.assetBundle.LoadAsset<Sprite>("someIcon")`

Initialization:

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/27a5951b-9059-4526-a504-dad74a26e6f6)

The code here is self explanatory or commented, this is where all the magic happens. We'll go into further detail in the sections below

## 1. Skill Setup

#### 1-1. EntityStateMachines
First and foremost, your character needs `EntityStateMachines`:

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/86b6d745-cff8-4e2a-a9fb-c37b98f0654d)
- A State Machine, as it sounds, is what governs the state your character is in. In ror2 you can have multiple state machines so that multiple states can happen at the same time, within your control.
- When you create skills, you will assign them to these state machines.
- The "Body" state machine typically has `GenericCharacterMain` as its main state. You can take a look at that class in the game's code to see why.
  - You can also create your own `EpicCharacterMain` class that inherits from it, to add your own functionality to the character as it is in its main state.

#### 1-2. Skills
- Your character's `skill`s are `def`ined in what's called a `SkillDef`. Mind blowing I know.
- The action that you take when you execute a skill is called an `EntityState` while the `SkillDef` governs the icon and cooldown and all that good stuff on the bottom of your screen.
- While the `SkillDef` holds the information, on your character are `GenericSkill` components that hold your currently selected `SkillDef`
- You then have a `SkillLocator` component which holds a reference to your respective `GenericSkill` components for Primary, Secondary, Utility, and Special, so your main state can call on them and perform your skills.
- Sounds like a lot, but tl;dr: `SkillLocator` => `GenericSkill` => `SkillDef` => `EntityState` => ??? => Profit

`InitializeSkills` method contains all the code for setting up your character's `SkillDefs`. It should be fairly self-explanatory and easy to work with.

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/14ba5825-01e3-4f25-aa13-f23547ace368)  
The `CreateSkillFamilies` will create `GenericSkills` on your bodyPrefab for your four skill slots, and create `SkillFamilies` for them. As you can guess, `SkillFamilies` are what hold your Skill `Variants` that you can select in loadout.

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/579fc1f3-03d8-48e3-84a8-f88a82eafe62)

So this is about as simple as it can get. Using `Skills.CreateSkillDef` you create the `SkillDef` for your skill and fill in all the values with what you need.
- `skillName` is the internal name of the skill used by saving/loading profiles
- `keywordTokens` are the keywords that pop up when you hover over the skill in the character select screen
- `activationState` is the `EntityState` you're entering when you use the skill
- `activationStateMachineName` is a bit complicated, simply put you want `Weapon` if it's a skill that allows you to move while using it and `Body` if you don't want that
- `interruptPriority` will be explained down below
- `baseRechargeInterval` is the skill cooldown
- `baseMaxStock` is the stock you start with
- `rechargeStock` is how many stocks you regain after every cooldown. usually for "ammo" type skills like visions of heresy
- `requiredStock` is how many stocks are required to use the skill. Usually set to 0 for primary attacks, or other unique kind of skills
- `stockToConsume` is how many stocks this skill loses on use. Usually set to 0 for fancy skills that will deduct the cooldown in different ways. 
- `resetCooldownTimerOnUse` usually used for "ammo" type skills like visions of heresy.
- `fullRestockOnAssign` is usually set to true.
- `dontAllowPastMaxStocks` is usually set to false.
- `mustKeyPress` determines whether the skill is automatically reactivated when held. false is recommended in most cases
- `beginSkillCooldownOnSkillEnd` is usually used for charged or aimed moves and the like.
- `isCombatSkill` determines whether out of combat effects like `Red Whip` are cancelled on use
- `canceledFromSprinting` determines whether the skill is cancelled by sprint inputs
- `cancelSprintingOnActivation` determines whether the skill cancels sprint on use
- `forceSprintDuringState` determines whether your character is forced to sprint during the skill

So once you've created your `SkillDef`, `Skills.AddPrimarySkill()`, `Skills.AddSecondarySkill()`, etc, will add it to the character pain free. You can add as many skills as you want with this method.

#### 1-3. EntityStates
- Go nuts.
- Some states have been included as examples, but there's also an entire library of skills in the base game and other mods to look to for reference.
- Don't forget to register your state as henry does in the `HenryStates` class

#### 1-4. SkillDefs, Expanded
- A `SkillDef` is more than just a vessel that holds your skill information. It governs your ability to use the skill, and can even help pass information into the state as it's entered.
  - For example, Henry's primary is a `SteppedSkillDef`. This will increment a `step` counter and pass that into the entity state, so you can do combo primaries. 
- I highly recommend reading into the `SkillDef` class in the game's code, and looking at examples of existing custom `SkillDefs`, for example `HuntressTrackingSkillDef`

#### 1-5. Hitboxes
Feel free to skip this section if you aren't going to do any melee attacks.

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/4e141485-4b12-45ce-9e64-9f080c3b43aa)

- Here's an example of how you can set up a `HitBoxGroup` for your character to use in `OverlapAttacks` (most commonly melee attacks). 
  - To use this, create an empty gameobject in unity under your character prefab, and add it to your ChildLocator. 
  - Then use the name of that child here.
- A HitBox in this game is just a transform, where the game runs `Physics.OverlapBox` using the dimensions of the transform.
  - These are the cube hitboxes you see if you check out the hitbox viewer.  
  - You can look at the scene in the henry unity project for an example of what a hitbox looks like.

## 2. Skins

If you don't have any skins, you don't need to worry about this section. A basic default skin is set up. All you might need to mess with are the name token and the icon.

Skin creation isn't streamlined as much as I'd like but it's not too bad to work with as of now. Inside the `InitializeSkins` method, you'll see this code:

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/e2452732-33be-4676-a938-d7f7ee3ae252)

Most of it can be ignored. It's just grabbing the necessary components and adding the `ModelSkinController` as well as creating a `List` of our `SkinDefs`. The rest of the skin code is below.  

Not too keen on explaining it all since it's so poorly written but the rest of this method should be fairly easy to understand. If not feel free to harass timesweeper with your questions.

## 3. Character Master (AI (Vengeance Doppelganger))

All survivors and enemies and others in this game are Characters. 
- A Character is very modular in RoR2, where a survivor is simply a player-controlled character, and an enemy is simply an ai-controlled character.
- This is why enemies can have items, why you can spawn as an enemy and use its attacks, and why AI can be applied to survivors.

When you create a character, you should also create a Character Master for it to be controlled by AI.
- A character master is simply an object that performs inputs on a character body.
- AI in this game has been simplified to components on the master, so it's easy for anyone to write AI for their character.

![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/71cb817f-e6b5-4e9b-8905-73b40539664d)  
it's as the comments say

## 4. Assets and Effects
There are a lot of helper functions in the `Modules.Assets` class used to load prefabs from your unity project to use as projectiles or effects. Examples of this are seen in the `HenryAssets` class.

As you get more and more experience creating assets for your character, you'll want to experiment with what workflow works best with you. Maybe you like cloning the game's assets and recoloring them. Maybe you like having just visuals set up in unity and putting together the necessary components in code. Maybe you prefer setting it up mostly in unity and just loading them into your mod ready to go. Feel free to ask questions as you go about these.
#### 4-1. Projectiles
I will probably be creating a page dedicated to projectiles. stay tuned for that. For now there are plenty of projectiles you can steal from the game. Henry's bomb is actually an example of this :P.

# To Go Further


## 5. Item Displays

Ah yes, who doesn't love 2400 lines of grueling, tedious code, doing nothing but copy pasting blocks and blocks of code and then typing in values.

Thankfully `KingEnderBrine` has made a tool that streamlines this process to an absurd degree, so no one else has to ever experience that pain again.

[ItemDisplayPlacementHelper](https://thunderstore.io/package/KingEnderBrine/ItemDisplayPlacementHelper/)

Now, all you need to do is install that tool, run the game, press `F2` on the main menu, select your character and start dragging items around.

Seriously, this section would've been a lot longer without that tool.

#### How do
- The code responsible for setting it up can be found back up top near your bodyInfo and customRendererInfos:  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/7da135ba-faca-4cf1-87f8-88dde837fd51)   
All of the initialization is handled in `CharacterBase`. If you don't want displays yet, simply set that to null.  

- Go into the `HenryItemDisplays` class and all the Item Display Rules have been generated.  
If you want to generate these rules yourself, or the game has updated and you want to generate the remaining rules, see the `Modules.ItemDisplayCheck` class.


- Head to the top of the `HenryItemDisplays` class and copy this block here:  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/fbee5e32-c5b8-4bd2-8d50-ef5e9ac8a3b2)

- In the Item Display Placement Helper mod:
  - select any character on the top left
  - select any item display on the middle left
  - select one of the display prefabs on the right
  - click this cog:  
  ![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/0381948c-d8c9-4275-8dc4-e728e8199fe1)
  - paste the block you copied above into the `custom` field

You're good to start setting your item displays!

#### Child Locator

You'll notice Henry has a bunch of entries on his ChildLocator for his various body parts. These are how item displays attach to your bones. 
- You don't have to create an entry for every bone on your rig, just the ones you want to put displays on. 
- There is an `ArrayTransferTool` component in the Unity Project that makes this a little easier.
  - Attach it next to your child locator in editor, drag many objects at once into the `objeys` field, right click and choose "Send to ChildLocator". remove the Array Transfer tool when you're done
- Even if your bones are named differently, I recommend naming the child locator entries for your main body parts (arms, legs, head, chest) the same as henry's
  - There are a few spots in the game where some code is looking for these names

## 6. Unlockables And Achievements
I know you're just here for Mastery Unlocks.

- `Unlockables`, as named, are the challenges that you can unlock. `Achievements` are the code used to check if you've completed a certain task.   
- These two things are connected by an `identifier` that you set.

Creating a mastery achievement has been greatly simplified. All you need to do is plug in your character's name and `RequiredDifficultyCoefficient`
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/706bd249-7f47-4ef2-85f8-56c798547b8b)  

The `RegisterAchievement` attribute is how you get your achievement in the game. I recommend storing those identifiers in const strings so you can reference them elsewhere:  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/9822418d-8b20-4c94-863f-703501435851)

If you're curious about writing your own achievements, you can take a look at the `BaseMasteryAchievement` there, and any of the vanilla achievements for reference
One thing to note is that certain things like on kill achievements must be server tracked, otherwise only the host can unlock them. [HAN-D Overclocked has a good example of this](https://github.com/Moffein/HAN-D_OVERCLOCKED/blob/master/HenryMod/Content/HANDSurvivor/Achievements/HANDSurvivorUnlockAchievement.cs), as well as, again, many of the vanilla achievements.

Unlocks are generally optional but people like having them, and it's a good way to get people to experiment mechanics with your character. If they don't there's the CheatUnlock mod so no biggie.

--------------------

That should cover just about everything. See [this page(under construction)](https://github.com/ArcPh1r3/HenryTutorial/wiki/Building-the-Mod) for how to properly build and install your mod.  
If there's anything I'm missing or anything that doesn't make sense feel free to let me (`thetimesweeper`) know in the modding discord.  
Remember, the discord is there to answer your questions and help you along the journey. There's loads of experience you can call upon there.

Happy modding!
