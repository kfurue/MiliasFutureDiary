# Milia's Future Diary

An experimental AI-powered 3D character built for the **2024 Gemini API Developer Competition**. Milia responds to typed messages with generated dialogue, speech, facial expressions, and layered body animations.

- [Competition entry](https://ai.google.dev/competition/projects/milias-future-diary?hl=en)
- [Source code](https://github.com/kfurue/MiliasFutureDiary)
- [Demo video](https://www.youtube.com/watch?v=wM4HUUrV7KE)

## How it works

1. The player enters text in the Unity UI, or clicks outside the UI to send a `なでなで` (head-pat) interaction.
2. **Gemini 1.5 Flash** receives the conversation history and a character/system prompt. It returns structured JSON containing dialogue, language, five emotion weights, a base animation, an optional overlay animation, and a speech-bubble dismissal time.
3. Unity applies the requested expressions to the VRM 1.0 character and selects the animations. Expression weights are halved and interpolated over 0.5 seconds to reduce interference with lip sync.
4. **Google Cloud Text-to-Speech** generates MP3 audio for the dialogue. Unity loads it into an `AudioSource` and plays it.
5. **[uLipSync](https://github.com/hecomi/uLipSync)** analyzes the audio. Its phoneme updates drive the VRM character's vowel mouth shapes through a Unity event connected to a blend-shape component.
6. Unity displays the dialogue in a speech bubble, adds it to the visible conversation log, and eventually resets the expressions and animations.

## Architecture

```text
Player input / click interaction
             |
             v
Gemini 1.5 Flash (conversation history + character prompt)
             |
             v
Structured response: dialogue / language / emotion / animations / dismissal
             |
       +-----+-------------------+
       |                         |
       v                         v
VRM expressions +          Google Cloud TTS
layered animations               |
                                 v
                          MP3 -> AudioSource
                                 |
                                 v
                              uLipSync
                                 |
                                 v
                       VRM vowel mouth shapes
```

## Lip sync: recovered Unity scene settings

The repository's `Assets/Scenes/SampleScene.unity` contains the following phoneme-to-blend-shape configuration:

| Phoneme | Blend-shape index |
| --- | ---: |
| A | 4 |
| E | 7 |
| I | 6 |
| O | 8 |
| U | 5 |

Additional serialized values: `maxBlendShapeValue: 100`, `minVolume: -2.5`, `maxVolume: -1.5`, `smoothness: 0.05`, and `usePhonemeBlend: 0`. The scene connects the uLipSync update event to `uLipSyncBlendShape.OnLipSyncUpdate`, and has an audio-source proxy assigned.

The repository also includes a `uLipSyncExpressionVRM` sample adapter that maps phonemes to `UniVRM10` expression weights. **Note:** the scene's serialized event specifically targets `uLipSyncBlendShape`; the presence of the VRM adapter source alone does not establish that this adapter was the active component in the saved scene.

## Character and interaction

Milia is an adventurer from the futuristic city **Nova Citadel**. Her scientist parents disappeared while researching lost ancient technology, giving her a reason to explore and help people. This backstory is embedded in the Gemini system prompt and may be introduced gradually during conversation.

The model returns five emotion weights (`Happy`, `Angry`, `Sad`, `Relaxed`, `Surprised`), a base animation selection, and an optional overlay animation. The prompt explicitly warns that extreme expression weights can disrupt lip sync; the Unity code additionally multiplies returned weights by `0.5` and fades them in over `0.5` seconds.

The animation assets are **Anime Girl Idle Animations Free v1.0.2** by Clean Curve Studio, with nine idle animations, three additive layer animations, and two other cycle/emote animations according to the included asset README. The application prompt exposes nine base-animation choices and three overlay choices to Gemini.

The TTS implementation selects `ja-JP-Wavenet-A` for Japanese and appends `-Journey-F` to the language code for other languages. The dialogue UI includes a speech bubble and a scrollable conversation log.

## Technology

- Unity **2022.3.26f1**
- Gemini **1.5 Flash** API
- Google Cloud Text-to-Speech API
- VRM 1.0 / UniVRM10
- [uLipSync](https://github.com/hecomi/uLipSync), installed from GitHub via Unity Package Manager
- TextMesh Pro
- Anime Girl Idle Animations Free (Clean Curve Studio)

## Key files

- [`Assets/PlayerController.cs`](Assets/PlayerController.cs): interaction, Gemini request/response, conversation history, expression/animation control, TTS, and dialogue UI.
- [`Assets/Scenes/SampleScene.unity`](Assets/Scenes/SampleScene.unity): saved Unity component wiring, phoneme mapping, and audio configuration.
- [`Assets/uLipSync/Samples/04.%20VRM/Runtime/uLipSyncExpressionVRM.cs`](Assets/uLipSync/Samples/04.%20VRM/Runtime/uLipSyncExpressionVRM.cs): included VRM expression adapter source.
- [`Packages/manifest.json`](Packages/manifest.json): package dependencies, including uLipSync.
- [`ProjectSettings/ProjectVersion.txt`](ProjectSettings/ProjectVersion.txt): Unity editor version.

## Historical notes and limitations

This README reconstructs the design from the public source snapshot rather than claiming a newly tested build. The public repository currently contains a single initial commit dated August 7, 2024, so intermediate implementation history cannot be inferred from its Git log. The scene references a uLipSync profile asset, but its exact phoneme-training contents have not been verified here. API credentials and current service compatibility must be configured and checked separately before attempting to run the historical project.
