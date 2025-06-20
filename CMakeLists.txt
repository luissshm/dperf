cmake_minimum_required(VERSION 3.10)
project(dperf VERSION 0.0.35 LANGUAGES C CXX)

# Abilitazione di compilazione condizionata NODAAS
option(NODAAS "Exclude libdaas library" OFF)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
#set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")

# Directory dei sorgenti
set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(LIB_DIR ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(INC_DIR ${LIB_DIR}/include)
set(SYSTEM_INFO_DIR ${SRC_DIR}/system_info_wrapper)
set(LOOPBACK_DIR ${SRC_DIR})

# Generazione dinamica dell'header di versione
configure_file(
    ${SRC_DIR}/version.h.in
    ${SRC_DIR}/version.h
    @ONLY
)

set(SOURCES
    ${LOOPBACK_DIR}/main.cpp
    ${SYSTEM_INFO_DIR}/system_info.c
    ${LOOPBACK_DIR}/test_models/model_bandwidth.cpp
    ${LOOPBACK_DIR}/timer.c
    ${LOOPBACK_DIR}/dperf_local.cpp
)

# Aggiunge facoltativamente libdaas headers and sources
if(NOT NODAAS)
    list(APPEND HEADERS
        ${INC_DIR}/daas.hpp
        ${INC_DIR}/daas_types.hpp
    )
endif()

set(HEADERS
    ${LOOPBACK_DIR}/test_models/model_bandwidth.h
    ${LOOPBACK_DIR}/timer.h
    ${LOOPBACK_DIR}/version.h
    ${LOOPBACK_DIR}/dperf_local.hpp
    # NOT IMPLEMENTED FILES
    # ${INC_DIR}/daas.hpp
    # ${INC_DIR}/daas_types.hpp
)

add_executable(dperf
    ${SOURCES}
    ${HEADERS}
)

target_link_libraries(
    dperf
    PRIVATE ${LIB_DIR}/libdaas.a
    PRIVATE pthread
    PRIVATE bluetooth
)

# Link libdaas solo se NODAAS è OFF
if(NOT NODAAS)
    target_link_libraries(dperf PRIVATE ${LIB_DIR}/libdaas.a)
    target_compile_definitions(dperf PRIVATE "WITH_DAAS")
endif()

target_include_directories(dperf PRIVATE
    ${SYSTEM_INFO_DIR}
    ${LOOPBACK_DIR}
)

# Aggiunge la daas directory solo se necessario
if(NOT NODAAS)
    target_include_directories(dperf PRIVATE ${INC_DIR})
endif()
