# Kalpa for Pipecat

Kalpa text-to-speech services for [Pipecat](https://pipecat.ai), maintained by
[Kalpa Labs](https://kalpalabs.ai), the company providing the Kalpa API.

This community integration provides `KalpaTTSService` for WebSocket streaming
and `KalpaHttpTTSService` for HTTP streaming. See the
[Kalpa speech documentation](https://docs.kalpalabs.ai/text-to-speech) for
available voices, models, and languages.

## Demo

[Watch the Pipecat + Kalpa TTS 1 demo](https://github.com/kalpalabs/pipecat-kalpa/releases/download/v0.1.0/kalpa-pipecat-live-demo.mp4).
The browser conversation includes English and Hindi speech, a spoken
interruption, and timestamped Pipecat events.

## Installation

Requires Python 3.11 or later and Pipecat 1.13.x.
Automated compatibility tests pass with Pipecat 1.13.0 on Python 3.12.

```bash
uv add "pipecat-kalpa @ git+https://github.com/kalpalabs/pipecat-kalpa.git@v0.1.0"
```

## Usage

Get an API key from the [Kalpa dashboard](https://kalpalabs.ai/dashboard/keys/) and set
`KALPA_API_KEY` in your environment. See
[authentication](https://docs.kalpalabs.ai/authentication) for details.

```python
import os

from pipecat.pipeline.pipeline import Pipeline
from pipecat_kalpa import KalpaTTSService

tts = KalpaTTSService(api_key=os.environ["KALPA_API_KEY"])

# Add Kalpa to an existing voice pipeline.
pipeline = Pipeline(
    [
        transport.input(),
        stt,
        user_aggregator,
        llm,
        tts,
        transport.output(),
        assistant_aggregator,
    ]
)
```

`transport`, `stt`, `llm`, and the context aggregators are the other components
of your application. A complete runnable example is provided below.

Use `KalpaTTSService` to send a reply's text as it arrives over one WebSocket
connection. For HTTP streaming, replace it with
`KalpaHttpTTSService(api_key=os.environ["KALPA_API_KEY"])`.

## Configuration

Both services require `api_key`. Pass voice and model overrides with
`settings=KalpaTTSService.Settings(...)` or
`settings=KalpaHttpTTSService.Settings(...)`:

```python
tts = KalpaTTSService(
    api_key=os.environ["KALPA_API_KEY"],
    settings=KalpaTTSService.Settings(
        voice="18c26f49-9f9c-400c-bec9-7467f28e1db5",
        model="kalpa-tts-1",
    ),
)
```

| Setting | Default | Description |
| --- | --- | --- |
| `voice` | `18c26f49-9f9c-400c-bec9-7467f28e1db5` | Arjun. Use a voice ID from `GET /v1/voices`. |
| `model` | `kalpa-tts-1` | A model ID from `GET /v1/models`. |
| `language` | `None` | Guides Pipecat sentence splitting; it is not sent to Kalpa. |

Voice and model changes can be sent during a session with
`TTSUpdateSettingsFrame`. The WebSocket service reconnects to apply them.

| Constructor argument | Service | Default |
| --- | --- | --- |
| `base_url` | WebSocket | `wss://api.kalpalabs.ai` |
| `base_url` | HTTP | `https://api.kalpalabs.ai` |
| `text_aggregation_mode` | WebSocket | `TextAggregationMode.TOKEN` |
| `aiohttp_session` | HTTP | `None`; the service creates and closes its own session |

The services produce 24 kHz mono, 16-bit PCM audio. Pipecat's output transport
resamples it when your pipeline uses a different output rate. Do not pass
`sample_rate` to these services.

## Run the voice example

The example uses Deepgram for speech recognition, OpenAI for generating
replies, and Kalpa for speech output. It connects to your browser over WebRTC.

```bash
git clone https://github.com/kalpalabs/pipecat-kalpa.git
cd pipecat-kalpa
uv sync --extra examples
cp .env.example .env
```

Set `KALPA_API_KEY`, `DEEPGRAM_API_KEY`, and `OPENAI_API_KEY` in `.env`, then run:

```bash
uv run --extra examples python examples/voice.py --transport webrtc
```

Open the address printed by the runner, enable microphone access, and connect.
Speak while the assistant is talking to interrupt its reply.

Set `KALPA_TTS_TRANSPORT=http` to run the same example with HTTP streaming:

```bash
KALPA_TTS_TRANSPORT=http uv run --extra examples python examples/voice.py --transport webrtc
```

`KALPA_VOICE_ID` optionally overrides the example's voice.

## Development

```bash
uv sync
uv run pytest
uv run ruff check .
uv run ruff format --check .
uv build
```

Tests use local fake API servers and do not need API keys. Report issues on
the [integration repository](https://github.com/kalpalabs/pipecat-kalpa/issues).

## License

BSD-2-Clause. See [LICENSE](LICENSE).
