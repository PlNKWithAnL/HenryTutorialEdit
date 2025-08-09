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
1. add transforms from your hierarchy to the ragdollcontroller (you don't have to add every single bone. just main body parts)
2. Right click on the ragdollcontroller component and hit "make bones on the bones". this will generate colliders and rigidbodies needed
- If you don't have the EditorAddRagdoll.cs script in your project, see here: https://github.com/TheTimeSweeper/HenryEditorTools.git 
- If you want things to detach when ragdolling, remove the `CharacterJoint` component and just keep the collider and ragdoll
- Make sure you remove the `CharacterJoint` from your base-most bone. 
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
  - set Additive Reference Pose in the import settings (see below)
- idle in (suddenly stop running -> idle)
- jump up (not the actual jump because jumping is instant. The aftermath of jumping)
- landing impact (additive)
  - should start from your character's idle pose

#### Abilities
- depends on your character
- depends on your character

#### Extras
no one does this lol, but if you do I'll be very impressed
- custom behavior for extra polish
  - Additive Air strafes (see below)
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
- Set Additive Reference Pose in the import settings  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/43706808-bf4e-4065-a766-bc549270ffd5)
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
- Additive Air strafes: character leaning in the direction they're jumping/falling
  - Have a Neutral pose (a frame from your ascend or descend). This animation has to be at least two frames  
  - Have Front, Left, Right, Back leaning poses
    - these animations need to have a dummy frame at the very beginning which is the Neutral pose, and at least two frames immediately after with the pose you want for that lean
    - in Unity, make the animation Start at frame 1, so the Neutral pose is not clipped in the animation
    - hit Additive Reference Pose and set it to 0   
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/59bfe550-bcec-47ea-b7b0-7ece7bb21a75)
  - put them together in a new blend tree, set up just like the one for Run
  - put them in a new layer, marked additive, with a transition going to/from it based on isGrounded parameter  
![image](https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/e69c54ef-fe59-43ad-aa23-e1c3c07babe4)

## Limb Masking
For Item Displays to hide certain parts of your body (e.g. goat hoof), set up vertex colors on your model.

From the discord:  
- Limb mask uses prime numbers on the model vertex colours red channel.  
- Valid colours (0-255 range) are 2 (Head), 3 (RightArm), 5 (LeftArm), 11 (RightCalf), 17 (LeftLeg)  
<img width="500" src="https://github.com/ArcPh1r3/HenryTutorial/assets/53384824/65ac2f92-83a0-49dc-bd81-494ef15fa398"/>

- *currently only head (ego), rightcalf (goat hoof), and right arm (capacitor) are used*  
- The model material should use `Hopoo Games/Deferred/Standard` shader and have `Enable Limb Removal` selected.  
  - Since henry uses the unity standard material, in unity, right click the inspector tab and choose Debug, then add `LIMBREMOVAL` to the keywords

### Okay but how
- Select the vertices of the model that you want to paint. we'll start with Head  
  <img width="502" height="388" alt="image" src="https://github.com/user-attachments/assets/e7985eb0-0080-42a1-94d4-f931e7d20d08" />
  - You can just select vertices using Edit mode.
- Then, move to vertex paint mode  
  <img width="267" height="217" alt="image" src="https://github.com/user-attachments/assets/6d4f394a-71bc-4f6c-ae07-1207cff052e9" />
- Next to the dropdown, select Vertex Selection, and select the paintbrush tool, so that the color is available to you  
  <img width="639" height="463" alt="image" src="https://github.com/user-attachments/assets/30c84c98-6450-464b-be58-b7069210f05d" />
- Click to change the color we're going to paint. What we're going to do is set the **hex color value**. I will explain what value to use in the next section  
  <img width="252" height="326" alt="image" src="https://github.com/user-attachments/assets/39a9b490-d695-41b4-a571-762e37d34911" />
- Click the Paint menu, and hit Set Vertex Colors to apply the colors  
  <img width="370" height="241" alt="image" src="https://github.com/user-attachments/assets/0f8e8449-5e53-4e51-8b34-83ee0a82c86f" />

### Hex Color Value
**tl;dr:** 020000 (Head), 030000 (RightArm), 050000 (LeftArm), 0B0000 (RightCalf), 110000 (LeftLeg)

You may be familiar with other softwares and websites treating colors from ranges of 0 - 255. The #FFFFFF color codes are hexadecimal representations of this. first two digits are RR then GG then BB.  
So a color code of #000000 is black, i.e. 00 for Red, 00 for Green, 00 for Blue. A color code of ##00FF00 is full Green. 00 for Red, FF for Green, and 00 for Blue.

But wait, why FF? why not 99?

#### Hexadecimal is a number system that computers like.
We as humans count from 1 to 9 then go to 10, this is called Decimal.  Hexadecimal doesn't stop at 9. it goes 1 to 9, A, B, C, D, E, F, then 10.  
This means that every 10 place in Hexadecimal is equivalent to 16 for us.

so a color value of #030E12 has
- 03 for red
- 0E for green, so count 9, a, b, c, d, e to get 14
- 12 for blue, so 10 + 2. 10 in hex is 16 for us, then add 2 more to get 18

did that make sense? maybe? if you're still curious, you can search "hexadecimal color codes" and find someone way better at explaining it than I am

#### So what does this have to do with goat hoof item display
In RoR2, the limb masking system just reads the vertex color of the model, and takes the red value. if it is equal to certain values, it hides that part of the model.
- Valid colours (0-255 range) are 2 (Head), 3 (RightArm), 5 (LeftArm), 11 (RightCalf), 17 (LeftLeg)
- This translate in hexadecimal to 02 (Head), 03 (RightArm), 05 (LeftArm), 0B (RightCalf), 11 (LeftLeg)

The rest of the color can be anything, so we can just set it to 0000

Now if gbx adds any other limb mask sections. you'll be able to deduce what hexadecimal number this is. (but i'll probably just update this page anyways lol)





