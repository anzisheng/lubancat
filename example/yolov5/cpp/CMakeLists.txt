cmake_minimum_required(VERSION 3.10)

project(rknn_yolov5_demo)

# rpath
set(CMAKE_INSTALL_RPATH "$ORIGIN/lib")

# opencv
set(OpenCV_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../3rdparty/opencv/opencv-linux-aarch64/share/OpenCV)
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

# rknn runtime
if (TARGET_SOC STREQUAL "rk3588" OR TARGET_SOC STREQUAL "rk356x")
    set(RKNN_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../../3rdparty/rknpu2)
    set(LIBRKNNRT ${RKNN_PATH}/${CMAKE_SYSTEM_NAME}/aarch64/librknnrt.so)
    set(LIBRKNNRT_INCLUDES ${RKNN_PATH}/include)
endif()
install(PROGRAMS ${LIBRKNNRT} DESTINATION lib)
set(LIBRKNNRT ${LIBRKNNRT})

add_executable(${PROJECT_NAME}
    main.cc
    postprocess.cc
    image_utils.c
    rknpu2/yolov5.cc
)

target_link_libraries(${PROJECT_NAME}
    ${LIBRGA}
    ${OpenCV_LIBS}
    ${LIBRKNNRT}
)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${RGA_INCLUDES}
    ${LIBRKNNRT_INCLUDES}
)

install(TARGETS ${PROJECT_NAME} DESTINATION .)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/../model/bus.jpg DESTINATION model)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/../model/coco_80_labels_list.txt DESTINATION model)
file(GLOB RKNN_FILES "${CMAKE_CURRENT_SOURCE_DIR}/../model/*.rknn")
install(FILES ${RKNN_FILES} DESTINATION model)