include(${PROJECT_SOURCE_DIR}/GUI/OpenGL/CMakeLists.txt)
include(${PROJECT_SOURCE_DIR}/GUI/TweakBar/CMakeLists.txt)

set(SIMULATION_LINK_LIBRARIES Utilities AntTweakBar glew SPlisHSPlasH ${PBD_LIBRARIES}	PBD_includes)
set(SIMULATION_DEPENDENCIES Utilities AntTweakBar glew SPlisHSPlasH Ext_PBD CopySPlisHSPlasHShaders CopyPBDShaders)

if(WIN32)
  set(SIMULATION_LINK_LIBRARIES freeglut opengl32.lib glu32.lib ${SIMULATION_LINK_LIBRARIES})
  set(SIMULATION_DEPENDENCIES freeglut ${SIMULATION_DEPENDENCIES})
else()
  set(OpenGL_GL_PREFERENCE GLVND)
  find_package(GLUT REQUIRED)
  find_package(OpenGL REQUIRED)

  include_directories( ${OPENGL_INCLUDE_DIRS} ${GLUT_INCLUDE_DIRS})

  set(SIMULATION_LINK_LIBRARIES
	${SIMULATION_LINK_LIBRARIES}
	${GLUT_LIBRARIES}
	${OPENGL_LIBRARIES}
  )
endif()

set(PBDWRAPPER_SOURCE_FILES	
	PositionBasedDynamicsWrapper/PBDWrapper.cpp
	PositionBasedDynamicsWrapper/PBDBoundarySimulator.cpp
)
set(PBDWRAPPER_HEADER_FILES
	PositionBasedDynamicsWrapper/PBDWrapper.h
	PositionBasedDynamicsWrapper/PBDRigidBody.h
	PositionBasedDynamicsWrapper/PBDBoundarySimulator.h
)

set(GUI_TWEAKBAR_SOURCE_FILES 
	${PROJECT_SOURCE_DIR}/Simulator/GUI/TweakBar/Simulator_GUI_TweakBar.cpp
	PositionBasedDynamicsWrapper/PBD_Simulator_GUI_TweakBar.cpp
)
set(GUI_TWEAKBAR_HEADER_FILES
	${PROJECT_SOURCE_DIR}/Simulator/GUI/TweakBar/Simulator_GUI_TweakBar.h
	PositionBasedDynamicsWrapper/PBD_Simulator_GUI_TweakBar.h
)

add_executable(SPHSimulator
	main.cpp

	${PROJECT_SOURCE_DIR}/Simulator/SimulatorBase.cpp
	${PROJECT_SOURCE_DIR}/Simulator/SimulatorBase.h
	${PROJECT_SOURCE_DIR}/Simulator/BoundarySimulator.cpp
	${PROJECT_SOURCE_DIR}/Simulator/BoundarySimulator.h
	${PROJECT_SOURCE_DIR}/Simulator/StaticBoundarySimulator.cpp
	${PROJECT_SOURCE_DIR}/Simulator/StaticBoundarySimulator.h

	${PROJECT_SOURCE_DIR}/Simulator/GUI/OpenGL/Simulator_OpenGL.cpp
	${PROJECT_SOURCE_DIR}/Simulator/GUI/OpenGL/Simulator_OpenGL.h

	${PROJECT_SOURCE_DIR}/Simulator/GUI/Simulator_GUI_Base.h
	${GUI_TWEAKBAR_SOURCE_FILES}
	${GUI_TWEAKBAR_HEADER_FILES}

	${VIS_FILES}
	${GUI_FILES}

	${PBDWRAPPER_SOURCE_FILES}
	${PBDWRAPPER_HEADER_FILES}
)

target_compile_definitions(SPHSimulator PUBLIC -DSPH_DATA_PATH=${SPH_DATA_PATH})
target_compile_definitions(SPHSimulator PRIVATE -DUSE_PBD -DTW_NO_LIB_PRAGMA -DTW_STATIC)
if(DL_OUTPUT)
	target_compile_definitions(SPHSimulator PRIVATE -DDL_OUTPUT)
endif()


set_target_properties(SPHSimulator PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
set_target_properties(SPHSimulator PROPERTIES RELWITHDEBINFO_POSTFIX ${CMAKE_RELWITHDEBINFO_POSTFIX})
set_target_properties(SPHSimulator PROPERTIES MINSIZEREL_POSTFIX ${CMAKE_MINSIZEREL_POSTFIX})
add_dependencies(SPHSimulator ${SIMULATION_DEPENDENCIES})
target_link_libraries(SPHSimulator ${SIMULATION_LINK_LIBRARIES})
VIS_SOURCE_GROUPS()
GUI_SOURCE_GROUPS()

source_group("Header Files\\PBD" FILES ${PBDWRAPPER_HEADER_FILES})
source_group("Source Files\\PBD" FILES ${PBDWRAPPER_SOURCE_FILES})
source_group("Header Files\\GUI\\TweakBar" FILES ${GUI_TWEAKBAR_HEADER_FILES})
source_group("Source Files\\GUI\\TweakBar" FILES ${GUI_TWEAKBAR_SOURCE_FILES})
source_group("Header Files\\GUI\\OpenGL" FILES ${PROJECT_SOURCE_DIR}/Simulator/GUI/OpenGL/Simulator_OpenGL.h)
source_group("Source Files\\GUI\\OpenGL" FILES ${PROJECT_SOURCE_DIR}/Simulator/GUI/OpenGL/Simulator_OpenGL.cpp)
