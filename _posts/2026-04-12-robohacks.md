---
title: "Y Combinator Robotics Hackathon: Robohacks"
author: Michael
date: 2026-04-12
categories: []
tags: []
math: true
mermaid: true
pin: false
image:
excerpt: "  How do you safely deploy complex manipulation policies to a robotic arm in just 36 hours without breaking the hardware? We solved this through a custom architecture, aggressive NVIDIA Jetson memory hacks, and a seamless zero-shot sim-to-real pipeline. This post breaks down how we built a MuJoCo digital twin, optimized our ROS2 pipeline, and won the Best Simulation Award at YC RoboHacks '26! 🚀 <br> (Click the Title to see more)"
---

<div>
  <iframe
    src="https://www.youtube.com/embed/aPjvCDiUd-A"
    width="720"
    height="405"
    frameborder="0"
    allowfullscreen>
  </iframe>
</div>

<br>
# Bridging Simulation and Reality: Winning the Best Simulation Award at YC RoboHacks '26

There is an undeniable energy that happens when you put 120 builders and 25 robots in a room for a single weekend. 

Recently, I participated in RoboHacks '26, a robotics hackathon hosted by Y Combinator, Google Deepmind, Scale AI and Innate (YC F24) at the YC headquarters in San Francisco. Builders were given 36 hours to develop innovative software and capabilities on top of Innate’s MARS--a robot equipped with a 7-DoF arm, sensors, and onboard NVIDIA Jetson running a custom OS built on ROS2.

My goal for the hackathon was straightforward: build a robust, safe, and highly optimized pipeline and simulation environment for executing complex manipulation tasks on physical hardware. I am incredibly proud to share that this technical focus earned my project the **Best Simulation Award**. 

Here is a deep dive into the architecture and optimizations that made it happen.

---

### The Challenge: Safe Execution and Edge Constraints
Deploying complex manipulation policies to a physical robotic arm is inherently risky. Sending unverified kinematic commands to real hardware can result in collisions, damaged motors, or broken environments. Furthermore, running the heavy machine learning models required for these tasks directly on the robot's edge hardware (the NVIDIA Jetson) presents severe memory constraints. 

My solution tackled both of these bottlenecks through a high-fidelity simulation pipeline and aggressive memory management on the backend.

### Winning the Best Simulation Award with MuJoCo
To solve the safety and testing bottleneck, I built a reliable digital twin of the MARS robot's arm using **MuJoCo**. 

I integrated a custom `maurice_sim` ROS2 package and loaded the robot's URDF (`maurice.urdf`) directly into the MuJoCo physics engine. Before any manipulation policy—specifically Action Chunking with Transformers (ACT) policies—was deployed to the physical hardware, it was routed through this simulation. 

This architecture allowed for:
* **Rigorous Kinematic Testing:** Safe verification of joint states, trajectory planning, and inverse kinematics without risking the physical robot.
* **Zero-Shot Real-World Transfer:** Because MuJoCo's physics engine is exceptionally accurate and fast, the transition from simulated movements to physical execution on the real MARS robot was incredibly seamless.

This reliable, risk-free testing pipeline was the core component that ultimately won the Best Simulation Award.

### Optimizing the Edge: Memory Management and ROS2 Tweaks
Beyond the simulation, the project required significant backend optimization to ensure the robot's operating system could handle the computational load without crashing. Loading multiple heavy models consecutively was causing Out-Of-Memory (OOM) crashes on the Jetson.

To resolve this, I overhauled the policy memory lifecycle within `manipulation_server.py`. The key optimizations included:

* **Aggressive Memory Clearing:** I implemented a strict `_release_policy()` function. Once a physical action was completed, this function explicitly freed the loaded ACT policies, ran Python's garbage collector (`gc.collect()`), cleared the PyTorch CUDA cache (`torch.cuda.empty_cache()`), and reset the Dynamo compiler.
* **Dynamic Model Loading:** Instead of hardcoding model parameters, I wrote logic to dynamically infer the `chunk_size` directly from the `model.decoder_pos_embed.weight` tensors of the loaded checkpoints. This allowed the system to seamlessly swap between different policies on the fly.
* **Network Latency Reduction:** To ensure smooth data streaming across the system, I optimized the ROS2 network layer. I specifically targeted the Best-Effort QoS profile for compressed images, reducing the `depth` queue from 10 down to 2 to eliminate processing backlog.

### Looking Forward
RoboHacks '26 demonstrated that the future of robotics isn't just about having great hardware; it's about building intelligent, harnessed, memory-efficient software and reliable simulation pipelines to bridge the gap between complex ML models and the physical world.

A huge shoutout to @Innate for providing the amazing MARS robots and to @Y Combinator & @Google Deepmind for hosting such a stellar event. Also, special thanks @NASA @ElevenLabs @Axel Peytavin & everyone else for all the support! The robotics sandbox is wide open, and I'm excited to see where these robotics capabilities go next.

---

### Gallery

<div class="image-grid">
  <img src="/images/mine/hack3.jpg">
  <img src="/images/mine/hack4.jpg">
  <img src="/images/mine/hack2.jpeg">
  <img src="/images/mine/hack5.jpg">
  <img src="/images/mine/hack6.jpg">
</div>