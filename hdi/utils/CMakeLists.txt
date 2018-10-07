set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

include_directories ("${CMAKE_SOURCE_DIR}")
include_directories ("${CMAKE_CURRENT_BINARY_DIR}")

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
  Widgets
)

# Gather list of all .h files in "/"
file(GLOB HeaderFiles *.h)
file(GLOB QOBJHeaderFiles *_qobj.h)
file(GLOB SourceFiles *.cpp)
file(GLOB FormFiles *.ui)
file(GLOB ResourceFiles *.qrc)

qt5_wrap_ui(UiHeaders ${FormFiles})
qt5_wrap_cpp(MocSources ${QOBJHeaderFiles})
qt5_add_resources(Resources ${ResourceFiles})

add_library(hdiutils ${HeaderFiles} ${SourceFiles} ${UiHeaders} ${MocSources} ${Resources} )

target_link_libraries (hdiutils Qt5::Core Qt5::Widgets)


########### INSTALL ##############
install (TARGETS hdiutils DESTINATION bin/hdi)
install (FILES ${HeaderFiles} ${UiHeaders} DESTINATION include/hdi/utils)
