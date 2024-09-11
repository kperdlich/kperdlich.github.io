---
layout: post
title: Unreal Engine 4 and FFmpeg 
---

### Concept Idea:
In 2021, I worked on a game concept that required a multiplayer replay system, similar to the KillCam in Call of Duty.
However, instead of the typical system that replays a kill from the player's perspective, I wanted players to be able to position a camcorder object (90's VHS aesthetic) 
anywhere in the game world to capture actions. Later, all players can view the recordings from in-game computers.

<iframe width="981" height="552" src="https://www.youtube.com/embed/VQO452whfBw" title="Project - FridayNight: DEV Concept Trailer (UE4 Horror Multiplayer)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Recording in Unreal Engine 4:
At the time, I was using Unreal Engine 4, which provides a built-in replay system by recording the engine’s replication layer.
However, the design of my game introduced unique challenges that a standard KillCam couldn’t address. 
The replay needed to be accessible in-game, allowing the player to view it while still remaining active in the world.
This meant the player could still interact with other players or even be killed while watching the footage. 
So I needed a solution that allowed replays to run without interrupting the live gameplay.

Reworking or extending the built-in replay system was one option, but it felt overly complex and potentially error-prone for what I needed.
Instead, I decided to do an actual recording by capturing the camcorder's render target, encoding the footage into a .mp4 file, and uploading it to a backend server, making it accessible to other players in the match. 
Doing this client-side poses some security risks and it isn’t the most robust approach, but it felt reasonable for my prototype. 
Another priority was ensuring game performance remained stable during the recording process.

To play the video in game, we can stream it directly from the backend server using Unreal Engine's Media Player. 
The backend server just needs an endpoint to serve the video for a given video ID. 
An example for C# ASP.Net Core can be found [here](https://learn.microsoft.com/en-us/answers/questions/726990/serving-video-file-stream-from-asp-net-core-6-mini).

For more information on Unreal Engine's Media Player, refer to [Unreal Engine's documentation on playing a video stream](https://dev.epicgames.com/documentation/en-us/unreal-engine/play-a-video-stream?application_version=4.27).


### Implementation via FFmpeg:
I integrated FFmpeg into the project for video encoding and set up a separate thread to handle the encoding process, ensuring the game thread remained unaffected and performance stayed stable.
The camcorder object provides a start and stop recording functions, which managed the entire recording flow:

{% highlight cpp %}
void ARecorder::StartRecording()
{
    if (bIsRecording)
        return;

    const FRenderTarget* RenderTarget = RecorderCaptureComponent->TextureTarget->
        GameThread_GetRenderTargetResource();

    ViewportSize = RenderTarget->GetSizeXY();

    static const FString SDirectory = FPaths::ProjectSavedDir();
    static const FString SFileTyp = TEXT("mp4");

    const FGuid Guid = FUniqueObjectGuid::GetOrCreateIDForObject(this)
        .GetGuid();

    Filename = FString::Printf(TEXT("%s_%d.%s"),
        *Guid.ToString(EGuidFormats::Base36Encoded),
        FDateTime::UtcNow().ToUnixTimestamp(),
        *SFileTyp);

    CurrentRecordingFilePath = *SDirectory + Filename;

    GetVideoCaptureManager()->StartCapturing(ViewportSize, CurrentRecordingFilePath);
    bIsRecording = true;
}
{% endhighlight %}

{% highlight cpp %}
void ARecorder::StopRecording()
{
    if (!bIsRecording)
        return;
    
    VideoCaptureDoneHandle = GameInstance->GetVideoCaptureManager()->
        OnVideoCaptureDone().AddUObject(this, &ARecorder::OnVideoCaptureDone);
    GetVideoCaptureManager()->StopCapturing();
}
{% endhighlight %}

{% highlight cpp %}
void ARecorder::OnVideoCaptureDone()
{
    GetVideoCaptureManager()->OnVideoCaptureDone().Remove(VideoCaptureDoneHandle);
    bIsRecording = false;
    
    static const FString& Url = TEXT("localhost:8080/videos");
    UploadCompleteDelegate.BindUObject(this, &ARecorder::OnRequestComplete);
    FNHttpClient::UploadMime(CurrentRecordingFilePath, Filename, Url, UploadCompleteDelegate);
}
{% endhighlight %}

{% highlight cpp %}
void ARecorder::OnRequestComplete(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bWasSuccessful)
{
    // Check HTTP response, parse upload result (JSON) and delete .mp4 file
}
{% endhighlight %}

The object will try to capture a frame every 33 ms via a timer:
{% highlight cpp %}
void ARecorder::CaptureFrame()
{
	ColorBuffer.Empty();
	FRenderTarget* RenderTarget = RecorderCaptureComponent->TextureTarget->GameThread_GetRenderTargetResource();

	if (!RenderTarget->ReadPixels(ColorBuffer, FReadSurfaceDataFlags(), FIntRect(0, 0, ViewportSize.X, ViewportSize.Y)))
	{
		UE_LOG(LogTemp, Error, TEXT("Cannot read pixels from Render target!"));
		return;
	}

	GetVideoCaptureManager()->EncodeFrame(ColorBuffer);
}
{% endhighlight %}

### Result:
With a small resolution for the camera’s viewport, the performance impact of the capture process was minimal.

Here's an in-game captured video by the system:
<video width="981" height="552" controls="controls">
  <source src="/images/3HCB25WTZCX3VF2VYQQAJ3TVT_1633870508.mp4" type="video/mp4">
</video>


