cmake_minimum_required(VERSION 3.10)

project(yolov5)

IF( DEFINED BATCH_SIZE )
    MESSAGE(STATUS "BATCH_SIZE = ${BATCH_SIZE}")
ELSE( DEFINED BATCH_SIZE )
    MESSAGE(STATUS "BATCH_SIZE is not defined")
    SET(BATCH_SIZE 1)
    MESSAGE(STATUS "Using BATCH_SIZE = ${BATCH_SIZE}")
ENDIF( DEFINED BATCH_SIZE )

IF( DEFINED CLASS_NUM )
    MESSAGE(STATUS "CLASS_NUM = ${CLASS_NUM}")
ELSE( DEFINED CLASS_NUM )
    MESSAGE(STATUS "CLASS_NUM is not defined")
    SET(CLASS_NUM 80)
    MESSAGE(STATUS "Using CLASS_NUM = ${CLASS_NUM}")
ENDIF( DEFINED CLASS_NUM )

IF( DEFINED INPUT_H )
    MESSAGE(STATUS "INPUT_H = ${INPUT_H}")
ELSE( DEFINED INPUT_H )
    MESSAGE(STATUS "INPUT_H is not defined")
    SET(INPUT_H 640)
    MESSAGE(STATUS "Using INPUT_H = ${INPUT_H}")
ENDIF( DEFINED INPUT_H )

IF( DEFINED INPUT_W )
    MESSAGE(STATUS "INPUT_W = ${INPUT_W}")
ELSE( DEFINED INPUT_W )
    MESSAGE(STATUS "INPUT_W is not defined")
    SET(INPUT_W 640)
    MESSAGE(STATUS "Using INPUT_W = ${INPUT_W}")
ENDIF( DEFINED INPUT_W )

add_definitions(-std=c++11)
add_definitions(-DAPI_EXPORTS)
option(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Debug)

# TODO(Call for PR): make cmake compatible with Windows
set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)
enable_language(CUDA)

# include and link dirs of cuda and tensorrt, you need adapt them if yours are different
# cuda
include_directories(/usr/local/cuda/include)
link_directories(/usr/local/cuda/lib64)
# tensorrt
# TODO(Call for PR): make TRT path configurable from command line
include_directories(/home/nvidia/TensorRT-8.2.5.1/include/)
link_directories(/home/nvidia/TensorRT-8.2.5.1/lib/)

include_directories(${PROJECT_SOURCE_DIR}/src/)
include_directories(${PROJECT_SOURCE_DIR}/plugin/)
file(GLOB_RECURSE SRCS ${PROJECT_SOURCE_DIR}/src/*.cpp ${PROJECT_SOURCE_DIR}/src/*.cu)
file(GLOB_RECURSE PLUGIN_SRCS ${PROJECT_SOURCE_DIR}/plugin/*.cu)

add_library(myplugins SHARED ${PLUGIN_SRCS})
target_link_libraries(myplugins nvinfer cudart)

find_package(OpenCV)
include_directories(${OpenCV_INCLUDE_DIRS})

add_executable(yolov5_det yolov5_det.cpp ${SRCS})
target_link_libraries(yolov5_det nvinfer)
target_link_libraries(yolov5_det cudart)
target_link_libraries(yolov5_det myplugins)
target_link_libraries(yolov5_det ${OpenCV_LIBS})

target_compile_definitions(yolov5_det PRIVATE 
    BATCH_SIZE=${BATCH_SIZE} 
    CUSTOM_CLASS_NUM=${CLASS_NUM}
    CUSTOM_INPUT_H=${INPUT_H} 
    CUSTOM_INPUT_W=${INPUT_W}
)

add_executable(yolov5_cls yolov5_cls.cpp ${SRCS})
target_link_libraries(yolov5_cls nvinfer)
target_link_libraries(yolov5_cls cudart)
target_link_libraries(yolov5_cls myplugins)
target_link_libraries(yolov5_cls ${OpenCV_LIBS})

target_compile_definitions(yolov5_cls PRIVATE 
    BATCH_SIZE=${BATCH_SIZE} 
    CUSTOM_CLASS_NUM=${CLASS_NUM}
    CUSTOM_INPUT_H=${INPUT_H} 
    CUSTOM_INPUT_W=${INPUT_W}
)

add_executable(yolov5_seg yolov5_seg.cpp ${SRCS})
target_link_libraries(yolov5_seg nvinfer)
target_link_libraries(yolov5_seg cudart)
target_link_libraries(yolov5_seg myplugins)
target_link_libraries(yolov5_seg ${OpenCV_LIBS})

target_compile_definitions(yolov5_seg PRIVATE 
    BATCH_SIZE=${BATCH_SIZE} 
    CUSTOM_CLASS_NUM=${CLASS_NUM}
    CUSTOM_INPUT_H=${INPUT_H} 
    CUSTOM_INPUT_W=${INPUT_W}
)