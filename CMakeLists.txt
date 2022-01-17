cmake_minimum_required(VERSION 2.8)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

project(blues)
include("${VITASDK}/share/vita.cmake" REQUIRED)

set(VITA_VERSION  "01.00")
set(VITA_MKSFOEX_FLAGS "${VITA_MKSFOEX_FLAGS} -d PARENTAL_LEVEL=1 -d ATTRIBUTE2=12")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=gnu11 -Wall -Wpedantic -g -O0")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++14 -Wall -Wpedantic -g -O0")

set(COMMON_SRCS main.c sys_sdl2.c util.c)

set(BB_SRCS bb/decode.c bb/game.c bb/level.c bb/objects.c bb/resource.c bb/screen.c bb/sound.c bb/staticres.c bb/tiles.c bb/unpack.c)
set(JA_SRCS ja/game.c ja/level.c ja/resource.c ja/screen.c ja/sound.c ja/staticres.c ja/unpack.c)
set(P2_SRCS p2/bosses.c p2/game.c p2/level.c p2/monsters.c p2/resource.c p2/screen.c p2/sound.c p2/staticres.c p2/unpack.c)

set(TARGET_SRCS ${BB_SRCS})
set(TARGET_INCS "bb/")
set(TARGET_DATA "bb_")
set(VITA_APP_NAME "Blues Brothers")
set(VITA_VPK_NAME "blues-bros.vpk")
set(VITA_TITLEID  "BLUES0001")

if(DEFINED TARGET_GAME)
    if(TARGET_GAME STREQUAL "JA")
        set(TARGET_SRCS ${JA_SRCS})
        set(TARGET_INCS "ja/")
        set(TARGET_DATA "ja_")
        set(VITA_APP_NAME "BB: Jukebox Adventure")
        set(VITA_VPK_NAME "bb-jukebox.vpk")
        set(VITA_TITLEID  "BLUES0002")
    endif()
    if(TARGET_GAME STREQUAL "P2")
        set(TARGET_SRCS ${P2_SRCS})
        set(TARGET_INCS "p2/")
        set(TARGET_DATA "p2_")
        set(VITA_APP_NAME "Prehistorik 2")
        set(VITA_VPK_NAME "prehistorik-2.vpk")
        set(VITA_TITLEID  "PREHIST01")
    endif()
endif()

include_directories(
    .
    ${TARGET_INCS}
)

add_executable(${PROJECT_NAME}
    ${COMMON_SRCS}
    ${TARGET_SRCS}
)

target_link_libraries(${PROJECT_NAME}
  SDL2
  modplug
  SceDisplay_stub
  SceCtrl_stub
  SceAudio_stub
  SceSysmodule_stub
  SceGxm_stub
  SceCommonDialog_stub
  SceAppMgr_stub
  SceTouch_stub
  SceHid_stub
  SceMotion_stub
  m
  stdc++
  c
)

vita_create_self(eboot.bin ${PROJECT_NAME})
vita_create_vpk(${VITA_VPK_NAME} ${VITA_TITLEID} eboot.bin
  VERSION ${VITA_VERSION}
  NAME ${VITA_APP_NAME}
  FILE sce_sys/${TARGET_DATA}icon0.png sce_sys/icon0.png
  FILE sce_sys/${TARGET_DATA}bg.png sce_sys/livearea/contents/bg.png
  FILE sce_sys/${TARGET_DATA}startup.png sce_sys/livearea/contents/startup.png
  FILE sce_sys/template.xml sce_sys/livearea/contents/template.xml
  FILE ${TARGET_DATA}data data
)