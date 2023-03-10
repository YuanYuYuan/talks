---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# page transition
transition: slide-left
# use UnoCSS
css: unocss
---

# Zenoh Bridge DDS

2023-03-10

---
layout: image-right
image: https://images.unsplash.com/photo-1517247229957-6ae724c93b55?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=687&q=80
---

# Outline

- Why we need to study the zenoh-plugin-dds?
- The probelms of V2V communication behind the DDS.
- What benefits does the zenoh-plugin-dds provide?

---

## Autoware

![](https://static.wixstatic.com/media/984e93_552e338be28543c7949717053cc3f11f~mv2.png/v1/crop/x_0,y_1,w_1500,h_879/fill/w_863,h_506,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/Autoware-GFX_edited.png)

---
layout: two-cols
---

## Pros

- Perfect for wired LAN
- Highly customizable with fine QoS settings
- Low latency

## Cons

- Does not work over the internet
- Designed to work within a network with low message loss
- DDS storm can corrupt the wireless network
- Push-mode only

::right::

# DDS
Data Distribution Service

![](pic/dds.png)

---

# Complexity of DDS Discovery
<br>

DDS discovery mechanism relies on

- **SPDP (Simple Participant Discovery Protocol)**, which is used to find who domain participants
- **SEDP (Simple Entity Discovery Protocol)**, which is used to exchange the information of data readers, data writers, and topics.

For $n$ participants each of which has $r$ readers and $w$ writers, the traffic scales with $n(n-1)(r+w)$.

---

# How Does Zenoh Solve Discovery Problem?

<br>

1. Only resource interests are advertised.
2. Resource interests can be generalized to compress discovery data.
3. Discovery message are extremely wire efficient.
4. The reliability (QoS) is set between runtimes instead of per entity as in DDS.


---

# How does Zenoh-plugin-DDS work?

<br>
<br>

```mermaid
flowchart LR
subgraph Machine1
    subgraph RosNode1
        A1(Pub)
        B1(DW)
    end
    subgraph Bridge1
        C1(DR)
        D1(put)
    end
end
subgraph Machine2
    subgraph RosNode2
        A2(Sub)
        B2(DR)
    end
    subgraph Bridge2
        C2(DW)
        D2(sub)
    end
end
A1 --> B1 -- DDS --> C1 --> D1 -- zenoh --> D2 --> C2 -- DDS --> B2 --> A2
```
* DW/DR: DDS DataWriter & DataReader

---

# Fleet Management

Commnuication can go through wireless environment

<br>

```mermaid
flowchart LR
subgraph Vehicle1
    subgraph AU1[Autoware]
        A1(ROS2 Topic\n/location)
        B1(DDS Topic\n/rt/location)
    end
    C1(zenoh-dds-bridge \n--scope Vehicle1)
end
subgraph Vehicle2
    subgraph AU2[Autoware]
        A2(ROS2 Topic\n/location)
        B2(DDS Topic\n/rt/location)
    end
    C2(zenoh-dds-bridge \n--scope Vehicle2)
end
D(Zenoh Topic\n**/rt/location)
A1 --> B1 --> C1 --> D
A2 --> B2 --> C2 --> D
```

---

# Fleet Management

Leverage cooperative sensing with V2V Communication

<br>

```mermaid
flowchart LR
subgraph Vehicle1
    subgraph AU1[Autoware]
        A1(ROS2 Topic\n/perception)
        B1(DDS Topic\n/rt/perception)
    end
    C1(zenoh-dds-bridge \n--scope Vehicle1)
end
subgraph Vehicle2
    subgraph AU2[Autoware]
        A2(ROS2 Topic\n/perception)
        B2(DDS Topic\n/rt/perception)
    end
    C2(zenoh-dds-bridge \n--scope Vehicle2)
end
D(Zenoh Topic\n**/rt/perception)
subgraph Vehicle3
    subgraph AU3[Autoware]
        A3(ROS3 Topic\n/speed)
        B3(ROS3 Topic\n/direction)
        D3(ROS3 Topic\n/perception)
    end
    C3(zenoh-dds-bridge \n--scope Vehicle3)
end
A1 --> B1 --> C1 --> D
A2 --> B2 --> C2 --> D
D --> C3 --> D3 --> B3
D3 --> A3
```

---

### Benchmark Report from OpenRobotics

![](pic/mont-blanc.png)

---

### Benchmark Report from OpenRobotics

![](pic/discovery-ros-vs-zenoh.png)

---


# Comparison

|                      | **DDS** | **Kafka** | **MQTT** | **Zenoh** |
| -                    | -       | -         | -        | -         |
| Pub/Sub              | Push    | Pull      | Push     | Push/Pull |
| High Performance     | ✗       | ✗         | ✗        | ✔         |
| Distributed Query    | ✗       | ✗         | ✔        | ✔         |
| Constrained Hardware | ✗       | ✗         | ✔        | ✔         |
| Constrained Networks | ✗       | ✗         | ✗        | ✔         |
| Internet Scale       | ✗       | ✔         | ✔        | ✔         |
| Standard Based       | ✔       | ✗         | ✔        | ✗         |
| QoS                  | ✔       | ✗         | ✔        | ✔         |
| Peer-to-peer         | ✔       | ✗         | ✗        | ✔         |
| Brokered             | ✗       | ✔         | ✔        | ✔         |
| Routed               | ✗       | ✗         | ✗        | ✔         |

<style>
table {
    table-layout: fixed;
}
.slidev-layout td, .slidev-layout th {
    padding-top: 0.1rem;
    padding-bottom: 0.1rem;
}
</style>

---

# Reference

- [Improving the communications layer of robot applications with ROS 2 and Zenoh by Geoffrey Biggs, Open Robotics](https://www.youtube.com/watch?v=1NE8cU72frk)
- [ROS2 Robot-to-Anything with Zenoh by Julien Enoch, ZettaScale](https://www.youtube.com/watch?v=9h01_MSKPS0)
- [Minimizing Discovery Overhead in ROS2](https://zenoh.io/blog/2021-03-23-discovery/)
- [FastDDS Discovery](https://fast-dds.docs.eprosima.com/en/v2.9.1/fastdds/discovery/simple.html)
