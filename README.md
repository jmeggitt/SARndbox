 # SARndbox

version 2.3
Copyright (c) 2012-2016 Oliver Kreylos

 ## Overview

The Augmented Reality Sandbox is an augmented reality application
scanning a sand surface using a Kinect 3D camera, and projecting a real-
time updated topography map with topographic contour lines, hillshading,
and an optional real-time water flow simulation back onto the sand
surface using a calibrated projector.

 ### Requirements
 - Vrui version 4.2 or newer
 - Kinect 3D Video Capture Project version 3.2 or newer

## Installation Guide

It is recommended to download or move the source packages for Vrui, the
Kinect 3D Video Capture Project, and the Augmented Reality Sandbox into
a src directory underneath the user's home directory. Otherwise,
references to `~/src` in the following instructions need to be changed.

 1. Install Vrui from `~/src/Vrui-<version>-<build>`. See [Vrui README](http://idav.ucdavis.edu/~okreylos/ResDev/Vrui/InstallationGuide.html).

 2. Install the Kinect 3D Video Capture Project from
    `~/src/Kinect-<version>`. See the [Kinect 3D Video Capture Project
    README](https://arsandbox.ucdavis.edu/technical-resources/downloads/kinect/).

1. Change into `~/src` directory and unzip the Augmented Reality Sandbox:
   > cd ~/src
   > tar xfz <download path>/SARndbox-<version>.tar.gz
   or
   > tar xf <download path>/SARndbox-<version>.tar

2. Change into the Augmented Reality Sandbox's base directory:
   > cd SARndbox-<version>

3. **If the Vrui version installed was not 4.2**, or Vrui's
   installation directory was changed from the default of `/usr/local`,
   adapt the makefile using a text editor. Add the following line to the begining of the `Makefile`.
   > VRUI_MAKEDIR := <Vrui install dir>/share/make

4. Build the Augmented Reality Sandbox using the command:
   > make

5. Install the Augmented Reality Sandbox in the selected target
   location. To install run the command `sudo make install`
   This will copy all executables into `<INSTALLDIR>/bin`, all
   configuration files into `<INSTALLDIR>/etc/SARndbox-<version>`, all
   resource files into `<INSTALLDIR>/share/SARndbox-<version>`, and all
   shader source codes into
   `<INSTALLDIR>/share/SARndbox-<version>/Shaders`.

 ## Use

The Augmented Reality Sandbox package contains the sandbox application
itself, SARndbox, and a calibration utility to interactively measure a
transformation between the Kinect camera scanning the sandbox surface,
and the projector projecting onto it. The setup procedure described
below also uses several utilities from the Kinect 3D video capture
project.

 ## Setup and Calibration

Before the Augmented Reality Sandbox can be used, the hardware (physical
sandbox, Kinect camera, and projector) has to be set up properly, and
the various components have to be calibrated internally and with respect
to each other. While the sandbox can be run in "trial mode" with very
little required setup, for the full effect the following steps have to
be performed in order:

1. *Optional* Calculate per-pixel depth correction coefficients for the
   Kinect camera. (trial and error)

2. *Optional* Internally calibrate the Kinect camera. (trial and error)

3. Mount the Kinect camera above the sandbox so that it is looking
   straight down, and can see the entire sand surface. Use
   `RawKinectViewer` from the Kinect 3D video capture project to line up
   the depth camera while ignoring the color camera.

4. Measure the base plane equation of the sand surface relative to the
   Kinect camera's internal coordinate system using RawKinectViewer's
   plane extraction tool. (See [Using Vrui Applications](http://idav.ucdavis.edu/~okreylos/ResDev/Vrui/InstallationGuide.html) in the Vrui documentation on how to use `RawKinectViewer`, and particularly on
   how to create / destroy tools.)

5. Measure the extents of the sand surface relative to the Kinect
   camera's internal coordinate system using KinectViewer and a 3D
   measurement tool.

6. Mount the projector above the sand surface so that it projects its
   image perpendicularly onto the flattened sand surface, and so that
   the projector's field-of-projection and the Kinect camera's field-of-
   view overlap as much as possible. Focus the projector to the
   flattened average-height sand surface.

7. Calculate a calibration matrix from the Kinect camera's camera space
   to projector space using the CalibrateProjector utility and a
   circular calibration target (a CD with a fitting white paper disk
   glued to one surface).

8. Test the setup by running the Augmented Reality Sandbox application.

 #### Step 1: Per-pixel depth correction

Kinect cameras have non-linear distortions in their depth measurements
due to uncorrected lens distortions in the depth camera. The Kinect 3D
video capture project has a calibration tool to gather per-pixel
correction factors to "straighten out" the depth image.

To calculate depth correction coefficients, start the RawKinectViewer
utility and create a "Calibrate Depth Lens" tool. (See "Using Vrui
Applications" in the Vrui HTML documentation on how to create tools.)
Then find a completely flat surface, and point the Kinect camera
perpendicularly at that surface from a variety of distances. Ensure that
the depth camera only sees the flat surface and no other objects, and
that there are no holes in the depth images.

Then capture one depth correction tie point for each distance between
the Kinect camera and the flat surface:

1. Line up the Kinect camera.

2. Capture an average depth frame by selecting the "Average Frames" main
   menu item, and wait until a static depth frame is displayed.

3. Create a tie point by pressing the first button bound to the
   "Calibrate Depth Lens" tool.

4. De-select the "Average Frames" main menu item, and repeat from step 1
   until the surface has been captured from sufficiently many distances.

After all tie points have been collected:

5. Press the second button bound to the "Calibrate Depth Lens" tool to
   calculate the per-pixel depth correction factors based on the
   collected tie points. This will write a depth correction file to the
   Kinect 3D video capture project's configuration directory, and print
   a status message to the terminal.

### Step 2: Internally calibrate the Kinect camera

Individual Kinect cameras have slightly different internal layouts and
slightly different optical properties, meaning that their internal
calibrations, i.e., the projection matrices defining how to project
depth images back out into 3D space, and how to project color images
onto those reprojected depth images, differ individually as well. While
all Kinects are factory-calibrated and contain the necessary calibration
data in their firmware, the format of those data is proprietary and
cannot be read by the Kinect 3D video capture project software, meaning
that each Kinect camera has to be calibrated internally before it can be
used. In practice, the differences are small, and a Kinect camera can be
used without internal calibration by assigning default calibration
values, but it is strongly recommended to perform calibration on each
device individually.

The internal calibration procedure requires a semi-transparent
calibration target; precisely, a checkerboard with alternating clear and
opaque tiles. Such a target can be constructed by gluing a large sheet
of paper to a clear glass plate, drawing or ideally printing a
checkerboard onto it, and cutting out all "odd" tiles using large rulers
and a sharp knife. It is important that the tiles are lined up precisely
and have precise sizes, and that the clear tiles are completely clean
without any dust, specks, or fingerprints. Calibration targets can have
a range of sizes and numbers of tiles, but we found the ideal target to
contain 7x5 tiles of 3.5"x3.5" each.

Given an appropriate calibration target, the calibration process is
performed using RawKinectViewer and its "Draw Grids" tool. The procedure
is to show the calibration target to the Kinect camera from a variety of
angles and distances, and to capture a calibration tie point for each
viewpoint by fitting a grid to the target's images in the depth and
color streams interactively.

The detailed procedure is:

1. Aim Kinect camera at calibration target from a certain position and
   angle. It is important to include several views where the calibration
   target is seen at an angle.

2. Capture an average depth frame by selecting the "Average Frames" main
   menu item, and wait until a static depth frame is displayed.

3. Drag the virtual grids displayed in the depth and color frames using
   the "Draw Grid" tool's first button until the virtual grids exactly
   match the calibration target. Matching the target in the depth frame
   is relatively tricky due to the inherent fuzziness of the Kinect's
   depth camera. Doing this properly will probably take some practice.
   The important idea is to get a "best fit" between the calibration
   target and the grid. For the particular purpose of the Augmented
   Reality Sandbox, the color frame grids can be completely ignored
   because only the depth camera is used; however, since calibration
   files are shared between all uses of the Kinect 3D video capture
   project, it is best to perform a full, depth and color, calibration.

4. Press the "Draw Grid" tool's second button to store the just-created
   calibration tie point.

5. Deselect the "Average Frames" main menu entry, and repeat from step 1
   until a sufficient number of calibration tie points have been
   captured. The set of all tie points already selected can be displayed
   by pressing the "Draw Grid" tool's third button.

After all tie points have been collected:

6. Press the "Draw Grid" tool's fourth button to calculate the Kinect
   camera's internal calibration parameters. These will be written to an
   intrinsic parameter file in the Kinect 3D video capture project's
   configuration directory.

This calibration step is illustrated in the following [tutorial video](http://www.youtube.com/watch?v=Qo05LVxdlfo)

### Step 3: Mount the Kinect camera above the sandbox

In theory, the Kinect camera can be aimed at the sand surface from any
position and/or angle, but for best results, we recommend to position
the camera such that it looks straight down onto the surface, and such
that the depth camera's field-of-view exactly matches the extents of the
sandbox. RawKinectViewer can be used to get real-time visual feedback
while aligning the Kinect camera.

### Step 4: Measure the base plane equation of the sand surface

Because the Kinect camera can be aimed at the sand surface arbitrarily,
the Augmented Reality Sandbox needs to know the equation of the base
plane corresponding to the average flattened sand surface, and the "up
direction" defining elevation above or below that base plane.

The base plane can be measured using RawKinectViewer and the "Extract
Planes" tool. Flatten and average the sand surface such that it is
exactly horizontal, or place a flat board above the sand surface. Then
capture an average depth frame by selecting the "Average Frames" main
menu entry, and wait until the depth image stabilizes. Now use the
"Extract Planes" tool to draw a rectangle in the depth frame that *only*
contains the flattened sand surface. After releasing the "Extract
Planes" tool's button, the tool will calculate the equation of the plane
best fitting the selected depth pixels, and print two versions of that
plane equation to the terminal: the equation in depth image space, and
the equation in camera space. Of these, only the second is important.
The tool prints the camera-space plane equation in the form

> x * (normal_x, normal_y, normal_z) = offset

This equation has to be entered into the sandbox layout file, which is
by default called BoxLayout.txt and contained in the Augmented Reality
Sandbox's configuration directory. The format of this file is simple:
the first line contains the sandbox's base plane equation in the form

> (normal_x, normal_y, normal_z), offset

The plane equation printed by the "Extract Planes" tool only needs to be
modified slightly when pasting it into the sandbox layout file: the "x
*" part has to be removed, and the equal sign has to be replaced by a
comma. The other four lines in the sandbox layout file are filled in in
the next calibration step.

The base plane equation defines the zero elevation level of the sand
surface. Since standard color maps equate zero elevation with sea level,
and due to practical reasons, the base plane is often measured above the
flattened average sand surface, it might be desirable to lower the zero
elevation level. This can be done easily be editing the sandbox layout
file. The zero elevation level can be shifted upwards by increasing the
offset value (the fourth component) of the plane equation, and can be
shifted downwards by decreasing the offset value. The offset value is
measured in cm; therefore, adding 10 to the original offset value will
move sea level 10 cm upwards.

This calibration step is illustrated in the following [tutorial video](http://www.youtube.com/watch?v=9Lt4J_BErs0).

### Step 5: Measure the extents of the sand surface

The Augmented Reality Sandbox needs to know the lateral extents of the
visible sand surface with respect to the base plane. These are defined
by measuring the 3D positions of the four corners of the flattened
average sand surface using RawKinectViewer and a 3D measurement tool,
and then entering those positions into the sandbox layout file.

Start RawKinectViewer, and create a 3D measurement tool by assigning a
"Measure 3D Positions" tool to some button. Then measure the 3D
positions of the four corners of the flattened sand surface in the order
lower left, lower right, upper left, upper right; in other words, form a
mirrored Z starting in the lower left.

To measure a 3D position, press and release the button to which the
measurement tool was bound inside the depth frame. This will query the
current depth value at the selected position, project it into 3D camera
space, and print the resulting 3D position to the console. Simply paste
the four corner positions, in the order mentioned above, into the
sandbox layout file.

### Step 6: Mount the projector above the sandbox

Just like with the Kinect camera, the Augmented Reality Sandbox is
capable of dealing with arbitrary projector alignments. As long as there
is some overlap between the Kinect camera's field-of-view and the
projector's projection area, the two can be calibrated with respect to
each other. However, for several reasons, it is best to align the
projector carefully such that it projects perpendicularly to the
flattened average sand surface. The main reason is pixel distortion: if
the projection is wildly off-axis, the size of projected pixels will
change sometimes drastically along the sand surface. While the Augmented
Reality Sandbox can account for overall geometric distortion, it cannot
change the size of displayed pixels, and the projected image looks best
if all pixels are approximately square and the same size.

Some projectors, especially short-throw projectors, have off-axis
projections, meaning that the image is not centered on a line coming
straight out of the projection lens. In such cases, perpendicular
projection does not imply that the projector is laterally centered above
the sandbox; in fact, it will have to be mounted off to one side. The
criterion to judge perpendicular projection is that the projected image
appears as a rectangle, not a trapezoid.

We strongly recommend against using any built-in keystone correction a
particular projector model might provide. The Augmented Reality Sandbox
corrects for keystoning internally, and projector-based keystone
correction works on an already pixelated image, meaning that it severely
degrages image quality. Never use keystone correction. Align the
projector as perpendicularly as possible, and let the Augmented Reality
Sandbox handle the rest.

The second reason to aim for perpendicular projections is focus.
Projector images are focused in a plane perpendicular to the projection
direction, meaning that only a single line of the projected image will
be in correct focus when a non-perpendicular projection is chosen.
Either way, after the projector has been mounted, we recommend to focus
it such that the entirety of the flattened average sand surface is as
much in focus as possible.

On a tangential note, we also strongly recommend to only run projectors
at their native pixel resolutions. Most projector models will support a
wide range of input video formats to accomodate multiple uses, but using
any resolution besides the one corresponding to the projector's image
generator is a very bad idea because the projector will have to rescale
the input pixel grid to its native pixel grid, which causes severe
degradation in image quality. Some projectors "lie" about their
capabilities to seem more advanced, resulting in a suboptimal resolution
when using plug&play or automatic setups. It is always a good idea to
check the projector's specification for its native resolution, and
ensure that the graphics card uses that resolution when the projector is
connected.

### Step 7: Calculate the projector calibration matrix

The most important step to create a true augmented reality display is to
calibrate the Kinect camera capturing the sand surface and the projector
projecting onto it with respect to each other, so that the projected
colors and topographic contour lines appear exactly in the right place.
Without this calibration, the Augmented Reality Sandbox is just a
sandbox with some projection.

This calibration step is performed using the CalibrateProjector utility,
and a custom calibration target. This target has to be a flat circular
disk whose exact center point is marked in some fashion. We recommend to
use an old CD, glue a white paper disk of the proper size to one side,
and draw two orthogonal lines through the CD's center point onto the
paper disk. It is important that the two lines intersect in the exact
center of the disk.

The calibration procedure is to place the disk target into the Kinect
camera's field-of-view in a sequence of prescribed positions, guided by
the projector. When CalibrateProjector is started, it will first capture
a background image of the current sand surface; it is important that the
surface is not disturbed during or after this capture step, and that no
other objects are between the Kinect camera and the sand surface.
Afterwards, CalibrateProjector will collect a sequence of 3D tie points.
For each tie point, it will display two intersecting lines. The user has
to position the disk target such that the projected lines exactly
intersect in the disk's center point, and such that the disk surface is
parallel to the flattened average sand surface, i.e., the base plane
that was collected in a previous calibration step. It is important to
place the disk at a variety of elevations above and ideally below the
base surface to collect a full 3D calibration matrix. If all tie points
are in the same plane, the calibration procedure will fail.

1. Start CalibrateProjector and wait for it to collect a background
   frame. Background capture is active while the screen is red. It is
   essential to run CalibrateProjector in full-screen mode on the
   projector, or the resulting calibration will be defective. See the
   Vrui user's manual on how to force Vrui applications to run at the
   proper position and size. Alternatively, switch CalibrateProjector
   into full-screen mode manually by pressing the Win+f key combination.
   
   When started, CalibrateProjector must be told the exact pixel size of
   the projector's image using the -s <width> <height> command line
   option. Using a wrong pixel size will result in a defective
   calibration. The recommended BenQ short-throw projector has 1024x768
   pixels, which is also the default in the software. In other words,
   when using an XGA-resolution projector, the -s option is not
   required.

2. Create a "Capture" tool and bind it to two keys (here "1" and "2").
   Press and hold "1" and move the mouse to highlight the "Capture" item
   in the tool selection menu that pops up. Then release "1" to select
   the highlighted item. This will open a dialog box prompting to press
   a second key; press and release "2". This will close the dialog box.
   Do not press "1" again when the dialog box is still open; that will
   cancel the tool creation process.
   This process binds functions to two keys: "1" will capture a tie
   point, and "2" will re-capture the background sand surface. "2"
   should only be pressed if the sand surface changes during the
   calibration procedure, for example if a hole is dug to capture a
   lower tie point. After any change to the sand surface, remove the
   calibration object and any other objects, press "2", and wait for the
   screen to turn black again.

3. Place the disk target at some random elevation above or below the
   flattened average sand surface such that the intersection of the
   projected white lines exactly coincides with the target's center
   point.

4. Remove your hands from the disk target and confirm that the target
   is seen by the Kinect camera. CalibrateProjector will display all
   non-background objects as yellow blobs, and the object it identified
   as the calibration target as a green blob. Because there is no
   calibration yet, the green blob corresponding to the disk target will
   not be aligned with the target; simply ensure that there is a green
   blob, that it is circular and stable, and that it matches the actual
   calibration target (put your hand next to it, and see if the yellow
   blob matching your hand appears next to the green blob).

5. Press the "Capture" tool's first button ("1"), and wait until the tie
   point is captured. Do not move the calibration target or hold any
   objects above the sand surface while a tie point is captured.

6. CalibrateProjector will move on to the next tie point position, and
   display a new set of white lines. Repeat from step 3 until all tie
   points have been captured. Once the full set has been collected,
   CalibrateProjector will calculate the resulting calibration matrix,
   print some status information, and write the matrix to a file inside
   the Augmented Reality Sandbox's configuration directory. The user can
   continue to capture more tie points to improve calibration as
   desired; the calibration file will be updated after every additional
   tie point. Simply close the application window when satisfied.
	 Additionally, after the first round of tie points has been collected,
   CalibrateProjector will track the calibration target in real-time and
   indicate its position with red crosshairs. To check calibration
   quality, place the target anywhere in or above the sandbox, remove
   your hands, and ensure that the red crosshairs intersect in the
   target's center.

This calibration step is illustrated in the following [tutorial video](http://www.youtube.com/watch?v=V_-Qn7oEsn4) or the [older video](http://www.youtube.com/watch?v=vXkA9gUoSAc)
shows the previous calibration procedure, and no longer applies.

### Step 8: Run the Augmented Reality Sandbox

At this point, calibration is complete. It is now possible to run the
main Augmented Reality Sandbox application from inside the source code
directory (or the -- optionally chosen -- installation directory):

> ./bin/SARndbox -fpv

The `-fpv` option tells the AR Sandbox to use the projector calibration
matrix created in step 7.

It is very important to run the application in full-screen mode on the
projector, or at least with the exact same window position and size as
CalibrateProjector in step 7. If this is not done correctly, the
calibration will not work as desired. To manually switch SARndbox into
full-screen mode after start-up, press the Win+f key combination.

To check the calibration, observe how the projected colors and
topographic contour lines exactly match the physical features of the
sand surface. If there are discrepancies between the two, repeat
calibration step 7 until satisfied. On a typical 40"x30" sandbox, where
the Kinect is mounted approximately 38" above the sandbox's center
point, and using a perpendicularly projecting 1024x768 projector,
alignment between the real sand surface and the projected features
should be on the order of 1 mm.

SARndbox provides a plethora of configuration files and command line
options to fine-tune the operation of the Augmented Reality Sandbox as
desired. Run SARndbox -h to see the full list of options and their
default values, or refer to external documentation on the project's web
site.

#### Note on water simulation

Without the real-time water simulation, the Augmented Reality Sandbox
has very reasonable hardware requirements. Any current PC with any
current graphics card should be able to run it smoothly. The water
simulation, on the other hand, places extreme load even on high-end
current hardware. We therefore recommend to turn off the water
simulation, using the -ws 0.0 0 command line option to SARndbox, or
reducing its resolution using the -wts <width> <height> command line
option with small sizes, e.g., -wts 200 150, for initial testing or
unless the PC running the Augmented Reality Sandbox has a top-of-the
line CPU, a high-end gaming graphics card, e.g., an Nvidia GeForce 970,
and the vendor-supplied proprietary drivers for that graphics card.
