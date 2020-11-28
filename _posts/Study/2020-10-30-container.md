---
layout: post
title: "Introduction to COntainers, VMs, and Docker"
date: 2020-10-30 00:00:00
excerpt: ""
tags:
- study
categories:
-
---


# Introduction to Container, VMs, and Docker

이 글은 [A Beginner-Friendly Introduction to Containers, VMs and Docker](#https://medium.com/free-code-camp/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b) 를 번역한 것입니다.

당신이 프로그래머라면, 최소한 *"container"* 를 이용해 어플리케이션을 실행하고, packing & shipping하는 Docker 에 대해 들어볼 기회가 있었을 것입니다.
최근 개발자, 시스템 관리자까지 모두 관심을 갖고있는 Docker를 들어보지 못하긴 어려울 것입니다. 심지어 Google, VMWare, Amazon같은 대기업들도 그것을 지원하기 위해 서비스들을 구축중입니다.

Docker를 당장 사용하느냐에 관계없이, *container* 가 무엇인지에 대한 기본적인 개념과 그것이 가상머신과 어떻게 다른지에 대해 이해하는것은 중요하다고 생각합니다.

인터넷에 Docker에 대한 좋은 가이드들이 많이 있지만, Docker에 처음 접근하는 입문 가이드 들은 찾지 못하였습니다. 이 글이 문제를 해결할 수 있을 것이라고 희망합니다.

가상머신과 container가 무엇인지 이해하는 것으로 시작합시다.

### What are "containers" and "VMs"?
Containers 와 VMs는 비슷한 목표를 가지고 있습니다. 그것은 어플리케이션과 의존성 들을 하나의 어디서든 실행 가능한 유닛으로 분리합니다.

게다가, 두 개념 모두 물리적인 하드웨어를 필요로 하지 않습니다. 이것은 에너지 소모 측면과 비용 효율의 관점에 있어 컴퓨팅 자원을 사용하는데 더욱 효율적입니다.

둘의 차이는 구조적인 접근방법에 있습니다.

### Virtual Machines
VM는 근본적으로 진짜 컴퓨터처럼 프로그램을 실행하게 하는 emulation입니다.
VM은 *hypervisor*를 이용해 물리적 시스템 위에서 실행됩니다. *hypervisor*는 또한 호스트 머신 또는 "bare-metal" 에서 실행됩니다.

*hypervisor* 란 VM이 실행되기 위한 소프트웨어, 펌웨어 또는 하드웨어입니다. 그 자체는 *host machine* 이라고 부르는 물리적 컴퓨터 위에서 실행됩니다. *host machine* 은 VM에 RAM과 CPU를 포함한 자원을 할당합니다. 이 자원들은 VM간에 나뉘어져 있으며 적합한 대로 분배될 수 있습니다. 즉, 만약 VM이 더 많은 리소스를 요구하는 무거운 프로그램을 실행한다면, 같은 *host machine* 에서 실행되는 VM들 중 해당 VM에 더 많은 자원을 할당할 수 있습니다.

*host machine* 위에서 실행되는 VM은 또한 *guest machine* 이라고도 불립니다. 이 *guest machine* 은 어플리케이션과, 어플리케이션을 실행하는데 필요한 것들(시스템 바이너리, 라이브러리 등)을 포함합니다. 또한 가상화된 네트워크, 스토리지, CPU를 포함한 가상화된 하드웨어 스택 전체를 가지며, 그것은 자체적인 독립가능한 게스트 OS를 가지고 있음을 의미합니다. 내부에서, *guest machine* 은 전용 리소스를 가진 하나의 유닛으로 작동합니다.

위에서 말했듯이, *guest machine* 은 hosted hypervisor 또는 bare-metal hypervisor 위에서 실행됩니다. 이 둘은 중요한 차이점이 있습니다.

첫번째로, hosted hypervisor는 *host machine* 의 OS 위에서 실행됩니다. 예를 들어, OSX를 쓰는 컴퓨터는 해당 OS 위에 설치된 VM을 가질 수 있습니다. VM은 직접적으로 하드웨어에 접근하지 않지만, *host machine* 의 OS를 거쳐야 합니다.

이런 hosted hypervisor의 장점은, 하드웨어가 덜 중요하다는 것입니다. hypervisor가 아닌 호스트의 OS가 하드웨어 드라이버를 담당하게 됩니다. 그리고 이것은 "hardware compatibility"를 가진다고 여겨집니다. 반면에, 이런 추가적인 하드웨어와 hypervisor 간의 레이어는 VM의 성능저하를 일으키는 리소스 오버헤드를 만듭니다.

반면 bare metal hypervisor는 호스트 머신의 하드웨어를 설치하고 실행함으로써 이러한 성능 이슈를 해결합니다. hypervisor가 직접 하드웨어와 상호작용하기 떄문에, 호스트 OS가 실행될 필요가 없습니다. 이 경우에, OS로써 호스트머신에 처음 설치되는 것이 hypervisor가 될 것입니다. hosted hypervisor와 달리, bare-metal은 자신만의 디바이스 드라이버를 가지고 I/O, 프로세싱, OS 작업 등을 직접적으로 상호작용합니다. 이것은 더 좋은 성능, 확장성, 안정성을 가져옵니다. 여기서의 tradeoff는 hypervisor가 많은 디바이스 드라이버를 가질 수 있기 떄문에 hardware compatibility가 제한된다는 것입니다.

왜 이런 *hypervisor* 레이어가 VM과 host machine 사이에 필요한지 궁금할 것입니다.

VM은 자체적인 virtual OS를 가지고 있기 때문에, hypervisor는 VM에 이 OS를 실행하고 관리할수 있는 플랫폼을 제공하는데 있어 필수적인 역할을 합니다.

![vm](https://miro.medium.com/max/1400/1*RKPXdVaqHRzmQ5RPBH_d-g.png)

위의 그림애서, VM들은 새 VM의 사용자 공간, OS, 가상 하드웨어를 패키징 합니다.


### Container
