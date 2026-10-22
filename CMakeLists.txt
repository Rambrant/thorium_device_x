cmake_minimum_required(VERSION 3.30)

# Device X's whole top-level build: a pinned copy of the Thorium framework, and the
# rig, DUT and suite in this repository (rig/, dut/, suite/, acceptance/ beside this
# file). Everything else -- the runner, the instrument drivers, hal, core -- is the
# framework's, at the version named below.
project(DeviceX LANGUAGES CXX)

include(FetchContent)

# The framework and its instrument drivers share one version, and this is where this
# repository states which one it is built against. Changing it is the upgrade: on
# this DUT's schedule, not on any other DUT's. Read the framework's CHANGELOG.md for
# every version between the old tag and the new before moving it.
#
# SOURCE_SUBDIR names a directory that does not exist, on purpose: it makes
# FetchContent download the framework and nothing else. Without it,
# FetchContent_MakeAvailable() would add_subdirectory() the framework's own
# top-level CMakeLists.txt, which is that repository's own deployment and not
# something to configure inside this one.
#
# THORIUM_REPOSITORY is a variable so a mirror -- or a local checkout, for working on
# the framework and this DUT together -- can be swapped in without editing this file:
#
#     cmake --preset macos-debug -DFETCHCONTENT_SOURCE_DIR_THORIUM=../thorium
set(THORIUM_REPOSITORY "https://github.com/Rambrant/thorium.git" CACHE STRING "Where the framework is fetched from")

FetchContent_Declare(thorium
    GIT_REPOSITORY ${THORIUM_REPOSITORY}
    GIT_TAG        v0.1.2
    GIT_SHALLOW    ON
    SOURCE_SUBDIR  no-such-directory
)
FetchContent_MakeAvailable(thorium)

include(${thorium_SOURCE_DIR}/cmake/ThoriumDeployment.cmake)

# What the logs call this DUT. Set here, and only when nobody has said otherwise (so
# -DTHORIUM_DUT_NAME=... on a configure still wins), because the framework no longer
# supplies a name of its own: from the release after v0.1.2 an unset one is
# "unknown". Setting it now changes nothing at v0.1.2 and saves the upgrade a surprise.
if(NOT DEFINED THORIUM_DUT_NAME)
    set(THORIUM_DUT_NAME "DeviceX")
endif()

thorium_deployment()
