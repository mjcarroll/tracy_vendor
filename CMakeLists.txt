cmake_minimum_required(VERSION 3.5)
project(tracy_vendor)

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
  add_compile_options(/W4)
endif()

find_package(ament_cmake_ros REQUIRED)
find_package(Threads REQUIRED)

macro(set_option option help value)
    option(${option} ${help} ${value})
    if(${option})
        message(STATUS "${option}: ON")
        target_compile_definitions(TracyClient PUBLIC ${option})
    else()
        message(STATUS "${option}: OFF")
    endif()
endmacro()

add_library(TracyClient ${CMAKE_SOURCE_DIR}/tracy/public/TracyClient.cpp)
target_compile_features(TracyClient PUBLIC cxx_std_11)
target_include_directories(TracyClient SYSTEM PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/tracy/public/>
    $<INSTALL_INTERFACE:include/${PROJECT_NAME}/>)
target_link_libraries(
    TracyClient
    PUBLIC
        Threads::Threads
        ${CMAKE_DL_LIBS}
)

set_option(TRACY_ENABLE "Enable profiling" ON)
set_option(TRACY_ON_DEMAND "On-demand profiling"  ON)
set_option(TRACY_CALLSTACK "Enfore callstack collection for tracy regions" ON)
set_option(TRACY_NO_CALLSTACK "Disable all callstack related functionality" OFF)
set_option(TRACY_NO_CALLSTACK_INLINES "Disables the inline functions in callstacks" OFF)
set_option(TRACY_ONLY_LOCALHOST "Only listen on the localhost interface" OFF)
set_option(TRACY_NO_BROADCAST "Disable client discovery by broadcast to local network" OFF)
set_option(TRACY_ONLY_IPV4 "Tracy will only accept connections on IPv4 addresses (disable IPv6)" OFF)
set_option(TRACY_NO_CODE_TRANSFER "Disable collection of source code" OFF)
set_option(TRACY_NO_CONTEXT_SWITCH "Disable capture of context switches" OFF)
set_option(TRACY_NO_EXIT "Client executable does not exit until all profile data is sent to server" OFF)
set_option(TRACY_NO_SAMPLING "Disable call stack sampling" ON)
set_option(TRACY_NO_VERIFY "Disable zone validation for C API" OFF)
set_option(TRACY_NO_VSYNC_CAPTURE "Disable capture of hardware Vsync events" OFF)
set_option(TRACY_NO_FRAME_IMAGE  "Disable the frame image support and its thread" OFF)
set_option(TRACY_NO_SYSTEM_TRACING  "Disable systrace sampling"  OFF)
set_option(TRACY_DELAYED_INIT "Enable delayed initialization of the library (init on first call)" OFF)
set_option(TRACY_MANUAL_LIFETIME "Enable the manual lifetime management of the profile" OFF)
set_option(TRACY_FIBERS "Enable fibers support" OFF)
set_option(TRACY_NO_CRASH_HANDLER "Disable crash handling" OFF)
set_option(TRACY_TIMER_FALLBACK "Use lower resolution timers" OFF)

if(NOT TRACY_STATIC)
    target_compile_definitions(TracyClient PRIVATE TRACY_EXPORTS)
    target_compile_definitions(TracyClient PUBLIC TRACY_IMPORTS)
endif()


set(tracy_includes
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/TracyC.h
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/Tracy.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/TracyD3D11.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/TracyD3D12.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/TracyLua.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/TracyOpenCL.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/TracyOpenGL.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/tracy/TracyVulkan.hpp)

set(client_includes
    ${CMAKE_SOURCE_DIR}/tracy/public/client/tracy_concurrentqueue.h
    ${CMAKE_SOURCE_DIR}/tracy/public/client/tracy_rpmalloc.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/tracy_SPSCQueue.h
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyArmCpuTable.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyCallstack.h
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyCallstack.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyDebug.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyDxt1.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyFastVector.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyLock.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyProfiler.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyRingBuffer.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyScoped.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyStringHelpers.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracySysTime.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracySysTrace.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/client/TracyThread.hpp)

set(common_includes
    ${CMAKE_SOURCE_DIR}/tracy/public/common/tracy_lz4.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/tracy_lz4hc.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyAlign.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyAlloc.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyApi.h
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyColor.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyForceInline.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyMutex.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyProtocol.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyQueue.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracySocket.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyStackFrames.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracySystem.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyUwp.hpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyYield.hpp)

install(
  TARGETS TracyClient EXPORT ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(FILES ${tracy_includes}
        DESTINATION include/${PROJECT_NAME}/tracy)
install(FILES ${client_includes}
        DESTINATION include/${PROJECT_NAME}/client)
install(FILES ${common_includes}
        DESTINATION include/${PROJECT_NAME}/common)

ament_export_targets(${PROJECT_NAME})
ament_package()
