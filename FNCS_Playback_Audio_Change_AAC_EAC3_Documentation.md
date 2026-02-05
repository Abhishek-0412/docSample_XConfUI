# FNCS_Playback_Audio_Change_AAC_EAC3 Test Case Documentation

## TestCase ID
FNCS_PLAYBACK_119

## TestCase Name
FNCS_Playback_Audio_Change_AAC_EAC3

## Table Of Contents
- [FNCS\_Playback\_Audio\_Change\_AAC\_EAC3 Test Case Documentation](#fncs_playback_audio_change_aac_eac3-test-case-documentation)
  - [TestCase ID](#testcase-id)
  - [TestCase Name](#testcase-name)
  - [Table Of Contents](#table-of-contents)
  - [Objective](#objective)
  - [GStreamer Elements and Properties](#gstreamer-elements-and-properties)
    - [playbin](#playbin)
    - [westerossink](#westerossink)
  - [Test Flow Diagram](#test-flow-diagram)
  - [Preconditions](#preconditions)
  - [Test Steps](#test-steps)
  - [Command Executed](#command-executed)
  - [Test Attributes](#test-attributes)

## Objective
To verify the capability of switching between different audio codecs (AAC and EAC3) present in the same stream without changing the video during playback. This test validates dynamic audio track switching functionality using the playbin element's current-audio property. The test stream source can be configured through TEST_STREAMS_BASE_PATH for different sources like filesrc example copying stream to USB (file:/tmp/usb/) or httpsrc via a https server (https://streamers.server:port/) based on user setup preference. Key GStreamer elements tested include playbin (audio stream management and switching), westerossink (video output), and audio tag processing for codec verification.

**VIDEO CODEC:** H264
**AUDIO CODEC:** AAC, EAC3

## GStreamer Elements and Properties

### playbin
- **uri property**: Set to multi-codec stream URL via g_object_set for AAC/EAC3 stream playback
- **flags property**: Set via g_object_set to configure audio/video playback flags
- **video-sink property**: Set to westerossink element via g_object_set for video output
- **async-handling property**: Set to true via g_object_set to enable asynchronous state changes
- **current-audio property**: Set via g_object_set to switch between different audio streams (AAC/EAC3)
- **get-audio-tags signal**: Emitted via g_signal_emit_by_name to retrieve audio codec information and tags

### westerossink
- **first-video-frame-callback signal**: Connected via g_signal_connect to monitor first frame reception during audio switching

## Test Flow Diagram

```
[Start Audio Change Test]
           ↓
[Initialize Test Environment]
           ↓
[Create Video Player Pipeline]
           ↓
[Load Multi-Codec Stream File]
     (AAC + EAC3 Audio Tracks)
           ↓
[Check Stream Has Multiple Audio Tracks] ──→ [FAIL: Only One Audio Track]
           ↓ (Pass)
[Start Playing with First Audio Codec]
           ↓
[Record Current Audio Track Info]
           ↓
[Switch to Different Audio Track] ←─┐
           ↓                         │
[Verify Audio Switch Successful]     │
           ↓                         │
[Play Video with New Audio Codec]    │
           ↓                         │
[Monitor Playback Quality]            │
  • Position Progress                 │
  • Frame Rendering                   │
  • Audio/Video Sync                  │
           ↓                         │
[Check for Audio/Video Errors] ──→ [FAIL: Quality Issues]
           ↓ (Pass)                  │
[More Audio Tracks Available?] ──Yes─┘
           ↓ (No)
[Verify All Codecs Tested Successfully]
           ↓
[Cleanup Pipeline and Resources]
           ↓
[Test Complete - SUCCESS]
```

## Preconditions
| ID | Conditions |
|----|-----------|
| 1 | tdk_mediapipelinetests application must be installed in the DUT |
| 2 | TDK_Asset_Sunrise_AAC_EAC3_v2.mp4 must be installed in the server hosting streams or installed inside the device if user is selecting filesrc instead of httpsrc |
| 3 | MediaValidationVariables.py must be configured with video_src_url_aac_eac3 variable |
| 4 | FIREBOLT_COMPLIANCE_CHECK_AV_STATUS can be set to yes/no, currently set to no in the device config file |
| 5 | FIREBOLT_COMPLIANCE_MEDIAPLAYBACK_TIMEOUT can be set to time to wait before checking for AV playback, currently set to 10 |

## Test Steps
| ID | StepName | Step Description | Expected Result |
|----|----------|------------------|-----------------|
| 1 | Initialize Test Environment | Initialize test framework data structures and create audio change log file for tracking codec switches | Test environment should be properly initialized with audio change logging enabled |
| 2 | Create GStreamer Elements | Create playbin element for pipeline management and westerossink for video output | All GStreamer elements should be successfully created and pipeline structure established |
| 3 | Configure Multi-Codec Pipeline | Set playbin URI to multi-codec stream containing both AAC and EAC3 audio tracks | Pipeline should be configured to access stream with multiple audio codecs |
| 4 | Validate Multiple Audio Streams | Query and verify that stream contains at least two different audio streams (AAC and EAC3) | Stream should contain minimum of two audio streams for codec switching functionality |
| 5 | Retrieve Initial Audio Stream Properties | Get current audio stream index and extract audio tags to identify initial codec type | Initial audio stream should be identified and codec information retrieved successfully |
| 6 | Start Initial Playback with Position Validation | Begin playback with first audio codec and monitor position progression each second to verify smooth playback rate | Playback position should progress at normal rate indicating successful playback start |
| 7 | Monitor Video Frame Rendering | Use westerossink stats property to track video frames rendered and dropped during initial playback | Frame drop rate should remain below 1% threshold with consistent rendering |
| 8 | Validate Video PTS Timing | Monitor westerossink video-pts property for presentation timestamp progression to ensure smooth video timing | Video PTS values should progress smoothly without timing discontinuities |
| 9 | Switch Audio Stream Using current-audio Property | Set playbin current-audio property to switch to different audio stream index | Audio stream switching should execute successfully to alternate codec |
| 10 | Validate Audio Stream Switch | Verify current-audio property reflects new stream index and confirm codec change | Audio stream index should match requested stream and codec change confirmed |
| 11 | Extract New Audio Stream Tags | Retrieve audio tags from new stream to verify codec type and stream properties | New audio codec information should be successfully extracted and logged |
| 12 | Continue Playback with Position Monitoring | Resume playback with new audio codec and continue position validation to ensure continuity | Position should continue progressing without interruption during codec switch |
| 13 | Monitor Audio Frame Rendering | Use native audio-sink stats property to track audio frames rendered and dropped during codec switch | Audio drop rate should remain below 1% threshold with proper codec transition |
| 14 | Validate Audio/Video Synchronization | Monitor for audio PTS errors and underflow conditions during playback with new codec | No audio PTS errors or underflows should occur indicating proper synchronization |
| 15 | Query Current Playback Position | Verify pipeline position to ensure continuous playback during codec switching | Playback position should be retrievable and indicate continuous playback progress |
| 16 | Repeat for All Available Audio Streams | Cycle through all available audio streams to test complete codec switching capability | All audio streams should be accessible and playback functional for each codec |
| 17 | Validate AV Status | Check SOC-level AV status to confirm video continues playing during audio codec switches | Video playback should remain active and synchronized throughout audio switching |
| 18 | Terminate Pipeline and Cleanup | Properly shut down pipeline, close log files, and release all allocated resources | Pipeline should terminate cleanly with all resources properly released |

## Command Executed
Based on configuration values from Video_Accelerator.config and test implementation:

```bash
tdk_mediapipelinetests test_audio_change TDK_Asset_Sunrise_AAC_EAC3_v2.mp4 checkavstatus=no timeout=10
```

Where:
- `test_audio_change`: Specific test case for audio codec switching functionality
- `TDK_Asset_Sunrise_AAC_EAC3_v2.mp4`: Multi-codec stream from MediaValidationVariables.video_src_url_aac_eac3
- `checkavstatus=no`: SOC level AV status check disabled (from FIREBOLT_COMPLIANCE_CHECK_AV_STATUS=no)
- `timeout=10`: Playback duration per audio stream (from FIREBOLT_COMPLIANCE_MEDIAPLAYBACK_TIMEOUT=10)

## Test Attributes
**Supported Models:** Video_Accelerator  
**Estimated Duration:** 3 minutes  
**Priority:** High  
**Release Version:** M121