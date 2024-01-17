---
layout: default
title: "Blog"
permalink: /blog
---

[Home](https://hiddendevxr.github.io/)

## GOAP System for a "Automated" Farm in Unity

[17/01/2024]

![Screenshot 2024-01-17 193910](https://github.com/hiddenDevXR/hiddenDevXR.github.io/assets/86928162/0c45f129-ea79-408d-bef9-ef467b280d57)

For the final project in the "Artificial Intelligence in Video Games" course, we were tasked with creating a prototype that employs the Goal-Oriented Action Planning (GOAP) system. For this project, I aimed to implement an "automated" farm system using Unity. The core concept was as follows:

> Theme: Cooperative Farming Robots
>
> In this scenario, you we a farm with three simple AI agents responsible for different tasks to ensure the success of the farm. The goal is to make farming operations efficient and collaborative.
>
> Agents:
>
> Harvester Bot:
> Specialized in harvesting crops.
> Actions include picking ripe crops, clearing fields, and depositing harvested crops.
>
> Watering Drone:
> Responsible for managing irrigation and watering crops.
> Actions involve checking soil moisture, watering crops, and adjusting irrigation systems.
>
> Security Scarecrow AI:
>
> Tasked with scaring away pests and ensuring the safety of crops.
> Actions include patrolling fields, shooing away pests, and alerting other agents in case of a threat.
>
> Actions:
>
> Harvester Bot action: Identifying and harvesting ripe crops.
> Watering Drone action: Checking soil moisture levels and watering crops as needed.
> Security Scarecrow AI action: Patrolling fields to scare away pests and ensure crop safety.
>
> Emergency Response:
> All agents collaboration: Responding to unexpected events, such as a sudden pest infestation or adverse weather conditions.
> Resource Sharing:
> Coordination between Harvester Bot and Watering Drone: Sharing information about crop status and water needs for efficient resource allocation.
> Harvester Bot action: Clearing fields of leftover plant debris for better cultivation.

However, as I commenced the prototype development, I decided to replace the security scarecrow with two additional agents: a vendor bot with the primary objective of selling processed crops and a client. In the following image, we can observe a straightforward representation of how the goals of each agent intertwine with one another.

![GOAP Diagrama](https://github.com/hiddenDevXR/hiddenDevXR.github.io/assets/86928162/eba64dc3-b2a9-4f16-b10f-5d2a53ac0422)

The harvest bot retained its behavior as per the initial design. It had three subgoals: checking if the crop was ready for harvest, delivering the harvested crops to the processing container, and recharging its battery whenever it was running low on energy. To determine if the crops were ready, the harvest bot needed to check in at a "computer" near the field. The crops were considered "ripe" once they were watered by the watering drone. After the harvest bot collected the crops, the crop object transitioned to a "crop dry state," signaling to the watering drone through a world state that they required water.

```c#
    void Start()
    {
        base.Start();

        SubGoal s0 = new SubGoal("cropsChecked", 1, false);
        goals.Add(s0, 2);

        SubGoal s1 = new SubGoal("deliver", 1, false);
        goals.Add(s1, 3);

        SubGoal s2 = new SubGoal("recharged", 1, false);
        goals.Add(s2, 4);

        Invoke("EmptyBattery", Random.Range(10, 20));
    }
```
![harves bot](https://github.com/hiddenDevXR/hiddenDevXR.github.io/assets/86928162/d3372135-4694-408b-b534-47476af43458)

The watering bot operates similarly to the harvest bot. It waters the crops when they are dry and returns to the pond to get more water afterward. It can deviate from its current target if it detects a fire arising in the field. Due to the higher weight assigned to the fire hazard state, the watering drone prioritizes addressing this goal.

```c#
public class PutFireDown : GAction
{
    public GameObject smokeVFX;

    public override bool PrePerform()
    {

        return true;
    }

    public override bool PostPerform()
    {
        beliefs.RemoveState("cropsOnFire");
        smokeVFX.SetActive(false);
        return true;
    }
}
```

![Screenshot 2024-01-17 231431](https://github.com/hiddenDevXR/hiddenDevXR.github.io/assets/86928162/306a822e-7615-4bbd-ad8a-437ba623cec7)

The vendor bot interacts with the harvest bot by waiting for new crops to arrive at the processing plant and with the client agent by waiting for orders. When clients arrive at the store, they place an order. Once the vendor receives an order and there's an available crop, it goes to the processing plant, collects the crop, and delivers it to the counter for the client.

```c#
public class SellCrop : GAction
{
    public GameObject crop;

    public override bool PrePerform()
    {
        return true;
    }

    public override bool PostPerform()
    {
        GWorld.Instance.GetWorld().ModifyState("orderReady", 1);
        crop.SetActive(false);
        return true;
    }
}
```

By modifying the world state to 1, the vendor bot communicates to the client agent that the order is ready and available for pickup at the counter. Once this action is completed, the client exits the store.

```c#
    public override bool PostPerform()
    {
        beliefs.GetStates().Clear();
        GWorld.Instance.GetWorld().ModifyState("orderReady", -1);
        Destroy(gameObject);
        return true;
    }
```



https://github.com/hiddenDevXR/hiddenDevXR.github.io/assets/86928162/f8e2d963-4747-4330-8f37-54d830471c66

* * *

## About team management in "indie" game dev

[01/01/2024]

In my eight years across different studio sizes and cultures, I've seen how management styles vary based on the game type and studio culture. From smaller teams crafting niche experiences to larger studios chasing mainstream success, the approach to team management is as diverse as the games they create. This exploration delves into the pivotal role team management plays in the dynamic realm of indie game development, where talent, strategy, and studio ethos converge to shape a game's destiny.

In this blog post, I'll briefly share my insights gained from managing teams in the production of two distinct types of games across separate studio environments.

### Case A: Medium Size Studio and a big size Game

In this production, we had a more structured team setup where each person had a clear role, like parts of a clock coming together. My role as a technical artist involved creating VFX, shaders, and optimizing assets for the game.

We kept in touch with clients through Slack and used Google Teams for our team discussions. Inside Google Teams, we had different channels—like one for the overall project, another for engineers, and one just for artists. This split helped us avoid cluttering everyone's feed with stuff that wasn't relevant to them. For instance, artists didn't have to worry about technical hiccups; they could focus on their creative work without getting tangled in code issues or server mishaps.

We managed the project using Atlassian's Jira and followed the scrum sprint setup. Honestly, I found this approach really comfortable because it gave me specific tasks (or 'tickets') to focus on and resolve. With Jira's ticket system and priority tags, I had a clear view of what I needed to do and who I needed to talk to if I hit any roadblocks or needed some guidance.

The daily standup meetings, where we shared our previous day's progress, outlined our daily objectives, and discussed any arising issues, were something I wasn't particularly fond of. However, I've come to see their significance, especially for fully remote teams like ours. They were a necessary part of keeping track of each team member's progress, fostering team communication, and ensuring everyone was doing alright, despite the distance.

### Case B: Medium Size Studio and small size game

<img src="/assets/vrl1.png">

Alright, this wasn't your typical video game per se. It was more of an interactive experience designed for the Tieranatomisches Theatre in Berlin. However, I'll refer to it as a game because we utilized game development technology like Unity and gaming hardware such as the Oculus Quest. Plus, the experience was enjoyable to interact with and experiment using game-like features.

Given the small scope of this game, our development team comprised only three members: the project manager, the 3D artist, and myself, the sole developer. Being the lone developer for a VR experience scheduled to launch in just a couple of months was undoubtedly stressful. Yet, I appreciate this team dynamic because it allowed me the freedom to tackle challenges in my own way. For issue tracking, we relied on GitLab, which might not be as user-friendly as tools like Jira or Trello, but it served its purpose well for our small-sized team.

It's worth noting that although our team was small, the studio itself could be considered medium-sized and boasted a well-structured internal setup. With several teams concurrently handling multiple projects, our team operated more like a specialized task force. What I particularly enjoyed about this dynamic was the occasional interaction with members from other studio teams, each bringing diverse backgrounds in fields like architecture, psychology, mathematics, fine arts, and more. This blend of expertise added a unique flavor to our collaborative atmosphere.

* * *

## HALO 4 review (2023)

[30/12/2023]

As part of my Master's in Game Design course, we were tasked with reviewing a game of our choice. Recently, I've been diving back into HALO MCC, a title that initially received mixed reception upon its release but one that I've loved from day one.

> _My HALO 4 Review_
> 
> As a long-time fan of the Halo series, diving into Halo 4 was an experience filled with anticipation and excitement. Developed by 343 Industries, this installment marks a significant change in the saga, both in narrative depth and game dynamics.
> 
> One of the notable aspects is how 343 Industries attempted to infuse their own distinctive style into the franchise. For some fans, this departure from Bungie's path wasn't exactly what they expected, but it's commendable that 343 took the step to leave their mark on the series rather than strictly sticking to the stablished formula.
> 
> The most obvious change lies in the gameplay, which appears to have taken inspiration from popular contemporary shooters like Call of Duty. This adjustment could polarize fans, as it leans towards a more modernized style seen in other games. However, it is an understandable attempt to keep up with changing video game trends.
> 
> Introducing the Forerunners and the Didact into the narrative was a brilliant move. It adds layers of depth and mythology to the Halo universe, offering a new perspective and expanding the mythos. However, there is a flaw in the execution: the Didact's character development relies heavily on a trilogy of books, leaving some players disconnected from his in-game arc.
> 
> The direction of the art style in terms of graphics was controversial. Although it looks better visually, it was perceived as misguided, as it took two additional games for 343 Industries to return to the original design formula with Halo Infinite.
> 
> The narrative unfolds with the return of the Master Chief, immersing players in an immersive story that delves into the humanity of the enigmatic Spartan. The emotional depth added to the Master Chief's character brings a new dimension to the series, making the journey feel personal and captivating.
> 
> In terms of gameplay, Halo 4 successfully retains the essence of what makes the series great while introducing new elements. The combat mechanics feel refined, offering a balanced mix of familiar mechanics and fresh innovations. The addition of new weapons and enemy types keeps the game engaging and challenging.
> 
> Multiplayer, a cornerstone of the Halo franchise, receives an impressive revamp in Halo 4. The revamped multiplayer introduces innovative features, offering diverse maps, engaging game modes, and a robust customization system that encourages replayability and the competitive spirit.
> 
> The sound design deserves applause, complementing the visual grandeur with a rich and immersive audio experience. The iconic soundtrack, composed by Neil Davidge, adds emotional depth to the game, enhancing every moment of the journey. Change this part to mention that Neil Davidge replaces the legendary Martin O'Donell and that he manages to preserve distinctive melodic passages from the original soundtrack while also adding his own identity...
> 
> However, even though Halo 4 excels in many aspects, it encounters some pacing issues in its campaign, occasionally interrupting the immersive experience.
> 
> In conclusion, Halo 4 is a triumphant continuation of the beloved series. With its captivating narrative, stunning graphics, refined gameplay, and revamped multiplayer, it stands as a testament to the evolution of the Halo universe. Despite small flaws, such as the Didact's development that relies heavily on external lore, it is a must-have for long-time fans and newcomers alike, offering an unforgettable gaming experience.
> 
> Rating: 4.5/5 stars

* * *

