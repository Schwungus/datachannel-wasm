cmake_minimum_required(VERSION 3.28.0 FATAL_ERROR)

project(datachannel-wasm
    DESCRIPTION "C++ WebRTC Data Channels for WebAssembly in browsers"
    VERSION 0.4.0
    LANGUAGES CXX)

if(NOT EMSCRIPTEN)
    message(FATAL_ERROR "datachannel-wasm must be compiled with Emscripten.")
endif()

option(DATACHANNEL_POLLUTE_EXPORTED_RUNTIME_METHODS
    "The Emscripten linker requires a single -sEXPORTED_RUNTIME_METHODS flag containing all exported methods. Set this option to OFF if you need to provide your own set of -sEXPORTED_RUNTIME_METHODS."
    ON)

set(WASM_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/wasm/src)

add_library(datachannel-wasm STATIC
    ${WASM_SRC_DIR}/candidate.cpp
    ${WASM_SRC_DIR}/channel.cpp
    ${WASM_SRC_DIR}/configuration.cpp
    ${WASM_SRC_DIR}/description.cpp
    ${WASM_SRC_DIR}/datachannel.cpp
    ${WASM_SRC_DIR}/global.cpp
    ${WASM_SRC_DIR}/peerconnection.cpp
    ${WASM_SRC_DIR}/websocket.cpp)
set_target_properties(datachannel-wasm PROPERTIES VERSION ${PROJECT_VERSION})
target_compile_features(datachannel-wasm PRIVATE cxx_std_17)

target_include_directories(datachannel-wasm
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/wasm/include
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/wasm/include/rtc)

target_link_options(datachannel-wasm
    PUBLIC
        "SHELL:--js-library \"${CMAKE_CURRENT_SOURCE_DIR}/wasm/js/webrtc.js\""
        "SHELL:--js-library \"${CMAKE_CURRENT_SOURCE_DIR}/wasm/js/websocket.js\"")

if(DATACHANNEL_POLLUTE_EXPORTED_RUNTIME_METHODS)
    target_link_options(datachannel-wasm
        PUBLIC "SHELL:-sEXPORTED_RUNTIME_METHODS=HEAPU32,HEAPU8")
endif()
