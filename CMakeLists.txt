cmake_minimum_required(VERSION 3.5)
project(tracy_vendor LANGUAGES C CXX ASM)

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

find_package(Freetype REQUIRED)
find_package(glfw3 REQUIRED)
find_package(Threads REQUIRED)
find_package(TBB REQUIRED)

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

find_package(PkgConfig)
pkg_check_modules(PC_DBUS QUIET dbus-1)
find_library(DBUS_LIBRARIES
    NAMES dbus-1
    HINTS ${PC_DBUS_LIBDIR}
          ${PC_DBUS_LIBRARY_DIRS}
)
find_path(DBUS_INCLUDE_DIR
    NAMES dbus/dbus.h
    HINTS ${PC_DBUS_INCLUDEDIR}
          ${PC_DBUS_INCLUDE_DIRS}
)
get_filename_component(_DBUS_LIBRARY_DIR ${DBUS_LIBRARIES} PATH)
find_path(DBUS_ARCH_INCLUDE_DIR
    NAMES dbus/dbus-arch-deps.h
    HINTS ${PC_DBUS_INCLUDEDIR}
          ${PC_DBUS_INCLUDE_DIRS}
          ${_DBUS_LIBRARY_DIR}
          ${DBUS_INCLUDE_DIR}
    PATH_SUFFIXES include
)
set(DBUS_INCLUDE_DIRS ${DBUS_INCLUDE_DIR} ${DBUS_ARCH_INCLUDE_DIR})

pkg_check_modules(PC_CAPSTONE QUIET capstone)
find_library(CAPSTONE_LIBRARIES
    NAMES capstone
    HINTS ${PC_CAPSTONE_LIBDIR}
          ${PC_CAPSTONE_LIBRARY_DIRS}
)
find_path(CAPSTONE_INCLUDE_DIRS
    NAMES capstone.h
    HINTS ${PC_CAPSTONE_INCLUDEDIR}
          ${PC_CAPSTONE_INCLUDE_DIRS}
)

add_executable(TracyProfiler
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/debug.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/entropy_common.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/error_private.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/fse_decompress.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/pool.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/threading.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/xxhash.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/common/zstd_common.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/fse_compress.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/hist.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/huf_compress.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_compress.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_compress_literals.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_compress_sequences.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_compress_superblock.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_double_fast.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_fast.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_lazy.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_ldm.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstdmt_compress.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/compress/zstd_opt.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/decompress/huf_decompress.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/decompress/huf_decompress_amd64.S
    ${CMAKE_SOURCE_DIR}/tracy/zstd/decompress/zstd_ddict.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/decompress/zstd_decompress_block.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/decompress/zstd_decompress.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/dictBuilder/cover.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/dictBuilder/divsufsort.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/dictBuilder/fastcover.c
    ${CMAKE_SOURCE_DIR}/tracy/zstd/dictBuilder/zdict.c
    ${CMAKE_SOURCE_DIR}/tracy/nfd/nfd_portal.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/winmainArchDiscovery.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/winmain.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/WindowPosition.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/RunQueue.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/ResolvService.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/main.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/IsElevated.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/imgui/imgui_impl_opengl3.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/imgui/imgui_impl_glfw.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/ImGuiContext.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/HttpRequest.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/Fonts.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/Filters.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/ConnectionHistory.cpp
    ${CMAKE_SOURCE_DIR}/tracy/profiler/src/BackendGlfw.cpp
    ${CMAKE_SOURCE_DIR}/tracy/imgui/imgui.cpp
    ${CMAKE_SOURCE_DIR}/tracy/imgui/imgui_draw.cpp
    ${CMAKE_SOURCE_DIR}/tracy/imgui/imgui_tables.cpp
    ${CMAKE_SOURCE_DIR}/tracy/imgui/imgui_widgets.cpp
    ${CMAKE_SOURCE_DIR}/tracy/imgui/misc/freetype/imgui_freetype.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyWorker.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyWeb.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_ZoneTimeline.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_ZoneInfo.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Utility.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_TraceInfo.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Timeline.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Statistics.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Samples.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Ranges.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Plots.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Playback.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Options.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_NotificationArea.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Navigation.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Messages.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Memory.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Locks.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_GpuTimeline.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_FrameTree.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_FrameTimeline.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_FrameOverview.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_FindZone.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_CpuData.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_ContextSwitch.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_ConnectionState.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Compare.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Callstack.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView_Annotations.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyView.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyUtility.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyUserData.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTimelineItemThread.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTimelineItemPlot.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTimelineItemGpu.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTimelineItemCpuData.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTimelineItem.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTimelineController.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyThreadCompress.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTextureCompression.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTexture.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyTaskDispatch.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyStorage.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracySourceView.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracySourceTokenizer.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracySourceContents.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyProtoHistory.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyPrint.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyMouse.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyMmap.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyMicroArchitecture.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyMemory.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyImGui.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyFilesystem.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyFileselector.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyColor.cpp
    ${CMAKE_SOURCE_DIR}/tracy/server/TracyBadVersion.cpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/tracy_lz4hc.cpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/tracy_lz4.cpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracySystem.cpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracyStackFrames.cpp
    ${CMAKE_SOURCE_DIR}/tracy/public/common/TracySocket.cpp
)

target_include_directories(TracyProfiler PRIVATE
  ${CMAKE_SOURCE_DIR}/tracy/imgui
  ${CAPSTONE_INCLUDE_DIRS}
  ${DBUS_INCLUDE_DIRS}
)

target_link_libraries(TracyProfiler PRIVATE
  Freetype::Freetype glfw Threads::Threads
  ${DBUS_LIBRARIES}
  ${CAPSTONE_LIBRARIES}
  TBB::tbb
  )

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
