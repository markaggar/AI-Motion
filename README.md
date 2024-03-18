# Home Assistant AI Motion Template Sensor
Home Assistant Template Sensor for Cameras with AI and Motion to prevent false positives and sustain the detection time if repeated motion is detected.

I was tired of getting woken up in the middle of the night by Home Assistant when my Reolink cameras would detect a person outside due to false positives.  What I noticed is that these AI person detections would rarely if ever coincide with a motion event.
Also, I wanted a way for my indoor cameras to be used as motion sensors for lighting.  However, I didn't want the motion of my dog to set them off, so person detection was important. However, unless the person is fully visible, the person detection wouldn't trigger.  However, motion would frequently trigger. This meant that a person coming into the room would trigger the lights to be turned on, but it wouldn't be sustained, and I'd have to rely on motion events to keep the lights on.

As such I built this template sensor which is easy to adjust for your specific AI and motion sensors.

![image](https://github.com/markaggar/AI-Motion/assets/25288127/2e2251af-4561-42b5-8e8c-932cc1eb733c)

As you can see there are a lot of sensor attributes which power the sensor.  In particular, I developed a technique to store the previous time the motion sensor was on such that I can calculate the time delta between 'motion on' events to determine whether the sensor should stay on if there are a succession of motion events after the AI person event is triggered.  If there is a motion event within the delta time, the AI motion sensor will stay on even if there hasn't been another person event in hours.  Obviously if your motion sensor is set to highly sensitive and there is a wind storm or something that causes a lot of motion to be detected, a person detection event could leave the sensor on for a long time erroneously - you've been warned.

## Configuration
There are three configuration options which you edit directly in the template sensor.  These are in the attributes section near the top of the sensor template.

### Sensor names
The first two are for the pair of sensors/binary_sensors (AI and motion) you want to use.

       attributes:
         motion: >
           {{states("binary_sensor.family_room_motion")}}  # Name of the motion sensor**
         ai: >
           {{states("binary_sensor.family_room_person")}}  # Name of the AI sensor (could be person, pet etc)**.

### Delta time
The third is the delta time between motion events.  This can be configurable in that it can have some logic to determine what the timeout should be in certain circumstances.

The simple version is just to have a fixed value

         delta: 300 **# Maximum time between motion events before the sensor turns off**

This is a more complex setting which sets the timeout based on whether the TV in the room is on or off.  If the TV is on, we typically aren't moving much 
         
         delta: >
          {% if states("media_player.family_room_tv") != "off" and states("media_player.family_room_tv") != "unavailable" %}
             900
          {% else %}
             300
          {% endif %}

### Avoiding template loops
Given Home Assistant's propensity to complain about template loops, the sensor is configured as a trigger sensor.  You should include all of the sensors/binary sensors for all of your template sensors here.  The sensor is also updated every 15 seconds.

        - trigger:
          - platform: time_pattern
            seconds: "/15"
          - platform: state
            entity_id:
              - binary_sensor.family_room_motion # name of your motion, AI sensors and any other sensors you are using to determine delta timeouts**
              - binary_sensor.family_room_person
              - binary_sensor.kitchen_motion
              - binary_sensor.kitchen_person
              - media_player.tv_family_room

Download the full template sensor here:


