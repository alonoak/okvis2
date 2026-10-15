cmake_minimum_required(VERSION 3.8...3.16)

# Copy file SOURCE to DESTINATION if DESTINATION exists and show an error
# message otherwise.
function(copy_if_exists SOURCE DESTINATION)
  if(EXISTS "${DESTINATION}")
    execute_process(COMMAND ${CMAKE_COMMAND} -E copy_if_different "${SOURCE}" "${DESTINATION}")
  else()
    message(FATAL_ERROR "File ${DESTINATION} doesn't exist. Did you forget to update the git submodules? Run:\ngit submodule update --init --recursive\n")
  endif()
endfunction()

if(NOT USE_SYSTEM_BRISK)
  # Use modified CMakeLists.txt files and OpenCV header for brisk to build with
  # both OpenCV 4 and OpenCV 5, which renamed the features2d module to features.
  copy_if_exists(${CMAKE_CURRENT_SOURCE_DIR}/patches/brisk/CMakeLists.txt ${CMAKE_CURRENT_SOURCE_DIR}/brisk/CMakeLists.txt)
  copy_if_exists(${CMAKE_CURRENT_SOURCE_DIR}/patches/brisk/agast/CMakeLists.txt ${CMAKE_CURRENT_SOURCE_DIR}/brisk/agast/CMakeLists.txt)
  copy_if_exists(${CMAKE_CURRENT_SOURCE_DIR}/patches/brisk/agast/include/agast/wrap-opencv.h ${CMAKE_CURRENT_SOURCE_DIR}/brisk/agast/include/agast/wrap-opencv.h)
  option(BRISK_BUILD_DEMO OFF)
  add_subdirectory(brisk)
endif()

if(NOT USE_SYSTEM_DBOW2)
  # Use a modified CMakeLists.txt for DBoW2 to allow building as a static
  # library and to fix issues when installing it along with okvis.
  copy_if_exists(${CMAKE_CURRENT_SOURCE_DIR}/patches/DBoW2/CMakeLists.txt ${CMAKE_CURRENT_SOURCE_DIR}/DBoW2/CMakeLists.txt)
  option(BUILD_Demo OFF)
  add_subdirectory(DBoW2)
  set(DBoW2_LIBRARIES DBoW2 PARENT_SCOPE)
endif()

if(NOT USE_SYSTEM_CERES)
  # Use a modified CMakeLists.txt for ceres to build with Eigen 5 and CUDA 13.
  copy_if_exists(${CMAKE_CURRENT_SOURCE_DIR}/patches/ceres-solver/CMakeLists.txt ${CMAKE_CURRENT_SOURCE_DIR}/ceres-solver/CMakeLists.txt)
  if(OPENMP_FOUND)
    set(CERES_THREADING_MODEL OPENMP)
  else()
    set(CERES_THREADING_MODEL CXX_THREADS)
  endif()
  option(BUILD_TESTING OFF)
  option(BUILD_EXAMPLES OFF)
  option(BUILD_BENCHMARKS OFF)
  add_subdirectory(ceres-solver)
endif()

# Use a modified CMakeLists.txt for OpenGV to build with the same flags as
# okvis2, and a modified types.hpp to build with Eigen 5.
copy_if_exists(${CMAKE_CURRENT_SOURCE_DIR}/patches/opengv/CMakeLists.txt ${CMAKE_CURRENT_SOURCE_DIR}/opengv/CMakeLists.txt)
copy_if_exists(${CMAKE_CURRENT_SOURCE_DIR}/patches/opengv/include/opengv/types.hpp ${CMAKE_CURRENT_SOURCE_DIR}/opengv/include/opengv/types.hpp)
add_subdirectory(opengv)
set(OPENGV_LIBRARIES opengv PARENT_SCOPE)

if(BUILD_TESTS)
  option(INSTALL_GTEST OFF)
  add_subdirectory(googletest)
endif()
