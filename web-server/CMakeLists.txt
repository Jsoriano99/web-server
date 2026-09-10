#puerta de entrada del build, le dice a cmake donde esta IDF y el nombre del proyecto

cmake_minimum_required(VERSION 3.22)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(web-server)


