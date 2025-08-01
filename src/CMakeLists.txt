cmake_minimum_required(VERSION 3.23)
project(ecfmp_sdk)
SET(PROJECT_NAME ecfmp_sdk)

set(CMAKE_CXX_STANDARD 20)

set(src_api api/ApiDataDownloader.cpp api/ApiDataDownloader.h api/ApiDataScheduler.cpp api/ApiDataScheduler.h api/FlightInformationRegionDataParser.cpp api/FlightInformationRegionDataParser.h api/InternalApiElementCollection.h api/InternalApiElementCollection.h api/ApiElementIterator.cpp api/InternalElementCollectionTypes.h api/EventDataParser.cpp api/EventDataParser.h api/FlowMeasureDataParser.cpp api/FlowMeasureDataParser.h api/FlowMeasureMeasureParserInterface.h api/FlowMeasureFilterParser.cpp api/FlowMeasureFilterParser.h api/ApiDataParser.cpp api/ApiDataParser.h api/ApiDataDownloadedEvent.h api/FlightInformationRegionDataParserInterface.h api/EventDataParserInterface.h api/FlowMeasureDataParserInterface.h ../include/ECFMP/api/StringIdentifierApiElementCollection.h api/InternalStringIdentifierApiElementCollection.h api/FlowMeasureMeasureParser.cpp api/FlowMeasureMeasureParser.h eventbus/InternalEventBusFactory.cpp)

set(src_date date/ParseDateStrings.cpp date/ParseDateStrings.h)

set(src_flight_information_regions
        flightinformationregion/ConcreteFlightInformationRegion.h flightinformationregion/ConcreteFlightInformationRegion.cpp)

set(src_events
        event/ConcreteEvent.cpp event/ConcreteEvent.h
        event/ConcreteEventParticipant.cpp "event/ConcreteEventParticipant.h"
        event/Event.cpp eventbus/EventStream.h ../include/ECFMP/eventbus/EventListener.h ../include/ECFMP/eventbus/EventFilter.h)

set(src_euroscope euroscope/EuroscopeAircraft.h euroscope/EuroscopeAircraftFactory.h euroscope/EuroscopeAircraftFactoryImpl.cpp euroscope/EuroscopeAircraftFactoryImpl.h euroscope/EuroscopeAircraftImpl.cpp euroscope/EuroscopeAircraftImpl.h)

set(src_eventbus
        eventbus/EventStream.h
        ../include/ECFMP/eventbus/EventListener.h
        ../include/ECFMP/eventbus/EventFilter.h ../include/ECFMP/eventbus/EventBus.h eventbus/InternalEventBus.h eventbus/EventDispatcher.h eventbus/SynchronousEventDispatcher.h eventbus/AsynchronousEventDispatcher.h eventbus/EuroscopeEventDispatcher.h eventbus/PendingEuroscopeEvents.cpp eventbus/PendingEuroscopeEvents.h eventbus/EventDispatcherFactory.h eventbus/SubscriptionFlags.h)

set(src_flowmeasures
        flowmeasure/ConcreteFlowMeasure.cpp flowmeasure/ConcreteFlowMeasure.h flowmeasure/ConcreteAirportFilter.cpp flowmeasure/ConcreteAirportFilter.h flowmeasure/ConcreteEventFilter.cpp flowmeasure/ConcreteEventFilter.h flowmeasure/ConcreteLevelRangeFilter.cpp flowmeasure/ConcreteLevelRangeFilter.h flowmeasure/ConcreteRouteFilter.cpp flowmeasure/ConcreteRouteFilter.h flowmeasure/ConcreteMeasure.cpp flowmeasure/ConcreteMeasure.h flowmeasure/ConcreteMeasureFactory.cpp flowmeasure/ConcreteMeasureFactory.h flowmeasure/ConcreteFlowMeasureFilters.cpp flowmeasure/ConcreteFlowMeasureFilters.h api/FlowMeasureFilterParserInterface.h ../include/ECFMP/flowmeasure/MultipleLevelFilter.h flowmeasure/ConcreteMultipleLevelFilter.cpp flowmeasure/ConcreteMultipleLevelFilter.h ../include/mock/MultipleLevelFilterMock.h ../include/ECFMP/flowmeasure/RangeToDestinationFilter.h flowmeasure/ConcreteRangeToDestinationFilter.cpp flowmeasure/ConcreteRangeToDestinationFilter.h flowmeasure/CanonicalFlowMeasureInfo.cpp ../include/ECFMP/flowmeasure/CanonicalFlowMeasureInfo.h flowmeasure/FlowMeasureStatusUpdates.cpp flowmeasure/FlowMeasureStatusUpdates.h ../include/ECFMP/flowmeasure/ChecksAircraftApplicability.h ../include/ECFMP/flowmeasure/CustomFlowMeasureFilter.h ../include/ECFMP/flowmeasure/LevelRangeFilterType.h)

set(src_log log/LogDecorator.cpp log/LogDecorator.h log/LogDecorator.h log/NullLogger.cpp log/NullLogger.h)

set(src_pch
        pch.h pch.cpp)

set(src_plugin
        plugin/InternalSdk.h
        plugin/InternalSdk.cpp
        plugin/SdkFactory.cpp ../include/ECFMP/SdkEvents.h plugin/InternalSdkEvents.h)

set(src_thread
        thread/ThreadPool.cpp
        thread/ThreadPool.h)

set(src_time time/Clock.cpp time/Clock.h)

set(ALL_FILES
        ${src_api}
        ${src_date}
        ${src_events}
        ${src_euroscope}
        ${src_eventbus}
        ${src_flowmeasures}
        ${src_flight_information_regions}
        ${src_log}
        ${src_pch}
        ${src_plugin}
        ${src_thread}
        ${src_time})

add_library(${PROJECT_NAME} STATIC ${ALL_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-m32" LINK_FLAGS "-m32" JSON_MultipleHeaders "ON")
target_precompile_headers(${PROJECT_NAME} PRIVATE "pch.h")
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 20)

target_include_directories(${PROJECT_NAME} PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/nlohmann;"
        "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/euroscope;"
)

target_include_directories(${PROJECT_NAME} PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR};"
        "${CMAKE_CURRENT_SOURCE_DIR}/../include;"
)

# Treat Euroscope as a system include directory to suppress warning, because they have lots
target_include_directories(${PROJECT_NAME} SYSTEM PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/euroscope;"
)

target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:
        /Od
        /Zi;
        >
        $<$<CONFIG:Release>:
        /O2;
        >
        /std:c++20;
        /sdl;
        # 2025-07: passing /EHa (as is typical elsewhere in the project) causes
        #          clang-cl to hang occasionally & unpredictably when compiling
        #          date/ParseDateStrings.cpp
        /EHsc;
        ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
        /Zm200;
        ${DEFAULT_CXX_EXCEPTION_HANDLING}
        /W4;
        /WX;
        -Wno-unused-parameter; # Lots of interfaces don't use everything
        -Wno-missing-field-initializers; # Windows has loads of this sadly
)
