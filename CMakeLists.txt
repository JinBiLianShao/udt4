cmake_minimum_required(VERSION 3.15)

project(udt
        VERSION 4.11
        LANGUAGES C CXX
)

# =========================
# 构建选项
# =========================
option(UDT_BUILD_SHARED "Build shared library (.so/.dll)" ON)
option(UDT_BUILD_STATIC "Build static library (.a/.lib)" ON)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# =========================
# 源文件
# =========================
set(UDT_SOURCES
        src/api.cpp
        src/buffer.cpp
        src/channel.cpp
        src/common.cpp
        src/core.cpp
        src/epoll.cpp
        src/list.cpp
        src/md5.cpp
        src/packet.cpp
        src/queue.cpp
        src/window.cpp
)

set(UDT_HEADERS
        src/udt.h
)

# =========================
# 公共属性
# =========================
set(UDT_PUBLIC_INCLUDES
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
        $<INSTALL_INTERFACE:include>
)

# =========================
# 平台依赖
# =========================
if (WIN32)
    set(UDT_PLATFORM_LIBS ws2_32)
else()
    set(UDT_PLATFORM_LIBS pthread)
endif()

# =========================
# 共享库
# =========================
if (UDT_BUILD_SHARED)
    add_library(udt_shared SHARED
            ${UDT_SOURCES}
            ${UDT_HEADERS}
    )

    target_compile_definitions(udt_shared
            PRIVATE UDT_BUILD_SHARED
            PUBLIC  UDT_USE_SHARED
    )

    target_include_directories(udt_shared PUBLIC
            ${UDT_PUBLIC_INCLUDES}
    )

    target_link_libraries(udt_shared
            ${UDT_PLATFORM_LIBS}
    )

    set_target_properties(udt_shared PROPERTIES
            OUTPUT_NAME udt
            WINDOWS_EXPORT_ALL_SYMBOLS OFF
    )
endif()

# =========================
# 静态库
# =========================
if (UDT_BUILD_STATIC)
    add_library(udt_static STATIC
            ${UDT_SOURCES}
            ${UDT_HEADERS}
    )

    target_include_directories(udt_static PUBLIC
            ${UDT_PUBLIC_INCLUDES}
    )

    target_link_libraries(udt_static
            ${UDT_PLATFORM_LIBS}
    )

    set_target_properties(udt_static PROPERTIES
            OUTPUT_NAME udt
    )
endif()

# =========================
# 安装（可选）
# =========================
include(GNUInstallDirs)

if (UDT_BUILD_SHARED)
    install(TARGETS udt_shared
            EXPORT udtTargets
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
endif()

if (UDT_BUILD_STATIC)
    install(TARGETS udt_static
            EXPORT udtTargets
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
endif()

install(FILES src/udt.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/udt
)
