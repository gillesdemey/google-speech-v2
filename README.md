# Google Speech API v2:

## NOTICE

Google has since launched it's official [Google Cloud Speech API](https://cloud.google.com/speech/). 
I strongly recommend looking over there.

## Host:
https://www.google.com/speech-api/v2/recognize

### Parameters
**output:** json, xml not supported.

**lang:** any valid locale (en-us, nl-be, fr-fr, etc.)

**key:** Please get one from the [Google Developers Console](https://console.developers.google.com)

Key is **not optional**.

**app:** optional

You can specify an optional query string called ```app```, which returns some extra transcripts for some reason.

**client:** optional, seems to do nothing in particular

## Data:

### FLAC
Flac file; 44100Hz 32bit float, exported with Audacity. Check the audio folder in this repository for some hilarious examples.

```
Channels       : 2
Sample Rate    : 44100
Precision      : 32-bit
Sample Encoding: 32-bit Float
```

### 16-bit PCM

The following audio options are confirmed working for 16-bit PCM sample encoding:

```
Channels       : 1
Sample Rate    : 16000
Precision      : 16-bit
Sample Encoding: 16-bit Signed Integer PCM
```

One-line sox recording command:

`rec --encoding signed-integer --bits 16 --channels 1 --rate 16000 test.wav`

### Headers:
**Content-Type:**

```Content-Type: audio/x-flac; rate=44100;```

Set the rate to be equal to the rate of the FLAC file (generally 44100Hz) but it supports different rates.

```Content-Type: audio/l16; rate=16000;``` is also supported with a rate of 44100Hz or 16000Hz for files encoded with LPCM 16-bit signed-integer.

**NOTE:** Make sure the rate in your header matches the sample rate you used for your audio capture.

**User-Agent:**

not required, but for spoofing purposes use one of Chrome’s userAgent strings.

### Response:

When Google is 100% confident in it's translation, it will return the following object:

```JSON
{
   "result":[
      {
         "alternative":[
            {
               "transcript":"good morning Google how are you feeling today"
            }
         ],
         "final":true
      }
   ],
   "result_index":0
}
```

When it's doubtful, it adds a confidence parameter for you. It also seems to add multiple transcripts for some reason.

```JSON
{
  "result":[
    {
      "alternative":[
        {
          "transcript":"this is a test",
          "confidence":0.97321892
        },
        {
          "transcript":"this is a test for"
        }
      ],
      "final":true
    }
  ],
  "result_index":0
}
```

## Example

### Install sox

On OS X with Homebrew installed:

`brew install sox`

### Record audio

`rec --encoding signed-integer --bits 16 --channels 1 --rate 16000 test.wav`

### Send the request

```
curl -X POST \
--data-binary @'audio/hello (16bit PCM).wav' \
--header 'Content-Type: audio/l16; rate=16000;' \
'https://www.google.com/speech-api/v2/recognize?output=json&lang=en-us&key=yourkey'
```

Or for FLAC encoded audio:

```
curl -X POST \
--data-binary @audio/good-morning-google.flac \
--header 'Content-Type: audio/x-flac; rate=44100;' \
'https://www.google.com/speech-api/v2/recognize?output=json&lang=en-us&key=yourkey'
```

## Caveats

Here are a few caveats you have to know about, should you decide to use this API in a production environment. (I don't recommend it)

* The API only accepts up to ~10-15 seconds of audio.
* Generating [your own Speech API Key](http://www.chromium.org/developers/how-tos/api-keys), you can only make 50 requests per day.
