set(test_app acf-pipeline)

####################
### acf-pipeline ###
####################

# Define list of sources, add gpu files as needed
set(acf_srcs 
  pipeline.cpp
  GPUDetectionPipeline.h
  GPUDetectionPipeline.cpp

  VideoCaptureImage.h
  VideoCaptureImage.cpp
  
  # Simple line segment shader for the usual green box annotations:
  lines.h
  lines.cpp
)

add_executable(${test_app} ${acf_srcs})
target_link_libraries(${test_app} PUBLIC acf acf_util cxxopts::cxxopts ${OpenCV_LIBS})

# cross platform testing
hunter_add_package(gauze)
find_package(gauze CONFIG REQUIRED)

# thread-pool-cpp:
hunter_add_package(thread-pool-cpp)
find_package(thread-pool-cpp CONFIG REQUIRED)
target_link_libraries(${test_app} PUBLIC thread-pool-cpp::thread-pool-cpp)

# aglet (opengl context):
hunter_add_package(aglet)
find_package(aglet CONFIG REQUIRED)
target_link_libraries(${test_app} PUBLIC aglet::aglet)

target_include_directories(${test_app} PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/../>")
target_compile_definitions(${test_app} PUBLIC ACF_DO_GPU=1)

set_property(TARGET ${test_app} PROPERTY FOLDER "app/console")
install(TARGETS ${test_app} DESTINATION bin)

#############
### TEST ####
#############
set(test_name AcfPipelineTest)
message("test_app ${test_app} test_name ${test_name}")

gauze_add_test(
  NAME ${test_name}
  COMMAND ${test_app}
      --input=$<GAUZE_RESOURCE_FILE:${DRISHTI_FACES_FACE_IMAGE}>
      --repeat=5
      --model=$<GAUZE_RESOURCE_FILE:${DRISHTI_ASSETS_FACE_DETECTOR}>
      --minimum=128
      --calibration=0.01
      --global
)

if(WIN32 OR CYGWIN)
  set_property(
    TEST ${test_name}
    PROPERTY
    ENVIRONMENT
    "PATH=$<TARGET_FILE_DIR:acf>;$ENV{PATH}"
   )
endif()
