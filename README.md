# Zello Channels SDK for Android

An Android SDK for integrating Zello push-to-talk functionality into your applications. The SDK communicates with a Zello server over WebSocket using a JSON-based protocol.

## Features

- **Voice Messages**
  - Send voice messages from device microphone
  - Play incoming voice messages through device speaker
  - Custom audio handling with VoiceSource and VoiceReceiver interfaces
  - Support for custom audio sources (e.g., streaming from files)

- **Text Messages**
  - Send and receive text messages in channels

- **Image Messages**
  - Send and receive images in channels
  - Automatic thumbnail and full-size image handling

- **Location Sharing**
  - Send device's current location
  - Receive location messages from other users

- **Channel Features**
  - Connect to Zello channels with authentication
  - Listen-only mode support
  - Real-time channel status updates
  - User online count tracking
  - Automatic reconnection on network changes

## Requirements

- **Android SDK:** API 21 (Android 5.0 Lollipop) or higher
- **Target SDK:** API 35 (Android 15)
- **NDK:** Version 27.3.13750724
- **Java:** Version 11
- **Kotlin:** 1.7.0+

## Building the SDK

### Prerequisites

Make sure you have the following installed:
- Android Studio or Android SDK command-line tools
- NDK version 27.3.13750724
- Gradle (included via wrapper)

### Compile the AAR

From the root of the project, run:

```bash
./gradlew :channel-sdk:assembleRelease
```

For a debug build:

```bash
./gradlew :channel-sdk:assembleDebug
```

To build both variants:

```bash
./gradlew :channel-sdk:assemble
```

### Output Location

The compiled AAR files will be located at:
- **Release:** `channel-sdk/build/outputs/aar/release/zello-channel-sdk.aar`
- **Debug:** `channel-sdk/build/outputs/aar/debug/zello-channel-sdk.aar`

## Using the SDK

### Installation

#### Option 1: Local AAR

1. Copy the compiled AAR file to your app's `libs` directory
2. Add the following to your app's `build.gradle`:

```gradle
dependencies {
    implementation files('libs/zello-channel-sdk.aar')

    // Required dependencies
    implementation 'androidx.appcompat:appcompat:1.4.2'
    implementation 'com.squareup.okhttp3:okhttp:4.9.3'
    implementation 'org.jetbrains.kotlin:kotlin-stdlib:1.7.0'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.3'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.3'
}
```

#### Option 2: Maven Central

Add the dependency to your app's `build.gradle`:

```gradle
dependencies {
    implementation 'com.zello:zello-channel-sdk:0.5.3'
}
```

### Basic Usage

#### 1. Create a Session

```kotlin
import com.zello.channel.sdk.Session
import com.zello.channel.sdk.SessionListener

// Create a session
val session = Session.Builder(context)
    .setAddress("wss://your-zello-server.com")
    .setAuthToken("your-auth-token")
    .setUsername("username")
    .setPassword("password")
    .setChannel("channel-name")
    .build()
```

#### 2. Implement SessionListener

```kotlin
class MySessionListener : SessionListener {
    override fun onConnectStarted(session: Session) {
        // Connection initiated
    }

    override fun onConnectSucceeded(session: Session) {
        // Successfully connected to channel
    }

    override fun onConnectFailed(session: Session, error: SessionConnectError) {
        // Connection failed
    }

    override fun onDisconnected(session: Session) {
        // Disconnected from channel
    }

    override fun onIncomingVoiceStarted(session: Session, stream: IncomingVoiceStream) {
        // User started speaking
    }

    override fun onIncomingVoiceStopped(session: Session, stream: IncomingVoiceStream) {
        // User stopped speaking
    }

    override fun onTextMessage(session: Session, sender: String, message: String) {
        // Received text message
    }

    override fun onImageMessage(session: Session, imageInfo: ImageInfo) {
        // Received image message
    }

    override fun onLocationMessage(session: Session, sender: String, location: Location) {
        // Received location message
    }

    override fun onOutgoingVoiceStateChanged(session: Session, stream: OutgoingVoiceStream) {
        // Outgoing voice stream state changed
    }

    override fun onOutgoingVoiceError(session: Session, stream: OutgoingVoiceStream, error: OutgoingVoiceStreamError) {
        // Error in outgoing voice stream
    }

    override fun onOutgoingVoiceProgress(session: Session, stream: OutgoingVoiceStream, positionMs: Int) {
        // Outgoing voice progress update
    }

    override fun onIncomingVoiceProgress(session: Session, stream: IncomingVoiceStream, positionMs: Int) {
        // Incoming voice progress update
    }
}
```

#### 3. Connect and Use

```kotlin
// Set the listener
session.sessionListener = MySessionListener()

// Connect to the channel
session.connect()

// Send a text message
session.sendText("Hello, channel!")

// Send an image
session.sendImage(bitmap, ImageInfo(...))

// Send current location
session.sendLocation()

// Start a voice message (push-to-talk)
session.startVoiceMessage()

// Stop the voice message
session.stopVoiceMessage()

// Disconnect when done
session.disconnect()
```

### Listen-Only Mode

To connect in listen-only mode (no username/password required):

```kotlin
val session = Session.Builder(context)
    .setAddress("wss://your-zello-server.com")
    .setAuthToken("your-auth-token")
    .setChannel("channel-name")
    .build()
```

### Custom Audio Handling

Implement `VoiceSource` for custom outgoing audio:

```kotlin
class CustomVoiceSource : VoiceSource {
    override fun startProvidingAudio(sampleRate: Int, listener: VoiceSourceListener) {
        // Provide audio data to listener
    }

    override fun stopProvidingAudio() {
        // Stop providing audio
    }
}

// Use custom source
val config = OutgoingVoiceConfiguration(customVoiceSource)
session.startVoiceMessage(config)
```

Implement `VoiceReceiver` for custom incoming audio:

```kotlin
class CustomVoiceReceiver : VoiceReceiver {
    override fun prepare(stream: IncomingVoiceStream) {
        // Prepare to receive audio
    }

    override fun provideAudioDataBuffer(): ShortArray {
        // Return buffer for audio data
    }

    override fun stop() {
        // Stop receiving audio
    }
}

// Return custom receiver in SessionListener
override fun onIncomingVoiceWillStart(
    session: Session,
    streamInfo: IncomingVoiceStreamInfo
): IncomingVoiceConfiguration? {
    return IncomingVoiceConfiguration(CustomVoiceReceiver())
}
```

## Permissions

Add the following permissions to your `AndroidManifest.xml`:

```xml
<!-- Required for network communication -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- Required for recording audio -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />

<!-- Required for location sharing -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<!-- Optional: wake lock for continuous operation -->
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

Remember to request runtime permissions for Android 6.0+ (API 23+).

## Publishing

The SDK is configured to publish to Maven Central via Sonatype. To publish:

```bash
./gradlew :channel-sdk:publishChannelsSdkPublicationToSonatypeRepository
```

Signing and credentials must be configured in your `local.properties` or Gradle properties.

## Architecture

- **Session**: Main entry point for SDK functionality
- **SessionListener**: Callback interface for events
- **VoiceManager**: Handles audio encoding/decoding using Opus codec
- **Transport**: WebSocket communication layer
- **ImageMessageManager**: Handles image message transmission
- **LocationManager**: Manages location services

The SDK uses native C++ code (NDK) for Opus audio codec integration, providing high-quality voice compression.

## License

MIT License - See the [LICENSE](http://www.opensource.org/licenses/mit-license.php) file for details.

## Links

- **GitHub:** https://github.com/zelloptt/zello-channel-api/
- **Protocol Specification:** Available in the main Zello Channel API repository
- **Maven Central:** https://search.maven.org/artifact/com.zello/zello-channel-sdk

## Support

For issues, questions, or contributions, please visit the [GitHub repository](https://github.com/zelloptt/zello-channel-api/).

## Version

Current version: **0.5.3**
