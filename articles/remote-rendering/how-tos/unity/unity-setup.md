---
title: Set up Remote Rendering for Unity
description: How to initialize Azure Remote Rendering in a Unity project
author: jakrams
ms.author: jakras
ms.date: 02/27/2020
ms.topic: how-to
ms.custom: devx-track-csharp
---

# Set up Remote Rendering for Unity

To enable Azure Remote Rendering (ARR) in Unity, we provide dedicated methods that take care of some Unity-specific aspects.

## Startup and shutdown

To initialize Remote Rendering, use `RemoteManagerUnity`. This class calls into the generic `RenderingConnection` but already implements Unity-specific details for you. For example, Unity uses a specific coordinate system. When calling `RemoteManagerUnity.Initialize`, the proper convention will be set up. The call also requires you to provide the Unity camera that should be used for displaying the remotely rendered content.

```cs
// initialize Azure Remote Rendering for use in Unity:
// it needs to know which camera is used for rendering the scene
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);
```

For shutting down Remote Rendering, call `RemoteManagerStatic.ShutdownRemoteRendering()`.

After an `RenderingSession` has been created and chosen as the primary rendering session, it must be registered with `RemoteManagerUnity`:

```cs
RemoteManagerUnity.CurrentSession = ...
```

### Full example code

The code below demonstrates all the steps needed to initialize Azure Remote Rendering in Unity:

```cs
// initialize Remote Rendering
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);

// create a frontend
SessionConfiguration sessionConfig = new SessionConfiguration();
// ... fill out sessionConfig ...
RemoteRenderingClient client = new RemoteRenderingClient(sessionConfig);

// start a session
CreateRenderingSessionResult result = await client.CreateNewRenderingSessionAsync(new RenderingSessionCreationOptions(RenderingSessionVmSize.Standard, 0, 30));
RenderingSession session = result.Session;

// let RemoteManagerUnity know about the session we want to use
RemoteManagerUnity.CurrentSession = session;

await session.ConnectAsync(new RendererInitOptions());

/// When connected, load and modify content

RemoteManagerStatic.ShutdownRemoteRendering();
```

## Convenience functions

### Session state events

`RemoteManagerUnity.OnSessionUpdate` emits events for when its session state changes, see the code documentation for details.

### ARRServiceUnity

`ARRServiceUnity` is an optional component to streamline setup and session management. It contains options to automatically stop its session when the application is exiting or play mode is exited in the editor, as well as automatically renew the session lease when needed. It caches data such as the session properties (see its `LastProperties` variable), and exposes events for session state changes and session errors.

There can't be more than one instance of `ARRServiceUnity` at a time. It's meant for getting you started quicker by implementing some common functionality. For a larger application it may be preferable to do those things yourself, though.

For an example how to set up and use `ARRServiceUnity` see [Tutorial: Viewing remotely rendered models](../../tutorials/unity/view-remote-models/view-remote-models.md).

## Next steps

* [Unity Render Pipelines](unity-render-pipelines.md)
* [Install the Remote Rendering package for Unity](install-remote-rendering-unity-package.md)
* [Tutorial: Viewing remotely rendered models](../../tutorials/unity/view-remote-models/view-remote-models.md)
