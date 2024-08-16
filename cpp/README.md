This folder contains the code files for the server-side program of ZenboNurseHelper for the Windows platform. It provides an Graphic User Interface (GUI) for a user to remotely control the robot's action. The GUI currently looks like the image below and allows a user to send commands to the robot-side's app, which calls Zenbo SDK to execute those commands.

![GUI_Windows](GUI_Windows.jpg "GUI Windows")

In this project, we utilize Intel OpenVINO's human_pose_estimation_demo in their Open Model Zoo 2024 demos as a tool to guide our Zenbo robot. Our server-side program receives frames transmitted from the robot-side app, estimates human pose landmark coordinates, and reports the results to the robot-side program.

# OpenVINO Setting
Please download the Intel OpenVINO 2024.3's Windows archive [(Link)](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/download.html?PACKAGE=OPENVINO_BASE&VERSION=v_2024_3_0&OP_SYSTEM=WINDOWS&DISTRIBUTION=ARCHIVE). There are many ways to install OpenVINO, and for our case, we should use the archive file because it contains a setupvars.bat file, which is required by the open_model_zoo demo code. Suppose the name of the downloaded archive file is w_openvino_toolkit_windows_2024.3.0.16041.1e3b88e4e3f_x86_64.zip. Open it using your File Explore, and unzip the file. You should get a folder w_openvino_toolkit_windows_2024.3.0.16041.1e3b88e4e3f_x86_64. Rename it to OpenVINO and copy the OpenVINO to your home folder, which is usually C:\Users\\<your_user_name>, and you will have a new folder C:\Users\\<your_user_name>\OpenVINO. Then, you can delete the downloaded zip file w_openvino_toolkit_windows_2024.3.0.16041.1e3b88e4e3f_x86_64.zip.

Download the open_model_zoo code by
```sh
cd C:\Users\<your_user_name>
git clone --recurse-submodules https://github.com/openvinotoolkit/open_model_zoo.git
```
We need a pretrained model human-pose-estimation-0001.xml and its bin file used in the human_pose_estimation_demo, which is a part of the OpenPose algorithm.
To download the model, we use a Python tool package omz_tools, whose installation instruction is a part of the open_model_zoo. See [(Link)](https://github.com/openvinotoolkit/open_model_zoo/blob/master/tools/model_tools/README.md).
However, Windows 11 does not contain the Python program by default. If you do not have Python installed, please follow the instruction below.

## Install Python
First, do not install the Python program provided by the Windows Store, which will install the Python program in your personal folder such as C:\Users\\<your_user_name>\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.12_qbz5n2kfra8p0. The length of the folder is very long in terms of the full path. When you install the omz_tools, the process will halt because omz_tools contains some files whose path and file names are too long to be install on a Windows system. By default, Windows only allows 256 characters for a file's full length. Although this setting is adjustable, we do not want to modify it.   

```sh
sudo apt install python3-pip
```
This command also installs the setuptools package, which is equivalent to the open_model_zoo's installation instruction "pip install setuptools".
Thereafter, we install the the openvino-dev package. Because Ubuntu 24.04 prevent system-wide Python package installation, we need to modify Intel's instruction by replacing "pip install openvino-dev" to
```sh
pip install openvino-dev --break-system-packages
```
Navigate to the open_model_zoo/tools/model_tools directory, and install the omz_tools package
```sh
cd ~/open_model_zoo/tools/model_tools
pip install . --break-system-packages
```
After installing the model_tools package, we use this command to download the human-pose-estimation models from a file server.
```sh
python3 ~/open_model_zoo/tools/model_tools/src/omz_tools/omz_downloader.py --list ~/open_model_zoo/demos/human_pose_estimation_demo/cpp/models.lst -o ~/open_model_zoo/models
```
It will download 23 files saved in ~/open_model_zoo/models/intel and ~/open_model_zoo/models/public although we only need 2 of them. However, this command is better than the Intel's instruction "omz_downloader --all" because it will download a lot of files and take a long time.

# Install Our Files
Suppose your Open Model Zoo is installed in ~/open_model_zoo.
Please git clone this repository into the demos directory.
```sh
cd ~/open_model_zoo/demos
git clone https://github.com/yangchihyuan/ZenboNurseHelper.git
```

# Install Dependencies
## Protocol Buffer 
We use this tool to pass messages from our server program to the Android app.
```sh
sudo apt install protobuf-compiler
```
It will install Protocol Buffer version 3.21.12-8.2.

## OpenCV
It is required by the open_model_zoo's human_pose_estimation demo, and we use it to show images captured by the Zenbo robot's camera.
```sh
sudo apt install libopencv-dev
```
It will install OpenCV version 4.6.0.

## libgflags
It is a tool library to help us parse command arguments
```sh
sudo apt install libgflags-dev
```
It will install libgflags 2.2.2-2.

## Qt 
We use it to create our GUI
```sh
sudo apt install qt6-base-dev
sudo apt install qt6-multimedia-dev
```
It will install Qt version 6.4.2.

### Hint
The two commands to install Qt base and multimedia libraries allow you to compile this project. However, they do not isntall Qt Designer, a convenient tool to the GUI file mainwindow.ui. If you want to install Qt Designer, you need to use this command
```sh
sudo apt install qtcreator
```
The Qt creator takes more than 1G disk space because it requires many libraries. Once installed, you can launch the program to open the mainwindow.ui file with Qt Designer.

![QtDesigner_Open](QtDesigner_Open.jpg "QtDesigner_Open")

## PortAudio 
We use it to play voice on the server transmitted from the Android app and received from the robot's microphone.
There is no package made for the Ubuntu system, and we need to compile it from downloaded source files, which are available on its GitHub page
```sh
cd ~
git clone https://github.com/PortAudio/portaudio.git
```
There is an instruction page teaching how to compile and install PortAudio [(Link)](https://www.portaudio.com/docs/v19-doxydocs/compile_linux.html)
However, as the page claims it is not reviewed, we modified its commands to
```sh
sudo apt-get install libasound2-dev
cd ~/portaudio
./configure
make
sudo make install
```

## whisper.cpp
It is an voice-to-text library and we utilize it on our server-side program to quickly generate sentences spoken by an operator, which will be sent to the Zenbo robot to speak out.
There is no package make for the Ubuntu system, and we need to compile it from it source file downloaded from its GitHub repository
```sh
cd ~
git clone https://github.com/ggerganov/whisper.cpp.git
```
We need a Whisper model. In out program, we use the base model for Mandarin.
```sh
cd ~/whisper.cpp
bash ./models/download-ggml-model.sh base
```
It will download ggml-base.bin from the HuggingFace website.
We need its compiled .o files, which will be used in our server-side program.
```sh
make
```
Because whisper.cpp runs slowly if it only uses CPUs, we need a GPU to accelerate its computation. In our case, Ubuntu desktop 24.04 installs the NVidia-driver 535 by default. It is not the latest one, but still works.

# Compile and Run
Run the OpenVINO's build_demos.sh in ~/open_model_zoo/demos to build this project, and an executable file 9_NurseHelper should be created at ~/omz_demos_build/intel64/Release/
To make it easy, we make s build_demos.sh in the directory ~/open_model_zoo/demos/ZenboNurseHelper/cpp
```sh
cd ~/open_model_zoo/demos/ZenboNurseHelper/cpp
./build_demos.sh
```
This command will compile all open_model_zoo's demos, including our ZenboNurseHelper. After make the execute file 9_NurseHelper, execute the command to launch it.
```sh
./run_server_side_program.sh
```
The program easily crashes because we use several libraries containing bugs. To detect those bugs, use this command
```sh
./run_server_side_program.sh debug
```
which use gdb for debugging.

# Known problems and workarounds
## Qt FreeType crash problem
Our program often crashes in this function FT_Load_Glyph () at /lib/x86_64-linux-gnu/libfreetype.so.6, which is called by QFontEngineFT::loadGlyph(QFontEngineFT::QGlyphSet*, unsigned int, QFixedPoint const&, QFontEngine::GlyphFormat, bool, bool) const () at /home/chihyuan/Qt/6.7.2/gcc_64/lib/libQt6Gui.so.6. According to two blogs [(Link1)](https://stackoverflow.com/questions/40490414/cannot-trace-cause-of-crash-in-qt-program) [(Link2)](https://blog.csdn.net/weixin_41797797/article/details/105861978), it is a Qt bug only occurring on Linux. To avoid this problem, use the command in the terminal window before launching our program.

