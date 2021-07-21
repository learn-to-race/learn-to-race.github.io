---
layout: article
titles:
  en      : &EN       Challenge
  en-GB   : *EN
  en-US   : *EN
  en-CA   : *EN
  en-AU   : *EN
key: page-challenge
sidebar:
  nav: challenge-nav
---


<script>
  {%- include scripts/lib/swiper.js -%}
  var SOURCES = window.TEXT_VARIABLES.sources;
  window.Lazyload.js(SOURCES.jquery, function() {
    $('.swiper-demo').swiper();
  });
</script>

## Overview

Racing is one of the ultimate challenges which involves high-speed and high-risk decision making while operating vehicles near their physical limits. Learn-to-Race is a [Gym](https://gym.openai.com/) compliant framework provides agents with the ability to interact with a powerful racing simulation environment. The objective of this challenge is to explore the area of safe policy optimisation in greater detail on this difficult task.

Participants will be evaluated on their racing performance on an unseen track, the North Road Track at the Las Vegas Motor Speedway, *left*, but have the opportunity to explore the environment with unfrozen model weights for a 1-hour prior to evaluation laps similar to a professional race car driver's practice session. They will also have access to the [Anglesey National](https://www.angleseycircuit.com/), *middle*, and [Thruxton Circuit](https://thruxtonracing.co.uk/), *right*, racetracks to train their agents.

<div class="swiper swiper-demo">
  <div class="swiper__wrapper">
    <div class="swiper__slide">
      <img src="/assets/images/challenge/lvms-overhead.png" alt="LVMS">
    </div>
    <div class="swiper__slide">
      <img src="/assets/images/challenge/anglesey-overhead.png" alt="LVMS">
    </div>
    <div class="swiper__slide">
      <img src="/assets/images/challenge/thruxton-overhead.png" alt="LVMS">
    </div>
  </div>
  <div class="swiper__button swiper__button--prev fas fa-chevron-left"></div>
  <div class="swiper__button swiper__button--next fas fa-chevron-right"></div>
</div>


## Rules

The task is to Learn-to-Race, so complete or over-reliance on classical planning methods is not allowed. Additionally, participants will be:

* limited to **1** submission every 24 hours
* restricted from accessing model weights or custom logs during evalation
* required to submit source code, for top performers

### Action Space

| Action | Type  |  Range  |
|:----------:|:-------------:|:------:|
| Steering | Continuous | *[-1.0, 1.0]* |
| Acceleration | Continuous | *[-1.0, 1.0]* |

### Observation Space

We do not restrict the usage of segmentation cameras nor the placement of cameras, including off-vehicle, during training. During evaluation, agents will only have access to RGB images from cameras placed on the front, right, and left of the vehicle as well as pose information.

```python
observation =
{
  'CameraFrontRGB': front_img, # numpy array of shape (width, height, 3)
  'CameraLeftRGB': left_img, # numpy array of shape (width, height, 3)
  'CameraRightRGB': right_img, # numpy array of shape (width, height, 3)
  'track_id': track_id, # integer value associated with a specific racetrack
  'pose': pose_array # numpy array of shape (30,), more detail below
}
```

```yaml
0: steering request
1: gear request
2: nearest track index
3-5: direction velocity in m/s
6-8: directional acceleration in m/s^2
9-11: directional angular velocity
12-14: vehicle yaw, pitch, and roll, respectively
15-17: center of vehicle coordinates in the format (y, x, z)
18-21: wheel revolutions per minute (per wheel)
22-25: wheel braking (per wheel)
26-29: wheel torque (per wheel)
```

## Evaluation

### Pre-Evaluation Stage

All submissions will have access to the unseen evaluation racetrack for 1-hour with unfrozen model weights. Participants are free to perform any model updates or exploration strategies that they choose. Submissions that fail to begin the evaluation stage within 1-hour of starting pre-evaluation will not be accepted.

### Evaluation Stage

The evaluation stage consists of *3* environment episodes beginning at the finish line of the evaluation racetrack. The official Learn-to-Race metrics will be recorded and displayed on the leaderboard following submission.

### Submission Instructions

#### Environment

Your submission will be run in an Ubuntu 18.04, [nvidia/cudagl](https://hub.docker.com/r/nvidia/cudagl) Docker image with Cuda 11.0.3 drivers. By default, a Python 3.6.9 environment is used, and we expect submissions to use Python3 versions >= 3.6.9. The evaluation structure allows for a variety of Python3 environments:

* For conda environments, include `environment.yml` in the top directory of your submission
* For pip3 installation, include `requirements.txt` in the top directory of your submission, run after the above step
* For Python3 virtual environments, include a directory named `venv` in the top directory of your submission, and our script will activate `venv/bin/activate`

#### File Structure

You submission will be a single file named `submission.zip` which contains:

```bash
.
│   # required files
├── eval.py
├── conf.yml
│
│   # optional environment files & directories
├── environment.yml
├── requirements.txt
├── venv
│   ├── bin
│       ├── activate
│        ...
│   ├── include
│   ...
│
│   # optional additional files
├── (model weights)
├── (additional utilities)
└── ...
```

#### eval.py

This is the primary file that you will modify and include in your submission. The [template](https://github.com/hermgerm29/learn-to-race/blob/main/l2r/eval/eval.py) you will use is located in `l2r/eval/eval.py` in the Learn-to-Race repository. You are free to modify any methods that are marked **"Modify this method"**, and you will, at minimum, need to modify the following methods:

* `Evaluator.load_agent` to create your agent and load in policy weights
* `Evaluator.pre_evaluate` to define your 1-hour pre-evaluation phase during which you will likely want to take gradient updates and perform exploration and fine-tuning on the new racetrack

A variety of mechanisms are implemented to strictly check that **"Do not modify"** methods are not modified, and submissions that do modify these methods will result in a thrown exception and no provided score.

#### conf.yml

The other required file is `conf.yml` which defines a number of configurations for the racing environment. A template of this file is shown below. We also impose a few modest restrictions including:

* You must include the keys `env_kwargs` and `sim_kwargs` in the configuration
* You cannot use segmentation cameras during pre-evaluation or evaluation
* We impose a maximum limit of 1080 pixels for a single camera dimension
* A maximum observation delay of `0.15` seconds

Furthermore, we recommend:

* Not modifying any of the interface addresses
* Removing cameras that you don't intend on using

```yaml
env_kwargs:
  multimodal: True
  max_timesteps: 500
  obs_delay: 0.1
  not_moving_timeout: 50
  reward_pol: 'default'
  reward_kwargs:
    oob_penalty: 5.0
    min_oob_penalty: 25.0
  # Our script will replace controller_kwargs with the appropriate settings
  controller_kwargs:
    sim_version: 'ArrivalSim-linux-0.7.0-cmu4'
    quiet: True
    user: 'ubuntu'
    start_container: False
    sim_path: '/home/ArrivalSim-linux-0.7.0.182276/LinuxNoEditor'
  action_if_kwargs:
    max_accel: 6.
    min_accel: 3.
    max_steer: 0.2
    min_steer: -0.2
    ip: '0.0.0.0'
    port: 7077
  pose_if_kwargs:
    ip: '0.0.0.0'
    port: 7078
  cameras:
    CameraFrontRGB:
      Addr: 'tcp://0.0.0.0:8008'
      Format: ColorBGR8
      FOVAngle: 90
      Width: 512
      Height: 384
      bAutoAdvertise: True
    CameraLeftRGB:
      Addr: 'tcp://0.0.0.0:8009'
      Format: ColorBGR8
      FOVAngle: 90
      Width: 512
      Height: 384
      bAutoAdvertise: True
    CameraRightRGB:
      Addr: 'tcp://0.0.0.0:8010'
      Format: ColorBGR8
      FOVAngle: 90
      Width: 512
      Height: 384
      bAutoAdvertise: True

sim_kwargs:
  racetrack: ['VegasNorthRoad']
  driver_params:
    DriverAPIClass: 'VApiUdp'
    DriverAPI_UDP_SendAddress: '0.0.0.0'
```


### Metrics

Vestibulum bibendum, enim vitae scelerisque aliquam, nisi augue cursus leo, eget dignissim eros ex sit amet dolor. Donec risus ex, luctus id orci ut, ultricies accumsan sem. Vivamus eget semper diam. Duis eget purus malesuada, efficitur orci rhoncus, ultricies enim. Maecenas eu feugiat augue. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia curae; Praesent non lectus risus. Integer dictum consectetur urna. Sed porta dolor faucibus eros scelerisque, sit amet egestas magna fringilla. Nunc a cursus mi. Vivamus ut lectus nunc. Sed sit amet leo nibh. Ut dignissim eleifend suscipit.
