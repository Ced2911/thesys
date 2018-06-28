project(THESYS_OS)

enable_language(ASM)

find_program(CMAKE_MKISOFS
    NAMES genisoimage)

set(THESYS_OS_SOURCES
	crt0_iso.s
	main.c)

if (PS_WANT_DEBUG)
	set_source_files_properties(crt0.s PROPERTIES COMPILE_FLAGS "-Wa,--defsym,ENABLE_DEBUG=1")
endif()

set(CMAKE_C_FLAGS "-O2 -Wall -m2")
set(CMAKE_EXE_LINKER_FLAGS "-nostartfiles -Wl,--script,${CMAKE_SOURCE_DIR}/src/iso.ld")
set(CMAKE_EXECUTABLE_SUFFIX ".elf")

add_definitions(-DTHESYS_OS_VERSION=\"${THESYS_OS_VERSION}\")
add_definitions(-DREENTRANT_SYSCALLS_PROVIDED)
add_definitions(-DMISSING_SYSCALL_NAMES)

add_executable(${PROJECT_NAME} ${THESYS_OS_SOURCES})

target_link_libraries(${PROJECT_NAME} ${THESYS_OS_LIBRARIES} )

add_custom_command(TARGET ${PROJECT_NAME}
	POST_BUILD
	DEPENDS ${PROJECT_NAME}
	COMMAND ${CMAKE_OBJCOPY} --output-format=binary ${PROJECT_NAME}.elf ${CMAKE_BINARY_DIR}/${PROJECT_NAME}.BIN
    COMMENT "Generating bin file ${PROJECT_NAME}.bin" )

add_custom_command(TARGET ${PROJECT_NAME}
	POST_BUILD
	DEPENDS ${CMAKE_BINARY_DIR}/${PROJECT_NAME}.bin
	COMMAND ${CMAKE_MKISOFS} 
	ARGS -G ${CMAKE_SOURCE_DIR}/src/ip.bin -sysid "SEGA SEGASATURN" -volid "PSEUDOSATURN INSTALLER" -volset "PSEUDOSATURN INSTALLER" -publisher "Cyber Warrior X" -preparer "Cyber Warrior X" -appid "PSEUDOSATURN INSTALLER" -abstract "abs.txt" -copyright "cpy.txt" -biblio "bib.txt" -full-iso9660-filenames -o ${PROJECT_NAME}.iso ${CMAKE_BINARY_DIR}/${PROJECT_NAME}.bin 
	COMMENT "Generating disc image for ${PROJECT_NAME}")

install(FILES ${CMAKE_BINARY_DIR}/src/installer/${PROJECT_NAME}.iso DESTINATION ".")

install(FILES ${CMAKE_BINARY_DIR}/${PROJECT_NAME}.BIN DESTINATION ".")
install(FILES ${CMAKE_SOURCE_DIR}/CHANGELOG DESTINATION ".")
install(FILES ${CMAKE_SOURCE_DIR}/README.TXT DESTINATION ".")

