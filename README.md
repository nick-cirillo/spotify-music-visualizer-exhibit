# spotify-music-visualizer-exhibit
A music visualizer made in TouchDesigner that uses a custom integration with Spotify. 

# Spotify API Integration
For a smooth experience, the visualizer operates not off preset tracks but from Spotify. To integrate Spotify with TouchDesigner, I found a [repository by jetXS](https://github.com/jetXS/Banana_CurrentlyPlaying) that links TouchDesigner with your Spotify account and submits an API request at intervals to retrieve the currently playing track. I [forked this repository and extended the functionality](https://github.com/nick-cirillo/Banana_CurrentlyPlaying_Plus_AudioFeatures) to also collect the track ID, and use this to submit another API request for the "Audio Features" of the track, provided by Spotify. These features detail different aspects of a song - a full list is available [here]([https://developer.spotify.com/documentation/web-api/reference/get-information-about-the-users-current-playback](https://developer.spotify.com/documentation/web-api/reference/get-audio-features)).

I use a pared down version of my integration fork that exclusively collects the track's `Energy`, `Tempo`, and `Valence` (whether the track evokes "positive" [happy, content, celebratory] or "negative" [sad, angry, paranoid] feelings). I use these traits to influence the music visualization, which I'll describe below.

# TouchDesigner Visualization
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
Examples to come!
