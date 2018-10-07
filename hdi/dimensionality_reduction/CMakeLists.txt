set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

include_directories ("${CMAKE_SOURCE_DIR}")

if(APPLE)
    if(EXISTS "/usr/local/opt/llvm/lib/libiomp5.dylib")
        set(OPENMP_FOUND TRUE)

        message("OS is macOS, clang with OpenMP support detected.")

        set(CMAKE_CXX_COMPILER "/usr/local/opt/llvm/bin/clang++")
        set(CMAKE_C_COMPILER "/usr/local/opt/llvm/bin/clang")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fopenmp")
        set(CMAKE_XCODE_ATTRIBUTE_CC "/usr/local/opt/llvm/bin/clang")
        set(CMAKE_XCODE_ATTRIBUTE_CXX "/usr/local/opt/llvm/bin/clang")
        include_directories("/usr/local/opt/llvm/lib/clang/7.0.0/include")
    else()
        message("OS is macOS, no OpenMP support detected.")
        add_definitions( -D__USE_GCD__)
    endif()

    set_target_properties(${PROJECT} PROPERTIES XCODE_ATTRIBUTE_COMPILER_INDEX_STORE_ENABLE "NO")

else(APPLE)
    find_package(OpenMP REQUIRED)
    if (OPENMP_FOUND)
        message (STATUS "OpenMP found")
        set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    endif()
endif(APPLE)

find_package(Qt5 REQUIRED
  Core
  Gui
)

include_directories ("${CMAKE_SOURCE_DIR}/../Flann/flann/src/cpp")
#include_directories("$ENV{FLANN_DIR}/include")
message (STATUS "$ENV{FLANN_DIR}/include")

# Gather list of all .h files in "/"
file(GLOB HeaderFiles *.h)
file(GLOB SourceFiles *.cpp)

add_library   (hdidimensionalityreduction ${HeaderFiles} ${SourceFiles} )

target_link_libraries (hdidimensionalityreduction Qt5::Core Qt5::Gui)


########### INSTALL ##############
install (TARGETS hdidimensionalityreduction DESTINATION bin/hdi)
install (FILES ${HeaderFiles} ${UiHeaders} DESTINATION include/hdi/dimensionality_reduction)
