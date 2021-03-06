
# KDE Application Version, managed by release script
set(KDE_APPLICATIONS_VERSION_MAJOR "18")
set(KDE_APPLICATIONS_VERSION_MINOR "07")
set(KDE_APPLICATIONS_VERSION_MICRO "70")
set(KDE_APPLICATIONS_VERSION "${KDE_APPLICATIONS_VERSION_MAJOR}.${KDE_APPLICATIONS_VERSION_MINOR}.${KDE_APPLICATIONS_VERSION_MICRO}")
set(SPECTACLE_VERSION ${KDE_APPLICATIONS_VERSION})
# minimum requirements

cmake_minimum_required (VERSION 3.0 FATAL_ERROR)
# Spectacle project
project(Spectacle VERSION ${SPECTACLE_VERSION})

set(QT_MIN_VERSION     "5.6.0")
set(KF5_MIN_VERSION    "5.29.0")
set(PLASMA_MIN_VERSION "5.4.0")

find_package(ECM ${KF5_MIN_VERSION} REQUIRED NO_MODULE)
set(
    CMAKE_MODULE_PATH
        ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules
        ${CMAKE_MODULE_PATH}
        ${ECM_MODULE_PATH}
        ${ECM_KDE_MODULE_DIR}
)

# set up kf5

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(ECMInstallIcons)
include(ECMSetupVersion)
include(FeatureSummary)

find_package(
    Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED
        Core
        Concurrent
        Widgets
        DBus
        PrintSupport
        Quick
)

find_package(
    KF5 ${KF5_MIN_VERSION} REQUIRED
        CoreAddons
        WidgetsAddons
        DBusAddons
        Notifications
        Config
        I18n
        KIO
        XmlGui
        WindowSystem
        DocTools
        Declarative
        NewStuff
)

add_definitions(-DQT_NO_URL_CAST_FROM_STRING)
add_definitions(-DQT_USE_QSTRINGBUILDER)
add_definitions(-DQT_NO_CAST_FROM_ASCII)
add_definitions(-DQT_NO_CAST_TO_ASCII)
add_definitions(-DQT_NO_NARROWING_CONVERSIONS_IN_CONNECT)

# optional components

find_package(KF5Kipi)
if (KF5Kipi_FOUND)
    set(KIPI_FOUND 1)
endif ()

find_package(KDEExperimentalPurpose)
if (KDEExperimentalPurpose_FOUND)
    set(PURPOSE_FOUND 1)
endif()

find_package(XCB COMPONENTS XFIXES IMAGE UTIL CURSOR)
set(XCB_COMPONENTS_ERRORS FALSE)
if (XCB_FOUND)
	find_package(Qt5X11Extras ${QT_MIN_VERSION} REQUIRED)
endif()
set(XCB_COMPONENTS_FOUND TRUE)
if(NOT XCB_XFIXES_FOUND)
	set(XCB_COMPONENTS_ERRORS "${XCB_COMPONENTS_ERRORS} XCB-XFIXES ")
	set(XCB_COMPONENTS_FOUND FALSE)
endif()
if(NOT XCB_IMAGE_FOUND)
	set(XCB_COMPONENTS_ERRORS "${XCB_COMPONENTS_ERRORS} XCB-IMAGE ")
	set(XCB_COMPONENTS_FOUND FALSE)
endif()
if(NOT XCB_UTIL_FOUND)
	set(XCB_COMPONENTS_ERRORS "${XCB_COMPONENTS_ERRORS} XCB-UTIL ")
	set(XCB_COMPONENTS_FOUND FALSE)
endif()
if(NOT XCB_CURSOR_FOUND)
	set(XCB_COMPONENTS_ERRORS "${XCB_COMPONENTS_ERRORS} XCB-CURSOR ")
	set(XCB_COMPONENTS_FOUND FALSE)
endif()

# fail build if none of the platform backends can be found
if (NOT XCB_FOUND OR NOT XCB_COMPONENTS_FOUND)
    message(FATAL_ERROR "No suitable backend platform was found. Currenty supported platforms are: XCB Components Required: ${XCB_COMPONENTS_ERRORS}")
endif()

add_definitions(-DQT_NO_URL_CAST_FROM_STRING)
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_definitions("-Wno-mismatched-tags")
endif()

# hand off to subdirectories

add_subdirectory(src)
add_subdirectory(dbus)
add_subdirectory(desktop)
add_subdirectory(icons)
add_subdirectory(doc)

# summaries

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
