# spotify-music-visualizer-exhibit
A music visualizer made in TouchDesigner that uses a custom Python integration with Spotify. 

[![Demo Video](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/573deef6-3940-41ec-b5a1-b0bdf9586b12)](https://www.youtube.com/watch?v=jmo1LZdjwf0)
> Please check out this demo video!

# Setup
Download the .toe file and open it in TouchDesigner. Then, [download the .tox file from my fork of this Python Spotify API integration](https://github.com/nick-cirillo/Banana_CurrentlyPlaying_Plus_AudioFeatures), drag that file into the project, and follow the setup instructions from that repository. It should be plug-and-play from there, but consider changing the `Period` parameter inside the `beat1` CHOP (Channel Operator) in the `BANANA_CURRENTLYPLAYING` extension to 4 or even 2 so that the visualizer updates faster. Also inspect the `window1` COMP in the main project area to make sure it goes fullscreen - try playing with some parameters here if not.

# Spotify API Integration
For a smooth experience, the visualizer operates not off preset tracks but from Spotify. To integrate Spotify with TouchDesigner, I found a [repository by jetXS](https://github.com/jetXS/Banana_CurrentlyPlaying) that links TouchDesigner with your Spotify account and submits an API request at intervals to retrieve the currently playing track. I [forked this repository and extended the functionality](https://github.com/nick-cirillo/Banana_CurrentlyPlaying_Plus_AudioFeatures) to also collect the track ID, and use this to submit another API request for the "Audio Features" of the track, provided by Spotify. These features detail different aspects of a song - a full list is available [here]([https://developer.spotify.com/documentation/web-api/reference/get-information-about-the-users-current-playback](https://developer.spotify.com/documentation/web-api/reference/get-audio-features)).

![Integration TouchDesigner Structure](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/2d9422a8-dd69-4955-a943-a8e7a819e6f8)
> The structure of the Spotify API Integration.



I use a pared down version of my integration fork that exclusively collects the track's `Energy`, `Tempo`, and `Valence` (whether the track evokes "positive" [happy, content, celebratory] or "negative" [sad, angry, paranoid] feelings). I use these traits to influence the music visualization, which I'll describe below.

# TouchDesigner Visualization
![Visualization TouchDesigner Structure](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/f8b3cc93-cdc4-482c-b88f-abb6d37aea22)

> Here's the structure of the visualization. I suggest following along with the text below!
## 3D Geometry
I begin with a Sphere SOP (Surface Operator), a 3D model of a sphere. In TouchDesigner, Operators are items that can link to other items and influence each other - they vary widely in function and form.

I linked this Sphere SOP to a Material SOP, which applies an image texture onto the surface of the sphere. However, there isn't yet have an image to apply!

Thus, I created a Movie File In TOP (Texture Operator), which allows me to add an image to the project. This is where I begin to draw on the Spotify integration - the Move File In SOP's input image is set to `op.BANANA_CURRENTLYPLAYING.Albumcover`, which reaches into the Spotify integration and gets the album cover of the currently playing track (or nothing at all if the playback is paused).

I use a Blur TOP to apply a dramatic Gaussian blur to the image, thus reducing it to its colors and vague shape, and apply this to a Constant MAT (Material), which is then applied to the Material SOP, thus giving the sphere a texture based on the album cover of the track. The Material SOP is linked to a Texture SOP, which completes the process.

Next, our textured sphere - represented farthest downstream by the Texture SOP - needs to twist, protrude, bubble, spike, and change with the music! I do this using a Noise SOP, which applies a noise function (the `Alligator` noise option, similar to Worley noise) to the sphere and deforms it over time. To differentiate between tracks, I introduce the Audio Features of the track to change the parameters of the noise function. I change the `Period` using `op.BANANA_CURRENTLYPLAYING.Valence` to change the "spikey/smooth" aspect of the deformation - lower valence means spikier visualizations, in line with the more upset and angry emotions associated with low valence, and higher valence leads to smoother, bubblier shapes, reflecting the more calm or happy emotions associated.

I change the `Exponent` using the `op.BANANA_CURRENTLYPLAYING.Tempo`, doing a simple linear interpolation to map the values backwards - a lower tempo leads to a slightly higher exponent. This has the effect of "slowing down" the visualization's changes, or making them less excitable, which I thought was appropriate to convey a sense of tempo. Having a beat-by-beat, moment-by-moment visualization would require fetching the [Audio Analysis](https://developer.spotify.com/documentation/web-api/reference/get-audio-analysis) of a track and using the tatums/beats/segments to change the image - a very interesting project for another time! Thus, I thought generally correlating tempo and the rate-of-change of the visualization was a good compromise.

I then change the `Amplitude` of the noise using the `op.BANANA_CURRENTLYPLAYING.Energy` of the track. More energetic tracks have more bombastic spikes or bubbles in the visualization! Calmer tracks stay closer to the spherical shape. I'll provide some examples at the end to show how all the different Audio Features affect the visualization.

## Background
All this thus far is affecting the 3D sphere I introduced. This does not address the background image! To add an interesting background, I went back to the Movie File In TOP, and introduced a Flip TOP to flip the image along its x-axis. I then used the existing Blur TOP of the upright image, along with a new Blur TOP of the flipped image with the same parameters, and used a Composition TOP to combine the images by averaging them. In this Comp TOP I also included some 2D Simplex noise through a Noise TOP. I zoomed far into the noise, and made sure the spots of black and white were dramatic by adjusting the parameters. I then applied a change-over-time to the Z-axis of the 2D noise, which visually changes the noise gradually, and averaged this into the Comp TOP. This results in a shifting field of colors taken from the album cover, which goes nicely with the deforming geometry.

## Text
I also added a Text TOP that displays the track name and artist name in a nice font with some transparency. It wraps at the edge of the screen in case of very long track names ([which did indeed come up in testing](https://open.spotify.com/track/6xKQQTR4pQVAKok4J4lzMt)).

## Manual Adjustments
I found in testing that certain tracks had their BPM calculated at half tempo, for example, [Flight of the Bumblebee](https://open.spotify.com/track/4PyCoafhG3Up9kbDlEOAsE) was calculated at roughly 90 BPM - but the visualization plods along at this tempo. Thus, I added a `Double Time` button that, when pressed, doubles the tempo of the track when calculating the `Exponent`. This brought the visualization closer to the brisk speed of the piece.

I also found that some visualizations were simply too large for the screen! I lowered the near-clip plane to 0.001 and increased the FOV of the camera, but some songs - like [Chop Suey!](https://open.spotify.com/track/2DlHlPMa4M17kufBvI2lEN?si=6a43ec66fb5347cd) - still punch the viewer with huge humps of noise. Thus, I introduced a `Quieter` button, which changes the `Harmonics` parameter of the noise from 0 to 1 when it is pressed. This dampens the whole visualization, which is great for intense tracks like this, but reduces ballads like [Lovers Rock](https://open.spotify.com/track/6dBUzqjtbnIa1TwYbyw5CM?si=69285c1ed6e6434e) to practically nothing, hence why a toggle was more useful than hard-coding the `Harmonics` value at either 0 or 1.

## Wrapping Up
Now that all the visuals are in place, I sent the Noise SOP to a Render SOP, which also takes a camera, allowing me to view the shape in a Window COMP (Component). First, though, I take the Render SOP and overlay it onto our background Comp SOP in a new, third Comp SOP. I send this final composition to a Window COMP, and mark this as the performance window, so that when I enter performance mode, it will display the visualization in fullscreen (after adjusting a few parameters).

# Examples
![Nothing PLaying](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/8fe68307-08fd-431a-b001-8f39af150ab6)
> Here's what it looks like when nothing is playing. Ominous.

<img width="1920" alt="6rsdlSg2dH" src="https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/46043c0b-512d-4390-95fa-2ef6f9e1b0c1">

> Here's [Lovers Rock](https://open.spotify.com/track/6dBUzqjtbnIa1TwYbyw5CM?si=69285c1ed6e6434e) by TV Girl. The visualization returns to the basic sphere often, and feels like it's trying to break out of a shell, which fits the song nicely.

![scCQ5Bw0Ju](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/e3d6c02f-b311-4501-a286-8a0e1374206f)
> This is [Symphony No. 10 in E Minor, Op. 93: II. Allegro](https://open.spotify.com/track/6oPWvsZ7dDafTPhi2kBJnG?si=eadc2a8b50f84717) by Dmitri Shostakovich, performed by the Royal Scottish National Orchestra. In this visualization, the sphere silhouette remains visible at all times, but the jagged edges protrude out and remain for a few seconds before retracting, highlighting the dynamic of alternating restraint and attack the piece has.

<img width="1920" alt="fC1VM8ldnl" src="https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/b71244e5-c64b-44df-bad6-7750e23ab02c">
> Here is [Shanghai](https://open.spotify.com/track/6HlWKkzNE1WP67OGqtGeBW?si=29b1c1cf4b654e9f) by King Gizzard and the Wizard Lizard. It's an infectious, cheery, bubbly track, and the visualization matches this - big bubbles appear from nowhere, sometimes jumping right at you, never going exactly where you expect - and disappearing in an instant!

![Hsi76fLtMq](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/26819f3e-18b6-4311-9b1b-524fd0d9b69a)
> Now for [Black Paint](https://open.spotify.com/track/2yY0LXGpN7U2y5tbagNnXq?si=0fe5ca7b4f574166) by Death Grips. It's an ugly mess of giant crystalline spikes, one of which jumps at you with the gross mouth on the original album art. It's spontaneous and harsh, much like the song, and the spikes will assault you unexpectedly - much like the music does.

![RAaKvQ1F3M](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/4055cf35-884a-4650-9e30-d990a7795c11)
> [Strawberry Fields](https://open.spotify.com/track/3Am0IbOxmvlSXro7N5iSfZ?si=f5e79963d32f4259) by The Beatles lands somewhere between Shanghai and Lovers Rock, but with an added jagged edge to the bubbles. It's more random and willing to take risks than Lovers Rock, but never as unreservedly bubbly as Shanghai - but what it is is undulating and colorful, trippy, just like the source material.

![SO344Ue0n5](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/388be108-0450-43d7-b729-afc47a3a4926)
> The spikiest one yet, [BFG Division](https://open.spotify.com/track/4COR2ZPEyUn0lsbAouRWxA?si=c55e25c7fea340a3) by Mick Gordon is violent, harsh, and unrelenting. It's not as gross as Black Paint, but it is more violent and angry. Therefore, more spikes, faster spikes, and spikier spikes! This is the most fitting one so far.

![ODSvTZStUg](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/41d9f1ef-a1a1-4a40-87ac-1bbbed9d47cb)
> [Nude](https://open.spotify.com/track/35YyxFpE0ZTOoqFx5bADW8?si=2cccfd1ad2d54250) by Radiohead has a cynical edge to it, with those crystalline spikes, but they aren't moving so fast, and aren't willing to go so far. In motion, they're shaking and cowering back into the sphere. It's a much more reserved song, but still has a bite. And the colors are beautiful!

![i5zpJl7lbX](https://github.com/nick-cirillo/spotify-music-visualizer-exhibit/assets/77818146/932fc102-b3d1-40e1-8c99-ee3d51938d80)
> [RETURN OF EBIC: RISE OF THE RED MIST](https://open.spotify.com/track/6CgYe6LUAT3BmQuz7eqKI3?si=375ef4a68ea64832) by d-d-d-dit. This one is about as bizarre as the source material.

