﻿option(DYLIB "Builds dynamic library" OFF)
option(BUILD_UI "Builds UI using SDL" OFF)

# Needed for packet stuff
include(TestBigEndian)
TEST_BIG_ENDIAN(IS_BIG_ENDIAN)

if(IS_BIG_ENDIAN)
        add_compile_definitions(SYS_BIG_ENDIAN)
endif()

if(DYLIB)
	add_compile_definitions(SYS_DYLIB)
endif()

if(BUILD_UI)
	add_compile_definitions(SYS_USE_SDL2)
endif()

if((CMAKE_C_COMPILER_ID STREQUAL "Clang" OR CMAKE_C_COMPILER_ID STREQUAL "GNU"))
	set(CMAKE_C_FLAGS_DEBUG "-g")
	set(CMAKE_C_FLAGS_RELEASE "-O3")
	message("Clang/GCC flags applied")
endif()

set(SOURCES
        "Config.c"
        "Moderation.c"
        "Status.c"
        "cJSON.c"
        "UTF8.c"
        "CMath.c"
        "DyList.c"
        "Log.c"
        "Lib.c"
        "Api.c"
        "Server.c"
        "Vote.c"
        "Maps.c"
        "Packet.c"
        "Player.c"
        "Palette.c"
        "Zone.c"

        "Lobby.c"
        "Mapvote.c"
        "CharSelect.c"
        "Game.c"
        "Results.c"

        "io/Time.c"
        "io/File.c"

        "entities/Ring.c"
        "entities/CreamRing.c"
        "entities/TailsProjectile.c"
        "entities/EggmanTracker.c"
        "entities/BlackRing.c"
        "entities/ExellerClone.c"
        "entities/RMZSlug.c"
        "entities/RMZShard.c"
        "entities/Act9Wall.c"
        "entities/NotPerfect.c"
        "entities/PFLift.c"
        "entities/HDDoor.c"
        "entities/SpkieController.c"
        "entities/LCEye.c"
        "entities/LCChain.c"
        "entities/NAPSnowball.c"
        "entities/NAPIce.c"
        "entities/HillThunder.c"
        "entities/VVLava.c"
        "entities/YouCantRun.c"
        "entities/TCAcid.c"
        "entities/KAFSpeedBox.c"
        "entities/TailsDoll.c"
        "entities/DTBall.c"
        "entities/DTStalactits.c"
        "entities/Dummy.c"
        "entities/WDLatern.c"
        "entities/MJLava.c"
        "entities/MJAss.c"
        "entities/MJJudger.c"

        "maps/RavineMist.c"
        "maps/Act9.c"
        "maps/NotPerfect.c"
        "maps/PricelessFreedom.c"
        "maps/HauntingDream.c"
        "maps/DotDotDot.c"
        "maps/LimpCity.c"
        "maps/NastyParadise.c"
        "maps/Hill.c"
        "maps/VolcanoValley.c"
        "maps/YouCantRun.c"
        "maps/TortureCave.c"
        "maps/KindAndFair.c"
        "maps/DarkTower.c"
        "maps/FartZone.c"
        "maps/MajinForest.c"
        "maps/HideAndSeek.c"
        "maps/WoodDream.c"
        "maps/Marijuna.c"
)

if(DYLIB) # Build .dll or .so
		add_library(AdvancedServer SHARED ${SOURCES})

		if (WIN32)
			target_link_libraries(AdvancedServer PRIVATE enet ws2_32 winmm dbghelp)
		else() # UNIX
			set(THREADS_PREFER_PTHREAD_FLAG ON)
			find_package(Threads REQUIRED)
			target_link_libraries(AdvancedServer PRIVATE enet m Threads::Threads)
		endif()
else() # Build executable
		if(BUILD_UI)
			set(UI_SOURCES 
				"ui/Resources.c"
				"ui/Main.c"
				"ui/Image.c"
				"ui/Label.c"
				"ui/Button.c"
				"ui/TButton.c"
				"ui/MapList.c"
			  	"ui/Presets.c" 
				"ui/MapListPreset.c" 
				"ui/PingPreset.c"
				"ui/PlayerList.c"
			)

			if (WIN32)
				find_package(SDL2 CONFIG REQUIRED)
				find_package(SDL2_image CONFIG REQUIRED)
				include_directories(${SDL2_INCLUDE_DIRS} ${SDL2_IMAGE_INCLUDE_DIRS})

				add_executable(AdvancedServer
					WIN32
					"icon.rc"
					${SOURCES}
					${UI_SOURCES}
				)
			
				target_link_libraries(AdvancedServer PRIVATE ws2_32 enet winmm dbghelp $<TARGET_NAME_IF_EXISTS:SDL2::SDL2main>
					$<IF:$<TARGET_EXISTS:SDL2::SDL2>,SDL2::SDL2,SDL2::SDL2-static> 
					$<IF:$<TARGET_EXISTS:SDL2_image::SDL2_image>,SDL2_image::SDL2_image,SDL2_image::SDL2_image-static>)

			else() # UNIX
				include(FindPkgConfig)
				PKG_SEARCH_MODULE(SDL2 REQUIRED sdl2)
				PKG_SEARCH_MODULE(SDL2IMAGE REQUIRED SDL2_image>=2.0.0)
				include_directories(${SDL2_INCLUDE_DIRS} ${SDL2IMAGE_INCLUDE_DIRS})
			
				add_executable(AdvancedServer 
					${SOURCES}
					${UI_SOURCES}
				)

				set(THREADS_PREFER_PTHREAD_FLAG ON)
				find_package(Threads REQUIRED)
				target_link_libraries(AdvancedServer PRIVATE enet m Threads::Threads ${SDL2_LIBRARIES} ${SDL2IMAGE_LIBRARIES})
			endif()

		else()
			add_executable(AdvancedServer 
				${SOURCES}
				Main.c
			)

			if (WIN32)
				target_link_libraries(AdvancedServer PRIVATE enet ws2_32 winmm dbghelp)
			else() # UNIX
				set(THREADS_PREFER_PTHREAD_FLAG ON)
				find_package(Threads REQUIRED)
				target_link_libraries(AdvancedServer PRIVATE enet m Threads::Threads)
			endif()

		endif()
endif()


if(MSVC)
        add_definitions(-D_CRT_SECURE_NO_WARNINGS)
        set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /MT")
endif()

unset(DYLIB CACHE)
unset(BUILD_UI CACHE)
