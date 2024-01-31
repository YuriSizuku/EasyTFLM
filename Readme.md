# EasyTFLM  

An easy cmake project for [tflite-micro](https://github.com/tensorflow/tflite-micro),  
aims to develope or debug tflm on multi platforms.  
The structure is also compatible with arduino library.  

## enviroments

- [x] win64 clang14  
      `cmake .. -DCMAKE_C_COMPILER=clang -G "Unix Makefiles"`
- [x] win64 gcc 12.2  
      `cmake .. -DCMAKE_C_COMPILER=gcc -G "Unix Makefiles"`
- [x] win64 msvc 14  
      `cmake .. -G "Visual Studio 17 2022"`
- [x] linux64 gcc 9.4  
      `cmake .. -DCMAKE_C_COMPILER=gcc -G "Unix Makefiles"`
- [x] esp32s3 gcc 8.4  
      `cmake .. -DCMAKE_C_COMPILER=xtensa-esp32s3-elf-gcc -DCMAKE_SYSTEM_NAME=Linux -G "Unix Makefiles"`  

In platformio enviroment, put into `lib/tflm` and config ini :  

``` ini
[env:esp32s3]
platform = espressif32
framework = arduino
board = um_pros3
monitor_speed = 115200
board_build.partitions = no_ota.csv
build_flags = 
   -DTF_LITE_STATIC_MEMORY
   -Ilib/tflm/src
   -Ilib/tflm/src/third_party
   -Ilib/tflm/src/third_party/flatbuffers/include
   -Ilib/tflm/src/third_party/gemmlowp
   -Ilib/tflm/src/third_party/kissfft
   -Ilib/tflm/src/third_party/ruy
```

## source

``` sh
sudo apt-get install unzip
sudo pip3 install numpy pillow
make -f tensorflow/lite/micro/tools/make/Makefile third_party_downloads
python3 tensorflow/lite/micro/tools/project_generation/create_tflm_tree.py -e hello_world  /tmp/tflm
```

Then modify some details to fix errors on compile, such as reporter `TF_LITE_REMOVE_VIRTUAL_DELETE` to public function. And make a `CmakeList.txt` file to for multi platforms.  
