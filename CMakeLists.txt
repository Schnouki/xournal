cmake_minimum_required(VERSION 2.8)

project(xournal C)
set(VERSION 0.4.6)

find_package(Gettext)
find_package(GTK2 2.10 REQUIRED gtk)

find_package(PkgConfig REQUIRED)
pkg_check_modules(PACKAGE REQUIRED libgnomecanvas-2.0>=2.4.0 poppler-glib>=0.5.4 pangoft2>=1.0)
string(REPLACE ";" " " PACKAGE_CFLAGS "${PACKAGE_CFLAGS}")

set(TTSUBSET_SRC
  src/ttsubset/list.c
  src/ttsubset/sft.c
  src/ttsubset/ttcr.c)
set_source_files_properties(${TTSUBSET_SRC} PROPERTIES
  COMPILE_DEFINITIONS "NO_MAPPERS;NO_TYPE3;NO_TYPE42")

set(XOURNAL_SRC
  src/main.c
  src/xo-callbacks.c
  src/xo-file.c
  src/xo-interface.c
  src/xo-misc.c
  src/xo-paint.c
  src/xo-print.c
  src/xo-shapes.c
  src/xo-support.c
  ${TTSUBSET_SRC})

if(GETTEXT_FOUND)
  file(GLOB PO_FILES "po/*.po")
  set(POT_FILE ${CMAKE_BINARY_DIR}/xournal.pot)
  
  find_program(XGETTEXT_EXECUTABLE xgettext)
  add_custom_command(
    OUTPUT ${POT_FILE}
    COMMAND ${XGETTEXT_EXECUTABLE}
        --output=${POT_FILE}
        --directory=${CMAKE_SOURCE_DIR}
        --omit-header
	--add-comments --keyword=_ --keyword=N_
        --keyword=C_:1c,2
        --keyword=NC_:1c,2
	--keyword=g_dcgettext:2
	--keyword=g_dngettext:2,3
	--keyword=g_dpgettext2:2c,3
	--flag=N_:1:pass-c-format
	--flag=C_:2:pass-c-format
	--flag=NC_:2:pass-c-format
	--flag=g_dngettext:2:pass-c-format
        --flag=g_strdup_printf:1:c-format
        --flag=g_string_printf:2:c-format
        --flag=g_string_append_printf:2:c-format
        --flag=g_error_new:3:c-format
        --flag=g_set_error:4:c-format
        --flag=g_markup_printf_escaped:1:c-format
        --flag=g_log:3:c-format
        --flag=g_print:1:c-format
        --flag=g_printerr:1:c-format
        --flag=g_printf:1:c-format
        --flag=g_fprintf:2:c-format
        --flag=g_sprintf:2:c-format
        --flag=g_snprintf:3:c-format
        --flag=g_scanner_error:2:c-format
        --flag=g_scanner_warn:2:c-format
        ${XOURNAL_SRC}
    DEPENDS ${XOURNAL_SRC})
  add_custom_target(pot DEPENDS ${POT_FILE} ${XOURNAL_SRC})
  
  gettext_create_translations(${POT_FILE} ALL ${PO_FILES})
  add_definitions(
    -DENABLE_NLS
    -DGETTEXT_PACKAGE="xournal"
    -DPACKAGE_LOCALE_DIR="${CMAKE_INSTALL_PREFIX}/share/locale")
endif(GETTEXT_FOUND)

add_executable(xournal ${XOURNAL_SRC})
add_definitions(
  -DVERSION="${VERSION}"
  -DPACKAGE="xournal"
  -DPACKAGE_DATA_DIR="${CMAKE_INSTALL_PREFIX}/share")
set_target_properties(xournal PROPERTIES
  COMPILE_FLAGS "${GTK2_CFLAGS} ${PACKAGE_CFLAGS}"
)
target_link_libraries(xournal ${GTK2_LIBRARIES} ${PACKAGE_LIBRARIES})
install(TARGETS xournal RUNTIME DESTINATION bin)

install(
  DIRECTORY   pixmaps html-doc
  DESTINATION "${CMAKE_INSTALL_PREFIX}/share/xournal")
install(
  FILES       pixmaps/xournal.svg
  DESTINATION "${CMAKE_INSTALL_PREFIX}/share/icons/hicolor/scalable/apps")
install(
  FILES       pixmaps/xoj.svg
  DESTINATION "${CMAKE_INSTALL_PREFIX}/share/icons/hicolor/scalable/mimetypes")
install(
  FILES       pixmaps/xoj.svg
  DESTINATION "${CMAKE_INSTALL_PREFIX}/share/icons/hicolor/scalable/mimetypes"
  RENAME      gnome-mime-application-x-xoj.svg)
install(
  FILES       xournal.xml
  DESTINATION "${CMAKE_INSTALL_PREFIX}/share/mime/packages")
install(
  FILES       x-xoj.desktop
  DESTINATION "${CMAKE_INSTALL_PREFIX}/share/mimelnk/application")
