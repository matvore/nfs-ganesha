find_package(BISON)
find_package(FLEX)

add_definitions(
  -D__USE_GNU
  -D_GNU_SOURCE
)

BISON_TARGET(
  ConfigParser
  ${CMAKE_CURRENT_SOURCE_DIR}/conf_yacc.y
  ${CMAKE_CURRENT_BINARY_DIR}/conf_yacc.c
  COMPILE_FLAGS "--defines -pganesha_yy"
)

FLEX_TARGET(
  ConfigScanner
  ${CMAKE_CURRENT_SOURCE_DIR}/conf_lex.l
  ${CMAKE_CURRENT_BINARY_DIR}/conf_lex.c
  COMPILE_FLAGS "-Pganeshun_yy -olex.yy.c"
)

ADD_FLEX_BISON_DEPENDENCY(ConfigScanner ConfigParser)

include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

########### next target ###############

SET(config_parsing_STAT_SRCS
   analyse.c
   config_parsing.c
   conf_url.c
   analyse.h
)

add_library(config_parsing OBJECT
  ${config_parsing_STAT_SRCS}
  ${BISON_ConfigParser_OUTPUTS}
  ${FLEX_ConfigScanner_OUTPUTS}
)

add_sanitizers(config_parsing)
set_target_properties(config_parsing PROPERTIES COMPILE_FLAGS "-fPIC")

if(RADOS_URLS)
  set(config_parsing_RADOS_SRCS
    ${config_parsing_RADOS_SRCS}
    conf_url_rados.c
    )
  add_library(ganesha_rados_urls MODULE ${config_parsing_RADOS_SRCS})
  add_sanitizers(ganesha_rados_urls)
  target_link_libraries(ganesha_rados_urls
    ganesha_nfsd
    ${SYSTEM_LIBRARIES}
    ${RADOS_LIBRARIES}
    )
  include_directories(${RADOS_INCLUDE_DIR})
  set_target_properties(ganesha_rados_urls PROPERTIES SOVERSION
    "${GANESHA_MAJOR_VERSION}${GANESHA_MINOR_VERSION}")
  install(TARGETS ganesha_rados_urls LIBRARY DESTINATION ${LIB_INSTALL_DIR})

endif(RADOS_URLS)

########### install files ###############
