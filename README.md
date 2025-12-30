# stereoscopic-livestream-simple
A simple demo of high-quality, configurable streaming of a stereoscopic camera.

Currently this system is a just a modified verson of the [WebXR Stereovideo Sample](https://immersive-web.github.io/webxr-samples/stereo-video.html) ([source](https://github.com/immersive-web/webxr-samples/blob/main/stereo-video.html)) It is modified to accept a video stream from a [MediaMTX](https://github.com/bluenviron/mediamtx) server instead of just using a static video as the source. 

> These instructions are primarily designed for a linux environment (for me, NixOS btw) - but if you are on Windows it should still be possible, you might just have to change a few commands. 

# Prerequisites
- [Python 3](https://www.python.org/)
- [OBS](https://obsproject.com/)

# Setup

1. Clone the repository: 

###### Linux
```
git clone git@github.com:bluenviron/mediamtx.git
```

###### Windows
```
git clone https://github.com/bluenviron/mediamtx.git
```

2. Start the webserver

```
cd webapp
python -m http.server
```

3. Set up the MediaMTX Server 
    1. Download: Get the latest release of [MediaMTX](https://github.com/bluenviron/mediamtx/releases) for your OS (Windows/Linux).
    2. Run it: Unzip and run the executable (in a seperate terminal to the one you are running the python http server). By default, it will start listening for streams.
    3. Find your IP: Open your terminal/command prompt and type `ipconfig` (Windows) or `ifconfig` (Mac/Linux) to find your local IP address (e.g., 192.168.1.50).

> Note: I have included MediaMTX v1.15.5 in this repository under `/mediamtx_v1.15.5_linux_amd64/` (along with my modified `mediamtx.yml` config file) for easy access. However, I would imagine that it is best practice to just download the latest official release. 

> Another note: You might need to modify the `mediamtx.yml` config file to muck around with the ports that the MediaMTX server is listening on / serving to. It told me that there was a port conflict with the one that the python server was hosting on (port 8000), so I changed the YAML file untill it stopped complaining.

4. Configure OBS to Stream to MediaMTX
    1. Plug in the stereoscopic camera and make sure it shows up in the OBS preview (make a scene, make a Video Capture Device source etc. etc..). My stereoscopic camera outputs its two cameras as "Side By Side" or "stereoLeftRight". If you video comes out of the camera stacked on top of each other - then edit `index.html` so that it is set to `stereoTopBottom`.
    2. Go to Settings > Stream.
    3. Service: Select Custom...
    4. Server: rtmp://localhost/live
    5. Stream Key: mystream
    6. Go to Settings > Output:
    7. Output Mode: Advanced
    8. Tune: zerolatency (Crucial for live VR!)
    9. Keyframe Interval: 2 s
    10. Click Start Streaming.

5. Update the web server code
    1. Change line `99` of `index.html` to your server's local IP address (replace `192.168.1.36` below with your server's local IP). You might also need to change the port from `8888` if you have a different `mediamtx.yml` file config to me. I think the default is `8000` so maybe try that first if you are unsure.
```
const streamUrl = 'http://192.168.1.36:8888/live/mystream/index.m3u8';
```

6. Restart the webserver after updating this. Press `ctrl-c` then run the following command again

```
python -m http.server
```

> Yes we could have just not run the server untill now - but for some reason I wanted to run the server as the first step cause it felt natural. You do what you like. I will maybe make these instructions more efficient later. 

7. Okay, with any luck, the video will now stream from OBS, to the MediaMTX server to the webapp. Check to see this works on a web browser on the same computer as your server first (I use firefox). 

8. I used a Meta Quest 1 to test the actual VR functionality of this system. Therefore, this step, and all future steps speak to the Quest 1. Although, as far as I know, all current VR headsets from Meta, Apple and Valve (Steam Frame coming soon!!) support WebXR, and therefore should support this app. Its just that these next steps are probably just a Meta (Facebook) thing. Anyway, step 8 is just to log into the webapp on your VR device. For me, it is as simple as typing `http://192.168.1.36:8000` into the native Quest 1 web browser. The page loads! If it doesn't, try seeing if there is a firewall on your native machine. If there is you will need to allow traffic from ports `80, 8000, 8080, 8888` etc. etc.
    - For me, the page loaded, but the WebXR functionality didn't yet work. If it doesn't for you too, move onto step 9.

9. See, the Meta Quest native browser is built on Chromium. And as far as I can tell, Chromium doesn't let webapps use the WebXR protocol unless they are secure (`https`) our app is not secure (`http`). So the easiest way around this was to do the following:
    1. Open the Meta Quest Browser on your headset.
    2. In the address bar, type: chrome://flags/#unsafely-treat-insecure-origin-as-secure
    3. Find the section titled "Insecure origins treated as secure".
    4. Change the dropdown from Disabled to Enabled.
    5. In the text box below it, enter your full local address (e.g., `http://192.168.1.36:8000`). Note: If you have multiple apps, you can separate URLs with commas.
    6. Click the Relaunch button at the bottom of the screen.

10. Everything should now work perhaps. 

> At current, I haven't been able to reduce the realtime delay of the stereoscopic stream to be under 3 seconds. I am guessing more fiddling with the OBS stream settings might do the trick. Enjoy watching youself in semi-realtime 3D!
