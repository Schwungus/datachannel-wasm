cmake_minimum_required(VERSION 3.28.0 FATAL_ERROR)

project(datachannel-wasm
	DESCRIPTION "C++ WebRTC Data Channels for WebAssembly in browsers"
	VERSION 0.4.0
	LANGUAGES CXX)

if(NOT EMSCRIPTEN)
	message(FATAL_ERROR "datachannel-wasm must be compiled with Emscripten.")
endif()

set(WASM_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/wasm/src)
set(DATACHANNELS_SRC
	${WASM_SRC_DIR}/candidate.cpp
	${WASM_SRC_DIR}/channel.cpp
	${WASM_SRC_DIR}/configuration.cpp
	${WASM_SRC_DIR}/description.cpp
	${WASM_SRC_DIR}/datachannel.cpp
	${WASM_SRC_DIR}/global.cpp
	${WASM_SRC_DIR}/peerconnection.cpp
	${WASM_SRC_DIR}/websocket.cpp)

add_library(datachannel-wasm STATIC ${DATACHANNELS_SRC})
set_target_properties(datachannel-wasm PROPERTIES VERSION ${PROJECT_VERSION})
target_compile_features(datachannel-wasm PRIVATE cxx_std_17)

target_include_directories(datachannel-wasm
	PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/wasm/include
	PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/wasm/include/rtc)

target_link_options(datachannel-wasm
	PRIVATE
		"SHELL:-sEXPORTED_RUNTIME_METHODS=HEAPU32,HEAPU8"
	PUBLIC
		"SHELL:--js-library \"${CMAKE_CURRENT_SOURCE_DIR}/wasm/js/webrtc.js\""
	    "SHELL:--js-library \"${CMAKE_CURRENT_SOURCE_DIR}/wasm/js/websocket.js\"")
