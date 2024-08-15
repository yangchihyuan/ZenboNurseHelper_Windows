cmake_minimum_required(VERSION 3.5)

if( WIN32 )
	set(Protobuf_PREFIX_PATH
		"$ENV{HomeDrive}$ENV{HomePath}/vcpkg/installed/x64-windows/share/protobuf"
		"$ENV{HomeDrive}$ENV{HomePath}/vcpkg/installed/x64-windows/include"
		"$ENV{HomeDrive}$ENV{HomePath}/vcpkg/installed/x64-windows/lib" )
#		"$ENV{HomeDrive}$ENV{HomePath}/vcpkg/installed/x64-windows/bin" )
	set(Qt_PREFIX_PATH c:/Qt/6.7.2/msvc2019_64)
#    set(ABSL_PATH $ENV{HomeDrive}$ENV{HomePath}/abseil-cpp)
    set(ABSL_PATH $ENV{HomeDrive}$ENV{HomePath}/vcpkg/installed/x64-windows/share/absl)
	list(APPEND CMAKE_PREFIX_PATH ${Protobuf_PREFIX_PATH} ${Qt_PREFIX_PATH} ${ABSL_PATH})
	cmake_policy(SET CMP0071 NEW)     #to suppress a warning message
	set(PROTOBUF_PROTOC_EXECUTABLE $ENV{HomeDrive}$ENV{HomePath}/vcpkg/packages/protobuf_x64-windows/tools/protobuf/protoc.exe) 
    set(WHISPER_HEADER_PATH $ENV{HomeDrive}$ENV{HomePath}/whisper.cpp/include)    #This is C:\Users\yangc\whipser.cpp/spm-headers, add_demo macro won't translate the "\" to "/"
    set(WHISPER_LIB_PATH $ENV{HomeDrive}$ENV{HomePath}/whisper.cpp/out/build/x64-Debug/src/whisper.lib)
    set(GGML_HEADER_PATH $ENV{HomeDrive}$ENV{HomePath}/whisper.cpp/ggml/include)
    set(PORTAUDIO_INCLUDE_PATH $ENV{HomeDrive}$ENV{HomePath}/portaudio/include)
    set(PORTAUDIO_LIB_PATH $ENV{HomeDrive}$ENV{HomePath}/portaudio/out/build/x64-Debug/portaudio_x64.lib)
    set(PROTOBUF_ABSL_PROVIDER package)
#    add_subdirectory(${ABSL_PATH} binary_dir)
    set(CMAKE_TOOLCHAIN_FILE $ENV{HomeDrive}$ENV{HomePath}/vcpkg/scripts/buildsystems/vcpkg.cmake)
elseif (UNIX)
    set(Qt_PREFIX_PATH $ENV{HOME}/Qt/6.7.2/gcc_64)
	list(APPEND CMAKE_PREFIX_PATH Qt_PREFIX_PATH)
    set(WHISPER_HEADER_PATH $ENV{HOME}/whisper.cpp/spm-headers)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")   #this flag works for gcc, rather than Visual Studio 2019
	link_directories(${LIB_FOLDER} /usr/local/lib)  #this is Ubuntu's folder
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#The three settings used by Qt
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

if( WIN32 )
    find_package(absl CONFIG REQUIRED)
    find_package(Protobuf REQUIRED CONFIG)   #the CONFIG is very tricky, but without it, the find_package statement will not find absl library.
elseif( UNIX )
    find_package(Protobuf REQUIRED)
endif()

find_package(OpenCV REQUIRED COMPONENTS core highgui videoio imgproc imgcodecs)
find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets Network Multimedia)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets Network Multimedia)

set(PROTO_FILE ServerSend.proto)
protobuf_generate_cpp(PROTO_SRC PROTO_HDR ${PROTO_FILE})
file(GLOB_RECURSE HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/*.hpp)
file(GLOB GGML_OBJECT $ENV{HOME}/whisper.cpp/ggml/src/*.o)
file(GLOB EXAMPLES_OBJECT $ENV{HOME}/whisper.cpp/examples/*.o)
set(SOURCES 
            JPEG.cpp
            utility_directory.cpp
            utility_TimeRecorder.cpp
            utility_compute.cpp
            utility_string.cpp
            utility_csv.cpp
            main.cpp
            human_pose_estimator.cpp
            Pose.cpp
            ${PROTO_SRC}
            mainwindow.cpp
            mainwindow.h
            mainwindow.ui
            ProcessImageThread.cpp
            SendCommandThread.cpp
            ProcessAudioThread.cpp
    )

#add_demo(NAME 9_NurseHelper
#         SOURCES ${SOURCES}
#         HEADERS ${HEADERS}
#         DEPENDENCIES monitors models pipelines portaudio
#         INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
#         )

add_demo(NAME 9_NurseHelper
         SOURCES ${SOURCES}
         HEADERS ${HEADERS}
         DEPENDENCIES monitors models pipelines
         INCLUDE_DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
         )

include_directories(${PROTOBUF_INCLUDE_DIRS})
get_filename_component(PROTO_HDR_DIRECTORY ${PROTO_HDR} DIRECTORY)
include_directories(${PROTO_HDR_DIRECTORY})
include_directories(${WHISPER_HEADER_PATH})
include_directories(${GGML_HEADER_PATH})
include_directories(${PORTAUDIO_INCLUDE_PATH})

set_target_properties(${TARGET_NAME} PROPERTIES "CMAKE_CXX_FLAGS" "${CMAKE_CXX_FLAGS}")

if( WIN32 )
    target_link_libraries(${OMZ_DEMO_NAME} PRIVATE
        Qt${QT_VERSION_MAJOR}::Widgets Qt${QT_VERSION_MAJOR}::Network
        Qt${QT_VERSION_MAJOR}::Multimedia ${GGML_OBJECT} ${EXAMPLES_OBJECT}
        ${PORTAUDIO_LIB_PATH} ${WHISPER_LIB_PATH} protobuf::libprotobuf
        absl::any absl::log absl::base absl::bits)     
elseif(UNIX)
    target_link_libraries(${OMZ_DEMO_NAME} PRIVATE ${PROTOBUF_LIBRARIES}
        Qt${QT_VERSION_MAJOR}::Widgets Qt${QT_VERSION_MAJOR}::Network
        Qt${QT_VERSION_MAJOR}::Multimedia ${GGML_OBJECT} ${EXAMPLES_OBJECT}
        gomp "$ENV{HOME}/whisper.cpp/src/whisper.o" SDL2 ${PORTAUDIO_LIB_PATH})
endif()