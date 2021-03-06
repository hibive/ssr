# 'simplescreenrecorder' executable

find_package(Threads REQUIRED)
find_package(ALSA REQUIRED)
find_package(FFmpeg REQUIRED)
find_package(PulseAudio REQUIRED)
find_package(X11 REQUIRED)

if(WITH_JACK)
	find_package(Jack REQUIRED)
endif()

if(WITH_QT5)
	find_package(Qt5Core REQUIRED)
	find_package(Qt5Gui REQUIRED)
	find_package(Qt5Widgets REQUIRED)
	find_package(Qt5X11Extras REQUIRED)
else()
	find_package(Qt4 REQUIRED QtGui)
endif()

set(sources
	AV/Input/ALSAInput.cpp
	AV/Input/ALSAInput.h
	AV/Input/GLInjectInput.cpp
	AV/Input/GLInjectInput.h
	AV/Input/JACKInput.cpp
	AV/Input/JACKInput.h
	AV/Input/PulseAudioInput.cpp
	AV/Input/PulseAudioInput.h
	AV/Input/SSRVideoStream.h
	AV/Input/SSRVideoStreamReader.cpp
	AV/Input/SSRVideoStreamReader.h
	AV/Input/SSRVideoStreamWatcher.cpp
	AV/Input/SSRVideoStreamWatcher.h
	AV/Input/X11Input.cpp
	AV/Input/X11Input.h

	AV/Output/AudioEncoder.cpp
	AV/Output/AudioEncoder.h
	AV/Output/BaseEncoder.cpp
	AV/Output/BaseEncoder.h
	AV/Output/Muxer.cpp
	AV/Output/Muxer.h
	AV/Output/OutputManager.cpp
	AV/Output/OutputManager.h
	AV/Output/OutputSettings.h
	AV/Output/SyncDiagram.cpp
	AV/Output/SyncDiagram.h
	AV/Output/Synchronizer.cpp
	AV/Output/Synchronizer.h
	AV/Output/VideoEncoder.cpp
	AV/Output/VideoEncoder.h
	AV/Output/X264Presets.cpp
	AV/Output/X264Presets.h

	AV/AVWrapper.cpp
	AV/AVWrapper.h
	AV/FastResampler.cpp
	AV/FastResampler.h
	AV/FastResampler_FirFilter.h
	AV/FastResampler_FirFilter_Fallback.cpp
	AV/FastScaler.cpp
	AV/FastScaler.h
	AV/FastScaler_Convert.h
	AV/FastScaler_Convert_Fallback.cpp
	AV/FastScaler_Scale.h
	AV/FastScaler_Scale_Fallback.cpp
	AV/FastScaler_Scale_Generic.cpp
	AV/FastScaler_Scale_Generic.h
	AV/SampleCast.h
	AV/SimpleSynth.cpp
	AV/SimpleSynth.h
	AV/SourceSink.cpp
	AV/SourceSink.h

	common/CPUFeatures.cpp
	common/CPUFeatures.h
	common/Dialogs.cpp
	common/Dialogs.h
	common/EnumStrings.h
	common/LockFreeMessageQueue.h
	common/Logger.cpp
	common/Logger.h
	common/MutexDataPair.h
	common/QueueBuffer.h
	common/TempBuffer.h

	GUI/AudioPreviewer.cpp
	GUI/AudioPreviewer.h
	GUI/DialogGLInject.cpp
	GUI/DialogGLInject.h
	GUI/ElidedLabel.cpp
	GUI/ElidedLabel.h
	GUI/HiddenScrollArea.cpp
	GUI/HiddenScrollArea.h
	GUI/HotkeyListener.h
	GUI/HotkeyListener.cpp
	GUI/Icons.cpp
	GUI/Icons.h
	GUI/MainWindow.cpp
	GUI/MainWindow.h
	GUI/PageDone.cpp
	GUI/PageDone.h
	GUI/PageInput.cpp
	GUI/PageInput.h
	GUI/PageOutput.cpp
	GUI/PageOutput.h
	GUI/PageRecord.cpp
	GUI/PageRecord.h
	GUI/PageWelcome.cpp
	GUI/PageWelcome.h
	GUI/ProfileBox.cpp
	GUI/ProfileBox.h
	GUI/VideoPreviewer.cpp
	GUI/VideoPreviewer.h

	Benchmark.cpp
	Benchmark.h
	Global.h
	Main.cpp
	Main.h
	NVidia.cpp
	NVidia.h
)

if(ENABLE_X86_ASM)

	list(APPEND sources
		AV/FastResampler_FirFilter_SSE2.cpp
		AV/FastScaler_Convert_SSSE3.cpp
		AV/FastScaler_Scale_SSSE3.cpp
	)

	set_source_files_properties(
		AV/FastResampler_FirFilter_SSE2.cpp
		PROPERTIES COMPILE_FLAGS -msse2
	)

	set_source_files_properties(
		AV/FastScaler_Convert_SSSE3.cpp
		AV/FastScaler_Scale_SSSE3.cpp
		PROPERTIES COMPILE_FLAGS -mssse3
	)

endif()

set(moc_input
	AV/Input/X11Input.h

	AV/Output/SyncDiagram.h

	common/Logger.h

	GUI/AudioPreviewer.h
	GUI/DialogGLInject.h
	GUI/ElidedLabel.h
	GUI/HiddenScrollArea.h
	GUI/HotkeyListener.h
	GUI/MainWindow.h
	GUI/PageDone.h
	GUI/PageInput.h
	GUI/PageOutput.h
	GUI/PageRecord.h
	GUI/PageWelcome.h
	GUI/ProfileBox.h
	GUI/VideoPreviewer.h
)

set(res_input
	../data/resources/resources.qrc
)

if(WITH_QT5)
	qt5_add_resources(res_output ${res_input})
	qt5_wrap_cpp(moc_output ${moc_input})
	set(QT_LIBS
		Qt5::Core
		Qt5::Gui
		Qt5::Widgets
		Qt5::X11Extras
	)
else()
	qt4_add_resources(res_output ${res_input})
	qt4_wrap_cpp(moc_output ${moc_input})
	set(QT_LIBS
		Qt4::QtGui
	)
endif()

add_executable(simplescreenrecorder
	${sources}
	${moc_output}
	${res_output}
)

target_include_directories(simplescreenrecorder PRIVATE
	${ALSA_INCLUDE_DIRS}
	${FFMPEG_INCLUDE_DIRS}
	${X11_X11_INCLUDE_PATH}
	${X11_Xext_INCLUDE_PATH}
	${X11_Xfixes_INCLUDE_PATH}
	${X11_Xi_INCLUDE_PATH}
	$<$<BOOL:${WITH_PULSEAUDIO}>:${PULSEAUDIO_INCLUDE_DIRS}>
	$<$<BOOL:${WITH_JACK}>:${JACK_INCLUDE_DIRS}>
	${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_CURRENT_SOURCE_DIR}/AV
	${CMAKE_CURRENT_SOURCE_DIR}/AV/Input
	${CMAKE_CURRENT_SOURCE_DIR}/AV/Output
	${CMAKE_CURRENT_SOURCE_DIR}/common
	${CMAKE_CURRENT_SOURCE_DIR}/GUI
)

target_link_libraries(simplescreenrecorder PRIVATE
	${ALSA_LIBRARIES}
	${FFMPEG_LIBRARIES}
	${QT_LIBS}
	${CMAKE_THREAD_LIBS_INIT}
	${X11_X11_LIB}
	${X11_Xext_LIB}
	${X11_Xfixes_LIB}
	${X11_Xi_LIB}
	$<$<BOOL:${WITH_PULSEAUDIO}>:${PULSEAUDIO_LIBRARIES}>
	$<$<BOOL:${WITH_JACK}>:${JACK_LIBRARIES}>
)

target_compile_definitions(simplescreenrecorder PRIVATE
	-DSSR_USE_X86_ASM=$<BOOL:${ENABLE_X86_ASM}>
	-DSSR_USE_FFMPEG_VERSIONS=$<BOOL:${ENABLE_FFMPEG_VERSIONS}>
	-DSSR_USE_PULSEAUDIO=$<BOOL:${WITH_PULSEAUDIO}>
	-DSSR_USE_JACK=$<BOOL:${WITH_JACK}>
	-DSSR_SYSTEM_DIR="${CMAKE_INSTALL_FULL_DATADIR}/simplescreenrecorder"
	-DSSR_VERSION="${PROJECT_VERSION}"
)

install(TARGETS simplescreenrecorder RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_BINDIR})
