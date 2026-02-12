# add_definitions(-DAPI_EXPORTS)
set(TENSORRT_ROOT /usr/local/TensorRT-10.8.0.43)
include_directories(${TENSORRT_ROOT}/include)
link_directories(${TENSORRT_ROOT}/lib)

set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)
enable_language(CUDA)

add_library(yolov8_trt SHARED impl.cpp
                              block.cpp
                              calibrator.cpp
                              model.cpp
                              postprocess.cpp
                              postprocess.cu
                              preprocess.cu
                              yololayer.cu)
target_link_libraries(yolov8_trt PRIVATE nvinfer 
                              nvinfer_plugin nvonnxparser cudart 
                              PUBLIC ${OpenCV_LIBS})

#target_link_libraries(yolov8_trt PRIVATE nvinfer
#                                         cudart
#                                 PUBLIC ${OpenCV_LIBS})
target_compile_options(yolov8_trt PRIVATE -w)

add_executable(yolov8_det yolov8_det.cpp)
target_link_libraries(yolov8_det yolov8_trt nvinfer cudart)
target_compile_options(yolov8_det PRIVATE -w)

install(TARGETS yolov8_det
        DESTINATION lib/${PROJECT_NAME})
