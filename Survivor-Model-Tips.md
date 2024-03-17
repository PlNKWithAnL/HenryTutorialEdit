# (under construction)
I'll pretty this page up later, possibly much later, but I'll just dump things here for posterity for people to ask for further info or figure out

# Preface
This is information overload, and is not required for the tutorial. I recommend coming back to this when you're more comfortable with character creation and are basically ready to take your character to the next level. 

# Unity Stuff
## Working with the FBX (tips)
- In the tutorial, we drag the fbx to the scene, then drag that to the project to create a Prefab Variant of your fbx
- this means your prefab and the fbx file are linked. don't delete the fbx as your prefab depends on it
- I recommend working this way for several reasons
  - Any changes to your FBX will automatically be updated in your prefab. you can freely rename and rearrange bones and meshes without the insane hassles of the past
    - beware, if you make a major rename or rearrangement, unity might lose the reference to the transforms under it. if you're doing a large change like this, duplicate and unpack your existing prefab so you can transfer any necessary additional objects or components you may have added to your prefab.
    - of course, always use source control so you can undo your changes to the files if something goes wrong
  - in your animator, reference the animations from your fbx directly, so if you need to update animations in your fbx, simply save over the existing ones and your animations will update automatically
    - One caveat is that unity will hold some properties of the previous animations. 
      - when adding a new animation, you will have to add the clip in animation import settings
      - if you change the length of the animation, you'll have to update the length of the clip in unity as well
- as such, this workflow works best when you only use one fbx for your character. 
  - of course skins and such can be in another fbx, but try not to use multiple fbx's with different animations or different meshes. it gets messy

## Ragdoll
Ragdolls are pretty easy with The Bone Script. 
- add transforms from your hierarchy to the ragdollcontroller (you don't have to add every single bone. just main body parts)
  - right click and hit "make bones on the bones". this will generate colliders and rigidbodies needed
- if you want things to detach when ragdolling, remove the `CharacterJoint` component and just keep the collider and ragdoll
- make sure you remove the `CharacterJoint` from your base-most bone. 
  - I usually make this the stomach bone (so remove the `CharacterJoint`), then have the `CharacterJoint` in the pelvis bone's `ConnectedBody` field set to the stomach bone.

# Model Stuff
## Rigging
A reference for how we usually rig our characters:  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/fff3426a-2be3-4539-b438-9f4a23eca53a)


here's aliem. the most important part is that the pelvis can move around freely without affecting stomach/spine  
<img width="300" src="https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/f681daf5-437e-4d8e-8ae8-f1cbfb0a7cd8" />

some rigs look like this and this is bad (for our characters)  
<img width="300" src="https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/0c244f45-ec67-43ca-80e1-750740a56a08" />

### why
- we usually mask animations to the upper body so they can play while moving. If your lower body pelvis rotates, it'll affect your upper body as it does attacks n such
  - [Like This](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/ae4c8006-ea62-497e-9634-db9f6abf54ab)
- as such, unless you're doing a full body animation, never rotate the `base` bone. move it around all you want, but rotate the pelvis and stomach bones instead if you want rotation

## Animation
here's the list of animations characters usually do

#### Main Animations
- Idle
- Run (fwd, left, right, back)
- Sprint
- Ascend/Descend
- Lobby (intro, idle)

#### Additional/Polish/Legitness
- Aim pitch, yaw
  - set Additive Reference Pose in the import settings
- idle in (suddenly stop running -> idle)
- jump up (not the actual jump because jumping is instant. The aftermath of jumping)
- landing impact (additive)
  - should start from your character's idle pose

#### Abilities
- depends on your character
- depends on your character

#### Extras
no one does this lol, but if you do I'll be very impressed
- Additive Air strafes, Left-Right, Back-Front, similar to aimpitch/yaw
- custom behavior for extra polish
  - rest idle animation after standing still for a while
  - anything else you can think of
- emotes
  - yes emoteapi exists and I always recommend supporting it cause it's fun, but adding your own emotes is soul

#### Notes
- In your Run and Sprint animations, set animation events for Footstep, with the string parameter of the foot that's hitting the ground
- You'll want your run FLRB animations to all have same cadence for each foot. 
  - All animations start with the right foot, for example. 
  - All animations have the foot touch and leave the ground at the same frames.
- RUN NOT WALK
  - At the speed you go in this game, characters default to a brisk jog for basic movement.
  - If you animate your character walking, you'll end up with the skiing issue where their feet won't line up to the ground below, or you'll have to speed up the animation to match the feet on the ground, which will be too unnaturally fast
- You'll usually mask your attack animations in a "Gesture, Override" layer to only happen to the upper body, so feet can run around while moving
  - If you've hired a certain really good animator and they gave you a really fuckin sweet full body swing animation or so, you can play this as a "FullBody, Override" animation while standing still, as well as your "Gesture, Override" animation
  - have a transition in your animator to BufferEmpty that will activate when isMoving is true, so the full body can end early and let your feet move, but the upper body will continue the animation  
```csharp
base.PlayAnimation("Gesture, Override", "SwingL", "Swing.playbackRate", this.duration);
if (base.isGrounded & !base.GetModelAnimator().GetBool("isMoving"))
{
    base.PlayAnimation("FullBody, Override", "SwingL", "Swing.playbackRate", this.duration);
}
```
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/6696a55d-f15f-4fcd-9b7f-ade92ddc1004)  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/ec6bb926-9418-4065-9388-51635f193278)

## Limb Masking
For Item Displays to hide certain parts of your body (e.g. goat hoof), set up vertex colors on your model.

From the discord:  
- Limb mask uses prime numbers on the model vertex colours red channel.  
- Valid colours (0-255 range) are 2 (Head), 3 (RightArm), 5 (LeftArm), 11 (RightCalf), 17 (LeftLeg)  
<img width="500" src="https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/65ac2f92-83a0-49dc-bd81-494ef15fa398"/>

- *currently only head (ego), rightcalf (goat hoof), and right arm (capacitor) are used*  
- The model material should use `Hopoo Games/Deferred/Standard` shader and have `Enable Limb Removal` selected.  
  - Since henry uses the unity standard material, in unity, right click the inspector tab and choose Debug, then add `LIMBREMOVAL` to the keywords