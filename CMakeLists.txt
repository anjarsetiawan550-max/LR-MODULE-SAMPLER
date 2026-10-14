cmake_minimum_required(VERSION 3.22)
project(SampleCore VERSION 2.1.0)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_OSX_DEPLOYMENT_TARGET "10.13" CACHE STRING "")

include(FetchContent)
FetchContent_Declare(JUCE
    GIT_REPOSITORY https://github.com/juce-framework/JUCE.git
    GIT_TAG        8.0.4
    GIT_SHALLOW    ON)
FetchContent_MakeAvailable(JUCE)

juce_add_plugin(SampleCore
    COMPANY_NAME            "LR Audio"
    PRODUCT_NAME            "LR Module Sampler"
    PLUGIN_MANUFACTURER_CODE LrAu
    PLUGIN_CODE             ScCr
    FORMATS                 VST3 Standalone
    VST3_CATEGORIES         Fx
    IS_SYNTH                FALSE
    NEEDS_MIDI_INPUT        FALSE
    NEEDS_MIDI_OUTPUT       FALSE
    NEEDS_WEB_BROWSER       FALSE
    NEEDS_CURL              FALSE
    ICON_BIG                "${CMAKE_CURRENT_SOURCE_DIR}/assets/icon/icon_1024.png"
    ICON_SMALL              "${CMAKE_CURRENT_SOURCE_DIR}/assets/icon/icon_256.png"
    COPY_PLUGIN_AFTER_BUILD FALSE)

juce_generate_juce_header(SampleCore)

# Semua PNG di folder assets/ otomatis masuk BinaryData (BinaryData::nama_png)
file(GLOB SC_ASSET_FILES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/assets/*.png")
juce_add_binary_data(SampleCoreAssets SOURCES ${SC_ASSET_FILES})
set_target_properties(SampleCoreAssets PROPERTIES POSITION_INDEPENDENT_CODE TRUE)

target_sources(SampleCore PRIVATE
    Source/PluginProcessor.cpp
    Source/PluginProcessor.h
    Source/PluginEditor.cpp
    Source/PluginEditor.h
    Source/Dsp.h
    Source/UiKit.h
    Source/LcdDisplay.h)

target_compile_definitions(SampleCore PUBLIC
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_VST3_CAN_REPLACE_VST2=0
    JUCE_DISPLAY_SPLASH_SCREEN=0)

target_link_libraries(SampleCore
    PRIVATE
        SampleCoreAssets
        juce::juce_audio_utils
        juce::juce_dsp
    PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags)
