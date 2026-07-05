cmake_minimum_required(VERSION 3.16)
project(ossupdate VERSION 1.2.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

find_package(Qt6 REQUIRED COMPONENTS Widgets Svg Network DBus)

find_package(PkgConfig REQUIRED)
pkg_check_modules(QTKEYCHAIN qt6keychain)
if(NOT QTKEYCHAIN_FOUND)
    find_package(Qt6Keychain QUIET)
    if(Qt6Keychain_FOUND)
        set(QTKEYCHAIN_LIBRARIES Qt6Keychain::Qt6Keychain)
        set(QTKEYCHAIN_FOUND TRUE)
    endif()
endif()

if(NOT QTKEYCHAIN_FOUND)
    message(FATAL_ERROR "qt6keychain not found. Install qtkeychain-qt6-devel")
endif()

set(SOURCES
    src/main.cpp
    src/traymanager.cpp
    src/mainwindow.cpp
    src/updatechecker.cpp
    src/updatelistdialog.cpp
    src/lockmanager.cpp
    src/passwordmanager.cpp
    src/autostartmanager.cpp
    src/settings.cpp
    src/selfupdater.cpp
)

set(HEADERS
    src/traymanager.h
    src/mainwindow.h
    src/updatechecker.h
    src/updatelistdialog.h
    src/lockmanager.h
    src/passwordmanager.h
    src/autostartmanager.h
    src/settings.h
    src/selfupdater.h
)

set(RESOURCES
    resources/resources.qrc
)

add_executable(ossupdate ${SOURCES} ${HEADERS} ${RESOURCES})

target_link_libraries(ossupdate PRIVATE
    Qt6::Widgets
    Qt6::Svg
    Qt6::Network
    Qt6::DBus
    ${QTKEYCHAIN_LIBRARIES}
)

if(QTKEYCHAIN_INCLUDE_DIRS)
    target_include_directories(ossupdate PRIVATE ${QTKEYCHAIN_INCLUDE_DIRS})
endif()

target_include_directories(ossupdate PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)

target_compile_definitions(ossupdate PRIVATE APP_VERSION="${PROJECT_VERSION}")

install(TARGETS ossupdate RUNTIME DESTINATION bin)

install(FILES ossupdate.desktop DESTINATION share/applications)

install(FILES resources/icons/app_icon.svg DESTINATION share/icons/hicolor/scalable/apps
    RENAME ossupdate.svg)
