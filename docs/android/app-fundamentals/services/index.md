---
title: "创建 Android 服务"
description: "本指南讨论 Xamarin.Android 服务，允许进行不活动的用户界面工作的 Android 组件。 服务非常通常用于在后台，如耗时的计算，下载文件，播放音乐中执行任务，依次类推。 它说明服务适合的不同方案，并演示如何实现它们同时执行长时间运行的后台任务以及提供的远程过程调用的接口。"
ms.topic: article
ms.prod: xamarin
ms.assetid: BA371A59-6F7A-F62A-02FC-28253504ACC9
ms.technology: xamarin-android
author: topgenorth
ms.author: toopge
ms.date: 02/16/2018
ms.openlocfilehash: 5dc1fb0fb02014e123b3a161394155bde725f288
ms.sourcegitcommit: 0fdb243b46cf21be47584900805cadcd077121bf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/12/2018
---
# <a name="creating-android-services"></a>创建 Android 服务

_本指南讨论 Xamarin.Android 服务，允许进行不活动的用户界面工作的 Android 组件。服务非常通常用于在后台，如耗时的计算，下载文件，播放音乐中执行任务，依次类推。它说明服务适合的不同方案，并演示如何实现它们同时执行长时间运行的后台任务以及提供的远程过程调用的接口。_

## <a name="android-services-overview"></a>Android 服务概述

移动应用不像桌面应用中。 桌面都有大量数量的资源如屏幕的实际空间、 内存、 存储空间和已连接的电源，移动设备不这样做。 这些约束强制移动应用程序以不同方式行为。 例如，移动设备上的小屏幕通常意味着，只有一个应用程序 （即活动） 是可见的一次。 其他活动移动到后台，推送进入挂起状态，它们不能执行任何工作。 但是，只是因为 Android 应用程序是在后台并不意味着它是不是通过应用程序以继续工作。 

Android 应用程序由组成的至少一个以下四个主要组件：_活动_，_广播接收方_，_内容提供商_，和_服务_。 活动是许多卓越的 Android 应用程序的基础，因为它们提供用户界面，使用户与应用程序进行交互。 但是，当涉及到执行并发或后台工作，活动并不总是最佳选择。
 
在 Android 中的后台工作的主要机制是_服务_。 Android 服务是旨在执行某项操作没有用户界面的组件。 是服务可能下载的文件、 播放音乐，或将筛选器应用于映像。 服务还可用于进程间通信 (_IPC_) 之间 Android 应用程序。 例如，一个 Android 应用程序可以使用来自另一个应用程序的音乐播放器服务或应用程序可能会公开到其他通过服务的应用 （如个人的联系信息） 的数据。 

服务和其能够执行后台工作对提供的平滑、 流畅的用户界面至关重要。 所有 Android 应用程序具有_主线程_(也称为_UI 线程_) 上的活动都运行。 若要让设备保持响应状态，Android 必须能够更新每秒 60 帧速率的用户界面。 如果 Android 应用程序执行到主线程中，过多的工作，则 Android 将丢弃帧，这反过来会导致 UI 以显示平稳 (有时也称为_janky_)。 这意味着在 UI 线程上执行任何工作应该完成中两个帧，大约为 16 毫秒 （1，第二，个每 60 帧） 之间的时间跨度。 

若要解决此问题，开发人员可能使用线程在活动中执行某项操作会阻止 UI。 但是，这会导致问题。 它是 Android 将销毁，并重新创建多个活动实例的可能性很高。 但是，Android 自动不会破坏的线程，这样可能导致内存泄漏。 明显的例子是时[设备旋转](~/android/app-fundamentals/handling-rotation.md) &ndash; Android 会尝试销毁该活动的实例，然后重新创建一个新：

![设备旋转时, 销毁实例 1，并重新创建实例 2](images/image-01.png)

这是可能存在内存泄漏&ndash;仍在运行活动的第一个实例创建的线程。 如果线程可以对活动的第一个实例的引用，这将从收集对象的垃圾回收阻止 Android。 但是，该活动的第二个实例是仍创建 （这反过来可能会创建一个新线程）。 旋转几个场合中快速进行后续的设备可能会耗尽所有 RAM 和强制 Android 终止整个应用程序中，可以回收的内存。

根据经验，如果要执行的工作应生存期限超过活动，则服务应创建用于完成该工作。 但是，如果工作仅适用于活动的上下文，然后创建线程来执行的工作可能更合适。 例如，创建刚才添加的照片库应用到一张照片的缩略图应可能发生在服务中。 但是，一个线程可能更适合播放在前台活动时，仅能听到一些音乐。

后台工作可以分为两个广泛的分类：

1. **长时间运行任务**&ndash;这是为之前显式停止正在进行的工作。 一个示例_长时间运行的任务_流式处理音乐一个应用程序或，必须监视从传感器收集的数据。 即使应用程序具有没有可见的用户界面，必须运行这些任务。
2. **定期任务** &ndash; (有时称为_作业_) 定期任务是指的相对较短持续时间 （几秒钟） 和按计划运行 (即每天一次达一周或可能是中一次只下一步是 60 秒）。 此示例是从 internet 下载一个文件或生成图像的缩略图。

有四种不同类型的 Android 服务：

* **绑定服务** &ndash; A_绑定服务_是具有某些其他组件 （通常活动） 绑定到它的服务。 绑定的服务提供一个接口，用于绑定的组件和服务以进行相互交互。 后没有任何绑定到的服务的多个客户端，Android 将关闭该服务。

* **`IntentService`** &ndash;  _`IntentService`_ 是一个专门的子类的`Service`类，用于简化服务创建和使用情况。 `IntentService`旨在处理各个自治调用。 与一个服务，它可以同时处理多个调用，不同`IntentService`很像_工作队列处理器_&ndash;工作将会排队和`IntentService`在单个辅助线程上一次处理一个每个作业。 通常情况下，`IntentService`未绑定到活动或片段。 

* **已启动服务** &ndash; A_已启动服务_是已通过某些其他 Android 组件 （如活动） 启动，并且直到内容显式告知在后台运行持续进行的服务若要停止的服务。 绑定与服务不同，启动的服务没有任何客户端直接绑定到它。 出于此原因，很重要，以便它们才能正常重新启动根据需要设计启动的服务。

* **混合服务** &ndash; A_混合服务_是一项服务具有的特征_已启动服务_和_绑定服务_。 可以通过启动混合服务，当绑定到它的一个组件或某些事件可能启动它。 客户端组件可能或可能未绑定到混合服务。 混合服务将保持运行，直到被显式告知若要停止，或绑定到它没有更多客户端。

哪种类型以使用是服务的非常依赖于应用程序的要求。 根据经验，`IntentService`或绑定的服务足以满足大多数任务都必须执行 Android 应用程序，以便首选项应授予给服务这两种类型之一。 `IntentService`是不错的选择对于"单稳"任务，如下载文件，同时需要与活动/片段频繁交互时，绑定的服务即可。 

尽管大多数服务运行在后台，没有名为的特殊子类别_前景服务_。 这是一种服务，都有更高的优先级 （与普通服务比较） （如播放音乐） 的用户执行某些工作。 

也可以在同一设备上在它自己的进程中运行服务，这有时称为_远程服务_或_进程外服务_。 这确实需要更多工作来创建，但可用于应用程序需要共享功能与其他应用程序，并可以在某些情况下，提高应用程序的用户体验。 

所有这些服务都有其自己的特征和行为，并因此将其自己指南中的更详细地介绍。