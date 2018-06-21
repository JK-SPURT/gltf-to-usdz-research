# glTF to USDZ

Proof of concept of converting glTF to USDZ for AR Quick Look Gallery.

## Reasoning

Even though I think the intensions of Apple / Pixar are great with the open source [USD](https://github.com/PixarAnimationStudios/USD) pipeline I think we as an industry should be relying more on truly open formats that are not controlled by a single entity. Installing `USD` is cumbersome, requires a lot of disk space and is completely overkill for most situations (if your goal is to convert some 3D models to USDZ and show them using `AR Quick Look Gallery`)

In order to move away from using the [USD](https://github.com/PixarAnimationStudios/USD) pipeline solution offered by Pixar I think it would be wise to try and manipulate the intermediary readeable `USDA` format. Unfortunately there are very little examples available of `USDA` files.

My idea is to dynamically generate / manipulate the intermediary a general `USDA` file-structure and pass that to the `usdz-converter` to handle the further conversion to `USDZ`.

Please note that this is just an experimental setup and should be seen as an attempt to create a simple pipeline from glTF to USDZ.

Most of the findings come from `trayser` who posted details regarding OBJ to `USDZ` conversion on [developers.apple.com](https://forums.developer.apple.com/thread/104042). `trayser` also noticed that many OBJ files failed to in conversion if they include complex tags. He recommends commenting out any line that starts with anything other than `#`, `v`, `vn`, `vt`, `vp`, `f`, `g` etc.

## Live demo

[Live demo](https://timvanscherpenzeel.github.io/gltf-to-usdz/)

![Screenshot](/assets/textures_correct.jpg?raw=true)

## To do

- ~~Test if the constructed USDZ output can actually be loaded into AR Quick Look Gallery (requires iOS 12 and a recent iOS device)~~

  [Confirmed by @domenicopanacea](https://twitter.com/domenicopanacea/status/1008266095386644480)

- ~~Test with the processing of various OBJ files (find out what is possible and what is not).~~

  OBJ file exported from https://threejs.org/editor/ with glTF as source generated working OBJ's. I've used this as a source of truth to see if the output generated from the tool was the same as the output generated from the editor.

- ~~Find a way to convert glTF geometry to OBJ geometry and extract the used textures.~~

  Done!

- ~~Dynamically construct the USDA file based on passed in textures from the glTF file.~~

  Done!

- ~~Test if `assets/DamagedHelmet.usdz` works correctly in the AR Quick Look application~~

  Done! See image above.

- Convert the example `USDZ` examples to `USDA` structures by converting `USDC` to `USDA`. Unfortunately I think this requires the installation of the [USD pipeline](https://github.com/PixarAnimationStudios/USD) and the use of [usdcat](https://github.com/PixarAnimationStudios/USD/blob/e6ce9e884a65e7d6acd762e9dbc961dcf9aa36bb/pxr/usd/bin/usdcat/usdcat.py).

  I have not yet been able to install [USD](https://github.com/PixarAnimationStudios/USD) correctly, please see the development section below.

## Limitations

- Only the first mesh in the `glTF` file will be parsed and exported. Make sure you merge all meshes.

- Only `glTF` files with external textures are currently handled (as opposed to embedded base64 encoded textures).

## Installation

- Make sure you have [Node.js](http://nodejs.org/) installed

- Upgrade your operating system to macOS High Sierra 10.13.4 or newer

- Download Xcode 10 beta and put it in `/Applications/`

```
https://developer.apple.com/download/
```

- Link to the beta version instead of the normal version

```
sudo xcode-select --switch /Applications/Xcode-beta.app
```

- Construct a new `.usda` using `gltf-to-usdz` and run it through the `usdz_converter`

```
node ./bin/gltf-to-usdz.js -i ./assets/DamagedHelmet/DamagedHelmet.gltf -o ./assets/DamagedHelmet.usda && xcrun usdz_converter ./assets/DamagedHelmet.usda ./assets/DamagedHelmet.usdz
```

- On succes the following should be outputted to the console

```
2018-06-20 17:21:23.364 usdz_converter[82749:13335731]


Converting asset file 'DamagedHelmet.usda' ...
```

In order to see the contents of the outputted `USDZ` change the extension to `.zip` and unzip it. You will see a `.usdc` and several textures (if the original glTF file had textures).

## Development

In order to install [USD](https://github.com/PixarAnimationStudios/USD) on MacOS please follow the following instructions:

- Upgrade your operating system to macOS High Sierra 10.13.4 or newer

- Download Xcode 10 beta and put it in `/Applications/`

```
https://developer.apple.com/download/
```

- Link to the beta version instead of the normal version

```
sudo xcode-select --switch /Applications/Xcode-beta.app
```

- Install Cmake and QT

```
brew install cmake
brew install cartr/qt4/qt
```

- Install PyOpenGL

```
pip install PyOpenGL
```

- Install PySide2

```
pip install --index-url=http://download.qt.io/snapshots/ci/pyside/5.9/latest/ pyside2 --trusted-host download.qt.io
```

- Update OpenImageIO release version from `Release-1.7.14.zip` to `Release-1.8.12.zip` in `build_scripts/build_usd.py`.

- Run `python USD/build_scripts/build_usd.py BUILD`, it will take roughly 1 hour, resulting in the following output if succesfully installed:

```
➜  pixar python USD/build_scripts/build_usd.py BUILD

Building with settings:
  USD source directory          /Users/timvanscherpenzeel/Projects/pixar/USD
  USD install directory         /Users/timvanscherpenzeel/Projects/pixar/BUILD
  3rd-party source directory    /Users/timvanscherpenzeel/Projects/pixar/BUILD/src
  3rd-party install directory   /Users/timvanscherpenzeel/Projects/pixar/BUILD
  Build directory               /Users/timvanscherpenzeel/Projects/pixar/BUILD/build
  CMake generator               Default
  Downloader                    curl

  Building                      Shared libraries
    Imaging                     On
      Ptex support:             Off
    UsdImaging                  On
    Python support              On
    Documentation               Off
    Tests                       Off
    Alembic Plugin              Off
      HDF5 support:             Off
    Maya Plugin                 Off
    Katana Plugin               Off
    Houdini Plugin              Off

    Dependencies                zlib, boost, TBB, JPEG, TIFF, PNG, OpenEXR, GLEW, OpenImageIO, OpenSubdiv

STATUS: Installing zlib...
STATUS: Installing boost...
STATUS: Installing TBB...
STATUS: Installing JPEG...
STATUS: Installing TIFF...
STATUS: Installing PNG...
STATUS: Installing OpenEXR...
STATUS: Installing GLEW...
STATUS: Installing OpenImageIO...
STATUS: Installing OpenSubdiv...
STATUS: Installing USD...

Success! To use USD, please ensure that you have:

    The following in your PYTHONPATH environment variable:
    /Users/timvanscherpenzeel/Projects/pixar/BUILD/lib/python

    The following in your PATH environment variable:
    /Users/timvanscherpenzeel/Projects/pixar/BUILD/bin
```

Unfortunately I have the following error when running `usdcat`:

```
➜  cupandsaucer usdcat CupAndSaucer.usdc -o CupAndSaucer.usda

------------------------ 'Python' is dying ------------------------
Python crashed. FATAL ERROR: Failed axiom: ' Py_IsInitialized() '
in operator() at line 148 of /Users/timvanscherpenzeel/Projects/pixar/USD/pxr/base/lib/tf/pyTracing.cpp

The stack can be found in Tims-MacBook-Pro.local:/var/folders/s9/rnmbn61120s2hwww26_gk3k40000gn/T//st_Python.53393
done.
------------------------------------------------------------------
[1]    53393 abort      usdcat CupAndSaucer.usdc -o CupAndSaucer.usda
```

I've reported the issue here [#19](https://github.com/PixarAnimationStudios/USD/issues/19).

## Resources

- https://graphics.pixar.com/usd/docs/Converting-Between-Layer-Formats.html

- https://forums.developer.apple.com/thread/104042

- https://graphics.pixar.com/usd/docs/UsdPreviewSurface-Proposal.html
