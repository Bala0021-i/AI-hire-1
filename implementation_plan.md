# Voice-Based AI Interview using AssemblyAI

Enhance the candidate interview session by integrating voice-based response recording. Candidates will be able to answer interview questions using their microphone, which will automatically transcribe to text via AssemblyAI and submit as a standard message. Manual typing will remain available as a fallback in case microphone permission is denied or not supported.

## User Review Required

> [!IMPORTANT]
> The AssemblyAI API key must be configured as `ASSEMBLYAI_API_KEY` in your `.env.local` file. Since we want to ensure zero exposure of keys in the browser, the frontend will interact exclusively with the newly created server-side Next.js route `/api/speech-to-text`.

## Open Questions

> [!NOTE]
> None at the moment. The requirements are fully detailed. We will implement standard MediaRecorder capabilities in the browser, transcribing audio streams to WebM/WAV before transmitting.

## Proposed Changes

---

### Backend Integration Layer

#### [NEW] [route.ts](file:///c:/Users/gokul/Downloads/AI_Recruiter/app/api/speech-to-text/route.ts)
- Implement a Next.js App Router POST handler `/api/speech-to-text`.
- Accepts `multipart/form-data` containing the recorded audio file.
- Securely sends the raw audio to AssemblyAI's `v2/upload` endpoint.
- Triggers a transcription request via AssemblyAI's `v2/transcript` endpoint.
- Polls for completion status and returns the transcribed text to the client.

---

### Candidate Interview UI Component

#### [MODIFY] [page.tsx](file:///c:/Users/gokul/Downloads/AI_Recruiter/app/candidate/interviews/%5Bid%5D/page.tsx)
- Extract the core sending and follow-up generation logic from `handleSendMessage` into a reusable, async `sendCandidateText` function.
- Add React state variables to track microphone recording state:
  - `isRecording`: whether recording is in progress.
  - `recordingState`: `'idle' | 'recording' | 'transcribing' | 'saved'`.
  - `micPermissionDenied`: boolean to flag if microphone access is blocked.
  - `audioChunks`: Array of audio Blob chunks.
- Implement microphone helper functions:
  - `startRecording()`: Requests microphone permission (`getUserMedia`), instantiates a `MediaRecorder`, and begins recording.
  - `stopRecording()`: Stops the recorder and compiles the chunks.
  - `processAudioAndSend(audioBlob)`: Packages the blob into `FormData`, posts it to `/api/speech-to-text`, sets status indicators, and invokes `sendCandidateText` with the resulting transcript.
- Update the bottom input bar layout:
  - Show the microphone recording button by default.
  - Display progress indicators: `🎤 Recording...` (animated red badge), `Transcribing...`, `Answer Saved` with smooth transitions.
  - If `micPermissionDenied` is true, display a clean warning badge/alert and fallback to the original textarea/send layout.
  - Provide a toggle to manually fallback to typing if preferred or if permissions are missing.

---

## Verification Plan

### Automated Tests
- Build test: `npm run build` to verify page compilation and type consistency.

### Manual Verification
- Navigating to `/candidate/interviews/[id]` using the browser.
- Simulating microphone grant and denial:
  - **Permission Granted**: Clicking microphone, speaking an answer, clicking stop, watching transcription render, and checking if the AI posts a follow-up.
  - **Permission Denied**: Seeing the warning banner, verifying that the typing fallback textarea appears, and checking if typed responses function as usual.
- Verifying the Firestore database records to ensure transcripts are saved under `interviews/{id}/messages` matching the required schema.
