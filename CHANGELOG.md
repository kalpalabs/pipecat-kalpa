# Changelog

## 0.1.3

- Fix: when text generation paused mid-reply for longer than
  `stop_frame_timeout_s` (15 seconds by default) before Kalpa returned the
  reply's first audio, text sent before the pause was missing from the
  conversation history. A reply now stays open while its text pauses, for up
  to 45 seconds in all, and plays as one response. Speech queued behind a
  paused reply waits for it.
- Tested with Pipecat 1.17.

## 0.1.2

- Support Pipecat 1.13 through 1.x. Installing the package no longer
  downgrades Pipecat 1.14 or later to 1.13.
- Test the oldest and newest supported Pipecat releases in CI, weekly as well
  as on every change.
- Document a known limitation: if a WebSocket reply pauses for longer than
  `stop_frame_timeout_s` before its first audio, text sent before the pause
  can be missing from the conversation history (the unfinished sentence in
  token mode; with Pipecat 1.16.0, everything sent before the pause). Tests
  track it in both aggregation modes.
- No changes to the text-to-speech services since 0.1.1.

## 0.1.1

- Publish the integration on PyPI with GitHub Actions trusted publishing.
- Update installation instructions and make the README links work on PyPI.
- No changes to the text-to-speech services since 0.1.0.

## 0.1.0

- Add WebSocket and HTTP text-to-speech services for Kalpa.
- Support interruption handling and runtime voice and model settings.
- Include a browser voice example and automated tests.
