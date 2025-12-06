# Code-Switching Support Guide

## Overview

Code-switching is the practice of alternating between two or more languages within a single conversation or audio recording. This feature enables Whisper-WebUI to accurately transcribe audio that contains multiple languages.

This implementation is based on the approach discussed in [OpenAI Whisper Discussion #2009](https://github.com/openai/whisper/discussions/2009).

## Problem Statement

By default, Whisper is designed for monolingual audio inputs. When processing audio containing multiple languages (code-switching), Whisper often:
- Translates everything to English instead of transcribing in the original languages
- Produces inconsistent results depending on the model size
- May hallucinate or produce incorrect transcriptions at language boundaries

## Solution

The code-switching feature addresses this by:
1. **Segmenting the audio** into smaller chunks
2. **Detecting the language** for each chunk automatically
3. **Transcribing each chunk** with the appropriate language setting
4. **Combining all segments** back together with correct timestamps

## How to Use

### Via Web UI

1. **Open the Advanced Parameters** accordion in the Whisper-WebUI
2. **Enable Code-Switching**: Check the "Enable Code-Switching" checkbox
3. **Adjust Chunk Length** (optional): Set the "Code-Switching Chunk Length (s)" parameter
   - Default: 30 seconds
   - Smaller chunks = More frequent language detection, but may split sentences
   - Larger chunks = Better context for transcription, but less granular language detection
4. **Set Language to Automatic**: Make sure the Language dropdown is set to "Automatic Detection"
5. **Start transcription** as usual

### Via REST API

When using the REST API, include the following parameters in your request:

```python
{
    "whisper": {
        "enable_code_switching": true,
        "code_switching_chunk_length": 30,
        "lang": null  # Important: Set to null for automatic detection
    }
}
```

## Best Practices

### When to Use Code-Switching

Use this feature when your audio contains:
- **Multilingual conversations**: People switching between languages mid-conversation
- **Mixed-language content**: Educational content, interviews, or presentations in multiple languages
- **Code-switched speech**: Single speakers alternating between languages (e.g., Spanglish, Konglish)

### When NOT to Use Code-Switching

Avoid this feature when:
- **Monolingual audio**: Audio in a single language (better to specify the language explicitly)
- **Translation needed**: If you want everything translated to English (use the "Translate to English" option instead)
- **Very short audio**: Audio shorter than the chunk length

### Optimal Chunk Length Settings

| Audio Type | Recommended Chunk Length | Reason |
|------------|-------------------------|---------|
| Rapid code-switching | 15-20 seconds | Captures frequent language changes |
| Moderate mixing | 30 seconds (default) | Balanced performance |
| Infrequent switching | 45-60 seconds | Better context for each language |

## Parameters

### Enable Code-Switching
- **Type**: Boolean
- **Default**: `false`
- **Description**: Enable multi-language support for audio with code-switching

### Code-Switching Chunk Length
- **Type**: Integer (seconds)
- **Default**: `30`
- **Range**: > 0
- **Description**: Duration of each audio chunk for independent language detection and transcription

## Technical Details

### Implementation

The code-switching feature works by:

1. **Audio Chunking**: The input audio is divided into chunks of the specified duration
2. **Language Detection**: For each chunk, Whisper's built-in language detection is used (by setting `language=None`)
3. **Independent Transcription**: Each chunk is transcribed independently with its detected language
4. **Timestamp Adjustment**: All segment timestamps are adjusted to align with the original audio timeline
5. **Segment Combination**: All transcribed segments are combined into a single result

### Limitations

- **Chunk Boundaries**: Language changes near chunk boundaries might not be perfectly detected
- **Context Loss**: Each chunk is transcribed independently, which may reduce context for better accuracy
- **Processing Time**: Code-switching mode may take longer than standard transcription due to multiple language detection passes
- **Model Dependency**: Accuracy depends on Whisper's ability to detect and transcribe each language

## Example Use Cases

### Example 1: Bilingual Interview
```
Audio: English question → Spanish answer → English follow-up → Spanish response
Result: Each segment transcribed in its original language
```

### Example 2: Educational Content
```
Audio: Teacher explains in English → Provides example in Mandarin → Continues in English
Result: Accurate transcription maintaining both languages
```

### Example 3: Multilingual Meeting
```
Audio: Participants speaking in German, English, and French
Result: Each speaker's segment transcribed in their spoken language
```

## Troubleshooting

### Issue: Languages are still being translated to English
**Solution**:
- Ensure "Translate to English" is **unchecked**
- Verify "Language" is set to "Automatic Detection"
- Check that "Enable Code-Switching" is **checked**

### Issue: Poor transcription quality at language boundaries
**Solution**:
- Try reducing the chunk length to catch language changes more frequently
- Increase `language_detection_threshold` for more confident detection

### Issue: Transcription is slower than expected
**Solution**:
- This is expected behavior due to per-chunk processing
- Consider increasing chunk length if language switches are infrequent
- Use a smaller Whisper model if quality is acceptable

### Issue: Same language detected for all chunks
**Solution**:
- Verify the audio actually contains multiple languages
- Try adjusting `language_detection_threshold` parameter
- Ensure the Whisper model supports both languages

## Performance Considerations

- **Speed**: Code-switching mode is slower than standard transcription
- **Memory**: Memory usage is similar to standard transcription
- **Accuracy**: May be slightly lower than language-specific transcription for monolingual audio

## Related Features

This feature works in conjunction with:
- **Speaker Diarization**: Can identify which speaker uses which language
- **VAD (Voice Activity Detection)**: Can help reduce silent portions before chunking
- **Translation**: Can be applied after code-switching transcription if needed

## References

- [OpenAI Whisper Discussion #2009](https://github.com/openai/whisper/discussions/2009) - Original discussion on code-switching
- [Whisper Paper](https://arxiv.org/abs/2212.04356) - Whisper's language detection capabilities
- [faster-whisper Documentation](https://github.com/SYSTRAN/faster-whisper) - Implementation used by default
