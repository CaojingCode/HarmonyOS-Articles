### I. Scenarios and Constraints
**Scenario Requirements**:  
Convert Chinese audio (including mixed Chinese-English content) to text in real-time or offline on devices like phones/tablets without network connectivity. Suitable for scenarios such as real-time subtitles for hearing-impaired users and offline interactions with voice assistants.  

**Constraints**:  
● **Audio Format**: Supports PCM format, short speech mode (≤60 seconds), and long speech mode (≤8 hours).  
● **Device Limitations**: Does not support emulators; requires physical device debugging.  
● **Offline Capability**: Relies on HarmonyOS's local speech recognition engine, currently supporting only Mandarin Chinese.  



### II. Development Preparation
#### 1. Permission Configuration
○ **Microphone Permission**: Declare `ohos.permission.MICROPHONE` in `module.json5` and dynamically request user authorization:  

```json
"requestPermissions": [{
  "name": "ohos.permission.MICROPHONE",
  "reason": "Used for speech-to-text conversion",
  "usedScene": { "abilities": ["EntryAbility"], "when": "inuse" }
}]
```

#### 2. Dependency Import
Import speech recognition and audio processing modules:  

```typescript
import { speechRecognizer } from '@kit.CoreSpeechKit';
import { audio, AudioCapturer } from '@kit.AudioKit';
```



### III. Core Development Steps
#### 1. Initialize the Speech Recognition Engine
```typescript
let asrEngine: speechRecognizer.SpeechRecognitionEngine;
const sessionId = 'session_001';

// Configure engine parameters: Chinese, offline mode, short speech type
const initParams: speechRecognizer.CreateEngineParams = {
  language: 'zh-CN',
  online: 0, // 0 indicates offline mode
  extraParams: { "recognizerMode": "short" }
};

// Create the engine instance
speechRecognizer.createEngine(initParams, (err, engine) => {
  if (!err) {
    asrEngine = engine;
    console.info('Engine initialized successfully');
  } else {
    console.error(`Engine initialization failed: ${err.code}-${err.message}`);
  }
});
```

**Key Parameter Explanations**:  
● `online`: 0 for local recognition, 1 for online recognition (requires network).  
● `recognizerMode`: `short` (short speech) or `long` (long speech mode).  

#### 2. Configure Callback Listeners
Set up callbacks to handle recognition results and errors:  

```typescript
asrEngine.setListener({
  onStart: (sessionId, msg) => console.info(`Recognition started: ${msg}`),
  onResult: (sessionId, result) => {
    console.info(`Recognition result: ${result.result}`); // Real-time text output
    // Process intermediate (unstable) and final (stable) results
  },
  onError: (sessionId, code, msg) => console.error(`Error: ${code}-${msg}`),
  onComplete: (sessionId, msg) => console.info(`Recognition completed: ${msg}`)
});
```

#### 3. Audio Input Processing
**Option A: Read PCM Stream from File**  

```typescript
async function readPCMFile(filePath: string) {
  const file = fileIo.openSync(filePath, fileIo.OpenMode.READ_ONLY);
  let buffer = new ArrayBuffer(1280); // Read 1280 bytes at a time
  while (fileIo.readSync(file.fd, buffer) === 1280) {
    const uint8Array = new Uint8Array(buffer);
    asrEngine.writeAudio(sessionId, uint8Array); // Write audio stream in segments
    await sleep(40); // 40ms interval to simulate real-time stream
  }
  fileIo.closeSync(file);
}
```

**Option B: Real-time Microphone Recording**  

```typescript
const audioCapturer = new AudioCapturer();
audioCapturer.init((data: ArrayBuffer) => {
  const uint8Array = new Uint8Array(data);
  asrEngine.writeAudio(sessionId, uint8Array); // Write real-time audio stream
});
audioCapturer.start(); // Start recording
```

**Notes**:  
● Audio stream length must be 640 or 1280 bytes. Recommended write intervals: 20ms (640 bytes) or 40ms (1280 bytes).  
● Configure `maxAudioDuration` (unit: ms) in long speech mode to control maximum recognition time.  

#### 4. Start and Stop Recognition
```typescript
// Start recognition (file or real-time stream)
const startParams: speechRecognizer.StartParams = {
  sessionId,
  audioInfo: {
    audioType: 'pcm',
    sampleRate: 16000,  // 16kHz sampling rate
    soundChannel: 1,    // Mono channel
    sampleBit: 16       // 16-bit depth
  }
};
asrEngine.startListening(startParams);

// Stop recognition
asrEngine.finish(sessionId);  // Normal completion
asrEngine.cancel(sessionId);  // Force cancellation
asrEngine.shutdown();         // Release engine resources
```

#### 5. Error Handling and Debugging
**Common Error Codes**:  
● 1002200002: Recognition started repeatedly. Ensure single invocation per session.  
● 1002200006: Engine busy. Check concurrency limits.  
● Permission denial: Ensure dynamic microphone permission request and verify via `checkAccessToken()`.  



### IV. Complete Example and Optimization Suggestions
**Example Code**: The provided `Index.ets` and `AudioCapturer.ts` fully implement both file reading and microphone recording input methods. Developers can directly integrate and adapt them.  

**Optimization Suggestions**:  

1. **Audio Preprocessing**: Ensure input PCM meets 16kHz/16bit/mono format to avoid recognition failures.  
2. **Resource Management**: Call `shutdown()` promptly after recognition to prevent memory leaks.  
3. **User Experience**: Add UI progress indicators, e.g., update text area in real-time within `onResult` callback.



### V. Application Scenario Expansion
● **Real-time Subtitles**: Overlay recognized text on video playback interfaces using the `onResult` callback.  
● **Offline Command Recognition**: Predefine keywords to enable voice control without network connectivity.  
● **Accessibility Interaction**: Provide conversation transcription for hearing-impaired users, supporting text record saving.

