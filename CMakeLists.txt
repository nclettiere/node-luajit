cmake_minimum_required(VERSION 3.10)

project ("node-luajit")

execute_process(COMMAND node -p "require('node-addon-api').include"
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        OUTPUT_VARIABLE NODE_ADDON_API_DIR)
string(REPLACE "\n" "" NODE_ADDON_API_DIR ${NODE_ADDON_API_DIR})
string(REPLACE "\"" "" NODE_ADDON_API_DIR ${NODE_ADDON_API_DIR})

include_directories(
    ${CMAKE_JS_INC} 
    ${NODE_ADDON_API_DIR})

set(SOURCE_FILES
        "${CMAKE_SOURCE_DIR}/src/utils.cc"
        "${CMAKE_SOURCE_DIR}/src/luastate.cc"
        "${CMAKE_SOURCE_DIR}/src/nodelua.cc"
        "${CMAKE_SOURCE_DIR}/src/MyLuaWorker.cpp"
        "${CMAKE_SOURCE_DIR}/src/MyWorkerQueue.cpp"
        "${CMAKE_SOURCE_DIR}/src/MyLuaState.cpp"
        "${CMAKE_SOURCE_DIR}/3rdlibs/lua-cjson/fpconv.c"
        "${CMAKE_SOURCE_DIR}/3rdlibs/lua-cjson/lua_cjson.c"
        "${CMAKE_SOURCE_DIR}/3rdlibs/lua-cjson/strbuf.c")

add_library("node-luajit" SHARED ${CMAKE_JS_SRC} ${SOURCE_FILES})
set_target_properties("node-luajit" PROPERTIES PREFIX "" SUFFIX ".node")

if (WIN32)
    set(LUAJIT_DIR "${CMAKE_SOURCE_DIR}/win64luajit")
    set(THIRD_PARTY "${CMAKE_SOURCE_DIR}/3rdlibs/LuaJIT/src")
    target_include_directories("node-luajit" PRIVATE ${THIRD_PARTY})
    target_link_libraries("node-luajit" PRIVATE ${CMAKE_JS_LIB} "${LUAJIT_DIR}/lua51.lib")

    add_custom_command(
        TARGET "node-luajit"
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
        "${LUAJIT_DIR}/lua51.dll"
        "${CMAKE_BINARY_DIR}/Release/lua51.dll")

endif()