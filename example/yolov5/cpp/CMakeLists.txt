cmake_minimum_required(VERSION 3.10)

project(rknn_yolov5_demo)

set(CMAKE_INSTALL_RPATH "$ORIGIN/lib")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")

# 从系统中查找opencv
find_package(OpenCV REQUIRED)

# rga
set(RGA_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../../3rdparty/librga)
set(LIBRGA ${RGA_PATH}/${CMAKE_SYSTEM_NAME}/aarch64/librga.a)
set(RGA_INCLUDES ${RGA_PATH}/include)
install(PROGRAMS ${RGA_PATH}/${CMAKE_SYSTEM_NAME}/aarch64/librga.so DESTINATION lib)

# only RGA on rv1106 and rk3588 support handle
if (TARGET_SOC STREQUAL "rk3588")
    add_definitions(-DLIBRGA_IM2D_HANDLE)
endif()

# RGA2 only support under 4G memory
if (ENABLE_DMA32 STREQUAL "TRUE")
    add_definitions(-DDMA_ALLOC_DMA32)
endif()

# support rknn zero copy api
if (ENABLE_ZERO_COPY STREQUAL "TRUE")
    add_definitions(-DENABLE_ZERO_COPY)
endif()

# rknn runtime
if (TARGET_SOC STREQUAL "rk3588" OR TARGET_SOC STREQUAL "rk356x")
    set(RKNN_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../../3rdparty/rknpu2)
    set(LIBRKNNRT ${RKNN_PATH}/${CMAKE_SYSTEM_NAME}/aarch64/librknnrt.so)
    set(LIBRKNNRT_INCLUDES ${RKNN_PATH}/include)
endif()
install(PROGRAMS ${LIBRKNNRT} DESTINATION lib)
set(LIBRKNNRT ${LIBRKNNRT})

# dma
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../../3rdparty/allocator/dma)

# rknn_yolov5_demo(image)
add_executable(rknn_yolov5_demo
    yolov5_demo.cc
    postprocess.cc
    image_utils.c
    rknpu2/yolov5.cc
)

target_link_libraries(rknn_yolov5_demo
    ${LIBRGA}
    ${OpenCV_LIBS}
    ${LIBRKNNRT}
    dl
)

# yolov5_videocapture_demo
add_executable(yolov5_videocapture_demo
    yolov5_videocapture_demo.cc
    postprocess.cc
    image_utils.c
    rknpu2/yolov5.cc
)

target_link_libraries(yolov5_videocapture_demo
    ${LIBRGA}
    ${OpenCV_LIBS}
    ${LIBRKNNRT}
    dl
)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${RGA_INCLUDES}
    ${LIBRKNNRT_INCLUDES}
)

install(TARGETS rknn_yolov5_demo DESTINATION .)
install(TARGETS yolov5_videocapture_demo DESTINATION .)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/../model/bus.jpg DESTINATION model)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/../model/coco_80_labels_list.txt DESTINATION model)
file(GLOB RKNN_FILES "${CMAKE_CURRENT_SOURCE_DIR}/../model/*.rknn")
install(FILES ${RKNN_FILES} DESTINATION model)