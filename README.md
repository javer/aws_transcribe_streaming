AWS Transcribe Streaming client for producing real-time transcriptions for your media content using HTTP/2.

## Getting started

Add necessary dependencies to `pubspec.yaml`:
```yaml
dependencies:
  aws_common: ^0.6.0
  aws_transcribe_streaming: ^0.1.0
```

Obtain a pair of Access/Secret keys for the AWS IAM user with `transcribe:StartStreamTranscription` permission.
See details in AWS documentation: [Transcribing streaming audio](https://docs.aws.amazon.com/transcribe/latest/dg/streaming.html)

It is recommended to use [Temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) with session token obtained from the backend just before starting the transcribing process.

See also [best practices](https://docs.aws.amazon.com/transcribe/latest/dg/streaming.html#best-practices) to improve streaming transcription efficiency.

## Usage

1. Create a new transcribe streaming client:
```dart
import 'package:aws_common/aws_common.dart';
import 'package:aws_transcribe_streaming/aws_transcribe_streaming.dart';

final transcribeStreamingClient = TranscribeStreamingClient(
  region: 'eu-central-1',
  credentialsProvider: StaticCredentialsProvider(AWSCredentials(
    'ASIAIOEXAMPLEEXAMPLE',                       // accessKeyId
    'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',   // secretAccessKey
    'AQoDYXdzEJr...',                             // sessionToken
    DateTime.now().add(const Duration(hours: 1)), // expiration
  )),
);
```

2. Start a stream transcription:
```dart
final (response, audioStreamSink, transcriptEventStream) =
    await transcribeStreamingClient.startStreamTranscription(
  const StartStreamTranscriptionRequest(
    languageCode: LanguageCode.enUs,
    mediaSampleRateHertz: 48000,
    mediaEncoding: MediaEncoding.pcm,
  ),
);
```

3. Subscribe to a raw TranscriptEvent stream:
```dart
final transcriptSubscription = transcriptEventStream
    .listen((TranscriptEvent event) => print(event));
```
or use a custom strategy to decode TranscriptEvents and build the realtime transcription:
```dart
final transcriptSubscription = transcriptEventStream
    .transform(const TranscriptEventStreamDecoder(PlainTextTranscriptionStrategy()))
    .listen((String message) => print(message));
```

4. Start sending audio data to the audio stream sink:
```dart
// Raw audio data from the microphone in PCM signed 16-bit little-endian audio format
Stream<Uint8List> audioStream;

final audioStreamSubscription = audioStream.listen(
  audioStreamSink.add,
  onDone: audioStreamSink.close,
);
```
