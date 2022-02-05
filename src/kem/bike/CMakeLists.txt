# SPDX-License-Identifier: MIT

if(CMAKE_C_COMPILER_ID STREQUAL "GNU" AND
   CMAKE_C_COMPILER_VERSION VERSION_LESS "5" OR
   CMAKE_C_COMPILER_ID MATCHES "Clang")
    # GCC 4.9 erroneously reports warnings for certain valid code in BIKE
    # https://stackoverflow.com/questions/13746033/how-to-repair-warning-missing-braces-around-initializer
    add_compile_options(-Wno-missing-braces)
    add_compile_options(-Wno-missing-field-initializers)
endif()

if(OQS_ENABLE_KEM_bike_l1 OR OQS_ENABLE_KEM_bike_l3)
    set(_BIKE_OBJS "")
    set(CPP_DEFS_R3 "")

    add_library(kem_bike OBJECT kem_bike.c)
    set(_BIKE_OBJS $<TARGET_OBJECTS:kem_bike>)

    set(SRCS_R3 additional_r3/api.h
                additional_r3/conversions.c
                additional_r3/decode.c
                additional_r3/hash_wrapper.c
                additional_r3/kem.c
                additional_r3/ring_buffer.c
                additional_r3/ntl.cpp
                additional_r3/sampling.c
                additional_r3/shake_prng.c
                additional_r3/threshold.c
                additional_r3/utilities.c)

    add_compile_options(-include ${CMAKE_CURRENT_LIST_DIR}/functions_renaming.h)
    set_source_files_properties(additional_r3/ntl.cpp PROPERTIES COMPILE_FLAGS -lntl)

    # Round-3 KEMs
    if(OQS_ENABLE_KEM_bike_l1)
        add_library(bike_l1 OBJECT ${SRCS_R3})

        target_compile_definitions(bike_l1 PRIVATE LEVEL=1 FUNC_PREFIX=OQS_KEM_bike_l1 ${CPP_DEFS_R3})
        target_include_directories(bike_l1 PRIVATE ${CMAKE_CURRENT_LIST_DIR}/additional_r3)
        target_include_directories(bike_l1 PRIVATE ${CMAKE_CURRENT_LIST_DIR}/additional_r3/FromNist)
        target_link_libraries(bike_l1 -lntl)
        set(_BIKE_OBJS ${_BIKE_OBJS} $<TARGET_OBJECTS:bike_l1>)
    endif()

    if(OQS_ENABLE_KEM_bike_l3)
        add_library(bike_l3 OBJECT ${SRCS_R3})
        target_compile_definitions(bike_l3 PRIVATE LEVEL=3 FUNC_PREFIX=OQS_KEM_bike_l3 ${CPP_DEFS_R3})
        target_include_directories(bike_l3 PRIVATE ${CMAKE_CURRENT_LIST_DIR}/additional_r3)
        target_include_directories(bike_l1 PRIVATE ${CMAKE_CURRENT_LIST_DIR}/additional_r3/FromNist)
        target_link_libraries(bike_l1 -lntl)
        set(_BIKE_OBJS ${_BIKE_OBJS} $<TARGET_OBJECTS:bike_l3>)
    endif()
endif()

set(BIKE_OBJS ${_BIKE_OBJS} PARENT_SCOPE)
