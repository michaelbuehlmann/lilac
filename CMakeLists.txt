cmake_minimum_required(VERSION 3.11)
project(LILAC LANGUAGES Fortran)
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/CMakeModules/")
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/external/CMake_Fortran_utils)

include_directories(${CMAKE_SOURCE_DIR}/cmake/CMakeModules/)
include_directories(${CMAKE_SOURCE_DIR}/external/CMake_Fortran_utils)
include(genf90_utils)
include(Sourcelist_utils)

find_package(MPI REQUIRED)
find_package(ESMF REQUIRED)

if(NOT DEFINED CIME_ROOT)
    set(CIME_ROOT ${CMAKE_SOURCE_DIR}/external/cime)
endif()

add_subdirectory(${CIME_ROOT}/src/share/util csm_share)
add_subdirectory(${CIME_ROOT}/src/share/unit_test_stubs/util csm_share_stubs)
add_subdirectory(${CIME_ROOT}/src/share/esmf_wrf_timemgr esmf_wrf_timemgr)
add_subdirectory(${CIME_ROOT}/src/drivers/mct/shr drv_share)

# Compile flags
if(CMAKE_Fortran_COMPILER_ID MATCHES "GNU")
    set(dialect "-ffree-form -std=f2008 -fimplicit-none")
    set(bounds "-fbounds-check")
endif()

if(CMAKE_Fortran_COMPILER_ID MATCHES "Intel")
    set(dialect "-stand f08 -free -implicitnone")
    set(bounds "-check bounds")
endif()

if(CMAKE_Fortran_COMPILER_ID MATCHES "PGI")
    set(dialect "-Mfreeform -Mdclchk -Mstandard -Mallocatable=03")
    set(bounds "-C")
endif()

set(CMAKE_Fortran_FLAGS_DEBUG "${CMAKE_Fortran_FLAGS_DEBUG} ${bounds}")
set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} ${dialect}")

set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} ${ESMF_COMPILER_LINE}")
set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} ${ESMF_LINK_LINE} -g -cpp")

message(STATUS "==============================================================")
message(STATUS "Fortran    Compiler : ${CMAKE_Fortran_COMPILER}")
message(STATUS "cmake Fortran Flags : ${CMAKE_Fortran_FLAGS}")
message(STATUS "==============================================================")
message(STATUS "==============================================================")

file(GLOB_RECURSE SOURCES src/*.F90)

# set(EXTERNAL_SOURCES
# ${CIME_ROOT}/src/share/util/shr_kind_mod.F90
# ${CIME_ROOT}/src/share/util/shr_sys_mod.F90
# ${CIME_ROOT}/src/share/util/shr_log_mod.F90
# ${CIME_ROOT}/src/share/util/shr_strconvert_mod.F90
# ${CIME_ROOT}/src/share/util/shr_infnan_mod.F90
# )

# add_subdirectory(lilac)
# add_executable(${PROJECT_NAME}.exe ../lilac/demo_driver.F90
# ../lilac/lilac_mod.F90 ../lilac/atmos_cap.F90  ../lilac/lilac_utils.F90
# ../lilac/lnd_cap.F90 ../lilac/cpl_mod.F90)

# add_executable(${PROJECT_NAME}.exe ${SOURCES} ${EXTERNAL_SOURCES})
add_executable(${PROJECT_NAME}.exe ${SOURCES} ${share_sources})
target_link_libraries(${PROJECT_NAME}.exe ${LIB_TO_INCLUDE})

target_include_directories(${PROJECT_NAME}.exe PRIVATE ${MPI_Fortran_MODULE_DIR})
target_link_libraries(${PROJECT_NAME}.exe ${MPI_Fortran_LIBRARIES})

# add_subdirectory(lilac)
# add_subdirectory(tests)
