45 minutes  Introduction + 6 x 5 Pecha Kucha + Question Answers 


Topic unskippable:
- What is the Lynx & Why you should care
  - Whare the "Good" Company and philosophy around it
- AR Lynx vs Oculus Quest vs Hololens like vs Future
  - How to setup Unity For Quest 2 AR with Hand
- What is Open XR & Unity System 
  - How to use Open XR 
     - For trigger
     - For Hands 
  - How to use Unity System
  - Alterantive Transform and [Rewired](https://assetstore.unity.com/packages/tools/utilities/rewired-21676)
  - Unity Package a good way to split and avoid spaghetti code.
- Side Quest: How to install APK

(Note: I can't help on that topic but WebXR is good solution for some VR project because you can publish without publishing and so bypass le store blockage at the cost of having user kowing you exists.) 

# 0: Disclaimer: It makes 10 years that I have a foot in the industrie but it makes 2-3 years I did not touch VR.
- We need to kill the keyboard... like I said 2 years ago and since then I spent lot's of time creating lib for that.
- Tired of re-learning how to do virtual reality every  6 months
    -  (VRTK V1 2 3 4 5, DK1 screen DK2 API Gear VR Android  Quest 6dof Oculus half Open XR, Oculus full XR and Action based)...
    -  And I don't talk about Hololens API that is totaly different from the Steam VR.
    -  Like you can understand that the topic of this talk.
    -  OK so OpenXR, just a reminder that all this API exist and are different
       - Steam VR API
       - Oculus API
       - Microsoft MR API
       - Sony PS API
       - WebXR API
       - Alternative VR API
       - Phone Gryo API
       - But... you also have AR
       - ARkit API
       - ARCore API
       - Open CV
    -  Why Now I plan to restart VR ? 
- COVID yes but mainly because Mark and it is pre-Meta

Let's me talk about the story:

> One device to rule them all, one device to find them,
> One device to bring them all, and in the adsness bind them;
> In the Land of Meta where the shadows lie.

The story of the a device, a device sorge to
- Mark Lock Down: 3-5 weeks review before publishing
- Be accepted don't means beeing publish you are add to calendar
- You can't publish something that is not MVP and you still have 2-4 weeks review and can be refused

But then ![image](https://user-images.githubusercontent.com/20149493/175124900-6752cd95-5fb7-434b-b5ba-7bd92e3d2da8.png)
> Look to my coming on the first light of the september month, at dawn look to the east.


# 0 : Lynx a hope in the rise of the Open Community

- Mark VS Stan



What will happen when AR Glass will arrive on the market from several company at the same time

So Open XR... What is it ?
And why I think the Lynx through it company is a game changer.


#1 Lynx Is coming
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
- AR Lynx vs Oculus Quest 2 vs Hololens like vs Future
-
-
- As the Lynx this year and the other next year. Let's prepare ourself on Open XR and Quest.

# 1: Oculus Quest hand API and Ultar Leap

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

# 2: Hands instead of controller

- Oculus vs Hololens vs Leap
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

# Short reminder that all the presentation is on Github

GitHub: https://github.com/EloiStree/2022_06_18_HackInTheWood_Topic_LynxAndOpenXR/

# 3:  Unity System Introduction

- Rewired: https://assetstore.unity.com/packages/tools/utilities/rewired-21676 
- Thomas Van Bowel example of code
- Example de Magic Door 24
- Start example by explaining the Kinect problem instead of the controller The follow step is the continuation
- 
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

# 4: What is Open XR and how to use it

-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-

# 5: How to setup un AR Pass through app on Quest 2
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-
-


# 6: Demo of the library MRTK, ToolKit, ?VRTK?, Asset store


# 7: What we know won't changed

For variable you can use Unity System, yours or Rewired. 
For local point in space you can use Transform.

- Head & Center Eyes
- Controller Position (Warning on rotation)
- Root of the tracked Zone (event if lot's of way to do it it can be resume to one Transform point)
- A l'exception du Guardian(s) swap
- The ground, floor and wall ingame "detection"
- Oculus vs Ultar Leap vs Manus&Others vs Hololens vs Myo
- Finger of the hands
- Blurry zone of the hands wrist
- Ultra leap and arms tracked
- Vocal : Audio > Service (online ou offline) > Group of words
- " Vocal plus: Alexa, Facebook Ai ... >string and context "
- Spectrum of Voice: Training > Pattern > String Event (with start and end)
- EEG technology : NextMind  API > String Event + String State
- Future ?
- "Eyes Tracking ? Origine and destination"
- Oculus AI  based on camera in the Quest and outside the Quest #ViveOrigin
- "Body part track anchor points" Kinect and Open CV ?
- Mesh Around and in front ?
- Compute Shader & Job System vs Mesh or API 
- Sub topic: Kinect challenge of 512x420 = 215040 points (on phone it is a lot)
- AI Image Recognition service ? String words Cloud.

# 8 Call to action: It is a good moment to re-start on VR.

- Micro oled
- AR Passtrhough
- Open XR that start to be ready
- Oculus Quest "3" with I hope will provide 3D mesh of the room and Color AR passthrough
- Unity Input System that allows abstraction on the industry for most application
- ECS and Job system that allows multithreading
- Question
- I will spend all next year on Lynx so feel free to sent me contact or ask question at anytime on the subject.


# PS: Compute Shader and Job System for the Mesh
