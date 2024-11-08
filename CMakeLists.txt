cmake_minimum_required(VERSION 3.7)

project(RetroEngine)

set(DECOMP_VERSION 1.0.0)

set(RETRO_REVISION 3 CACHE STRING "What revision to compile for. Defaults to Origins = 3")
option(RETRO_FORCE_CASE_INSENSITIVE "Forces case insensivity." OFF)
option(RETRO_MOD_LOADER "Enables or disables the mod loader." ON)
option(RETRO_NETWORKING "Enables or disables networking features used for Sonic 2's 2P VS mode." ON)
option(RETRO_USE_HW_RENDER "Enables usage of the Hardware Render, menus are unplayable without it." ON)
option(RETRO_DISABLE_PLUS "Disables Plus. Should be set on for any public releases." OFF)
option(RETRO_ORIGINAL_CODE "Removes any change that differs from the original code, a playable game can't be built this way." OFF)
set(RETRO_SEPARATE_APPS 1 CACHE STRING "Decides if it builds for Sonic 1 being a separate app or Sonic 2 being a separate app. Defaults to Sonic 1")

set(RETRO_SDL_VERSION 2 CACHE STRING "Select between SDL2 and SDL1, defaults to SDL2")

if(RETRO_ORIGINAL_CODE)
    set(RETRO_MOD_LOADER OFF)
    set(RETRO_NETWORKING OFF)
endif()

set(RETRO_NAME "RSDKv4")

set(RETRO_OUTPUT_NAME ${RETRO_NAME} CACHE STRING "The exported name of the executable.")

set(RETRO_FILES
    dependencies/all/tinyxml2/tinyxml2.cpp
    RSDKv4/Graphics/Animation.cpp
    RSDKv4/Audio/Audio.cpp
    RSDKv4/Scene/Collision.cpp
    RSDKv4/Dev/Debug.cpp
    RSDKv4/Graphics/Drawing.cpp
    RSDKv4/Storage/Ini.cpp
    RSDKv4/Input/Input.cpp
    RSDKv4/fcaseopen.c
    RSDKv4/main.cpp
    RSDKv4/Core/Math.cpp
    RSDKv4/Core/ModAPI.cpp
    RSDKv4/Core/Networking.cpp
    RSDKv4/Scene/Object.cpp
    RSDKv4/Graphics/Palette.cpp
    RSDKv4/Core/Reader.cpp
    RSDKv4/Graphics/Renderer.cpp
    RSDKv4/Core/RetroEngine.cpp
    RSDKv4/Scene/Scene.cpp
    RSDKv4/Graphics/Scene3D.cpp
    RSDKv4/Scene/Script.cpp
    RSDKv4/Graphics/Sprite.cpp
    RSDKv4/Core/String.cpp
    RSDKv4/Storage/Text.cpp
    RSDKv4/Storage/Userdata.cpp
    RSDKv4/Scene/ObjectsNative/AboutScreen.cpp
    RSDKv4/Scene/ObjectsNative/AchievementDisplay.cpp
    RSDKv4/Scene/ObjectsNative/AchievementsButton.cpp
    RSDKv4/Scene/ObjectsNative/AchievementsMenu.cpp
    RSDKv4/Scene/ObjectsNative/BackButton.cpp
    RSDKv4/Scene/ObjectsNative/CWSplash.cpp
    RSDKv4/Scene/ObjectsNative/CreditText.cpp
    RSDKv4/Scene/ObjectsNative/DialogPanel.cpp
    RSDKv4/Scene/ObjectsNative/FadeScreen.cpp
    RSDKv4/Scene/ObjectsNative/InstructionsScreen.cpp
    RSDKv4/Scene/ObjectsNative/LeaderboardsButton.cpp
    RSDKv4/Scene/ObjectsNative/MenuBG.cpp
    RSDKv4/Scene/ObjectsNative/MenuControl.cpp
    RSDKv4/Scene/ObjectsNative/ModInfoButton.cpp
    RSDKv4/Scene/ObjectsNative/ModsButton.cpp
    RSDKv4/Scene/ObjectsNative/ModsMenu.cpp
    RSDKv4/Scene/ObjectsNative/MultiplayerButton.cpp
    RSDKv4/Scene/ObjectsNative/MultiplayerHandler.cpp
    RSDKv4/Scene/ObjectsNative/MultiplayerScreen.cpp
    RSDKv4/Scene/ObjectsNative/OptionsButton.cpp
    RSDKv4/Scene/ObjectsNative/OptionsMenu.cpp
    RSDKv4/Scene/ObjectsNative/PauseMenu.cpp
    RSDKv4/Scene/ObjectsNative/PlayerSelectScreen.cpp
    RSDKv4/Scene/ObjectsNative/PushButton.cpp
    RSDKv4/Scene/ObjectsNative/RecordsScreen.cpp
    RSDKv4/Scene/ObjectsNative/RetroGameLoop.cpp
    RSDKv4/Scene/ObjectsNative/SaveSelect.cpp
    RSDKv4/Scene/ObjectsNative/SegaIDButton.cpp
    RSDKv4/Scene/ObjectsNative/SegaSplash.cpp
    RSDKv4/Scene/ObjectsNative/SettingsScreen.cpp
    RSDKv4/Scene/ObjectsNative/StaffCredits.cpp
    RSDKv4/Scene/ObjectsNative/StartGameButton.cpp
    RSDKv4/Scene/ObjectsNative/SubMenuButton.cpp
    RSDKv4/Scene/ObjectsNative/TextLabel.cpp
    RSDKv4/Scene/ObjectsNative/TimeAttack.cpp
    RSDKv4/Scene/ObjectsNative/TimeAttackButton.cpp
    RSDKv4/Scene/ObjectsNative/TitleScreen.cpp
    RSDKv4/Scene/ObjectsNative/VirtualDPad.cpp
    RSDKv4/Scene/ObjectsNative/VirtualDPadM.cpp
    RSDKv4/Scene/ObjectsNative/ZoneButton.cpp
)

if(NOT PLATFORM)
    if(WIN32) # THIS ASSUMES VCPKG OR SOURCES !!!!!!!
        set(PLATFORM "Windows" CACHE STRING "The platform to compile for.")
    elseif(ANDROID)
        set(PLATFORM "Android" CACHE STRING "The platform to compile for.")
    else()
        set(PLATFORM ${CMAKE_SYSTEM_NAME} CACHE STRING "The platform to compile for.")
    endif()
endif()

include(platforms/${PLATFORM}.cmake)

set_target_properties(RetroEngine PROPERTIES OUTPUT_NAME ${RETRO_OUTPUT_NAME})

if(COMPILE_OGG)
    set(OGG_DIR dependencies/${DEP_PATH}/libogg)
    add_library(
        libogg
        STATIC
        ${OGG_DIR}/src/bitwise.c
        ${OGG_DIR}/src/framing.c
    )

    target_compile_options(libogg PRIVATE ${OGG_FLAGS})

    target_include_directories(libogg PRIVATE ${OGG_DIR}/include)
    target_include_directories(RetroEngine PRIVATE ${OGG_DIR}/include)
    target_link_libraries(RetroEngine libogg)
endif()

if(COMPILE_VORBIS)
    set(VORBIS_DIR dependencies/${DEP_PATH}/libvorbis)
    set(OGG_DIR dependencies/${DEP_PATH}/libogg)
    add_library(libvorbis STATIC
        ${VORBIS_DIR}/lib/analysis.c
        ${VORBIS_DIR}/lib/barkmel.c
        ${VORBIS_DIR}/lib/bitrate.c
        ${VORBIS_DIR}/lib/block.c
        ${VORBIS_DIR}/lib/codebook.c
        ${VORBIS_DIR}/lib/envelope.c
        ${VORBIS_DIR}/lib/floor0.c
        ${VORBIS_DIR}/lib/floor1.c
        ${VORBIS_DIR}/lib/info.c
        ${VORBIS_DIR}/lib/lookup.c
        ${VORBIS_DIR}/lib/lpc.c
        ${VORBIS_DIR}/lib/lsp.c
        ${VORBIS_DIR}/lib/mapping0.c
        ${VORBIS_DIR}/lib/mdct.c
        ${VORBIS_DIR}/lib/psy.c
        ${VORBIS_DIR}/lib/registry.c
        ${VORBIS_DIR}/lib/res0.c
        ${VORBIS_DIR}/lib/sharedbook.c
        ${VORBIS_DIR}/lib/smallft.c
        ${VORBIS_DIR}/lib/synthesis.c
        ${VORBIS_DIR}/lib/tone.c
        ${VORBIS_DIR}/lib/vorbisenc.c
        ${VORBIS_DIR}/lib/vorbisfile.c
        ${VORBIS_DIR}/lib/window.c
    )

    target_compile_options(libvorbis PRIVATE ${VORBIS_FLAGS})

    target_include_directories(libvorbis
    PRIVATE
    ${VORBIS_DIR}/include
    ${VORBIS_DIR}/lib
    ${OGG_DIR}/include
    )
    target_include_directories(RetroEngine PRIVATE ${VORBIS_DIR}/include)
    target_link_libraries(RetroEngine libvorbis libogg)
endif()

target_include_directories(RetroEngine PRIVATE
    RSDKv4/
    RSDKv4/Scene/ObjectsNative/
    dependencies/all/stb-image/
    dependencies/all/tinyxml2/
)

if(RETRO_NETWORKING)
    target_include_directories(RetroEngine PRIVATE
        dependencies/all/asio/asio/include/
    )
endif()

if(DEFINED DEP_PATH)
    target_include_directories(RetroEngine PRIVATE
        dependencies/${DEP_PATH}/
    )
endif()


target_compile_definitions(RetroEngine PRIVATE
    RSDK_REVISION=${RETRO_REVISION}
    RETRO_USE_MOD_LOADER=$<BOOL:${RETRO_MOD_LOADER}>
    RETRO_USE_NETWORKING=$<BOOL:${RETRO_NETWORKING}>
    RETRO_USING_OPENGL=$<BOOL:${RETRO_USE_HW_RENDER}>
    RETRO_USE_SDL${RETRO_SDL_VERSION}=1
    FORCE_CASE_INSENSITIVE=$<BOOL:${RETRO_FORCE_CASE_INSENSITIVE}>
    RETRO_USE_ORIGINAL_CODE=$<BOOL:${RETRO_ORIGINAL_CODE}>
    RETRO_SEPARATE_APPS=$<BOOL:${RETRO_SEPARATE_APPS}>
    
    RSDK_AUTOBUILD=$<BOOL:${RETRO_DISABLE_PLUS}>
    
    RETRO_DEV_EXTRA="${PLATFORM} - ${CMAKE_CXX_COMPILER_ID}"
    DECOMP_VERSION="${DECOMP_VERSION}"
)
